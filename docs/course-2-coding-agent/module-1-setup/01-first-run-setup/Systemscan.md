using System.ComponentModel;            // Win32Exception
using System.Diagnostics;               // Process, ProcessStartInfo
using System.Globalization;             // CultureInfo
using System.Management;                // ManagementObjectSearcher, ManagementObject
using System.Net.NetworkInformation;    // IPGlobalProperties, NetworkInformationException
using System.Runtime.InteropServices;   // RuntimeInformation
using System.Runtime.Versioning;        // SupportedOSPlatform
using System.Security;                  // SecurityException
using Microsoft.Win32;                  // Registry

namespace LocalAgent.Features.FirstRun
{
    // Marks the whole class as Windows-only. This tells the platform analyzer (CA1416)
    // that the Windows APIs used below are intentional.
    [SupportedOSPlatform("windows")]
    public static class MachineScanner
    {
        // Converts bytes to gigabytes (1024 * 1024 * 1024 bytes).
        private const double Gb = 1024d * 1024d * 1024d;

        // Calls one reader per field and assembles the result.
        // Each reader records its outcome in "sources".
        public static MachineInfo Scan(string modelDir)
        {
            var sources = new Dictionary<string, string>();

            var osName = ReadOsName(sources);
            var cpuName = ReadCpuName(sources);
            var cpuCores = ReadCpuCores(sources);
            var totalRamGb = ReadTotalRamGb(sources);

            // Deconstructs the four values returned by ReadGpu.
            var (gpuName, vramGb, vramIsLowerBound, hasNvidiaSmi) = ReadGpu(sources);

            var freeDiskGb = ReadFreeDiskGb(modelDir, sources);
            var llamaServerPath = FindLlamaServer(sources);
            var portsInUse = ReadListeningPorts(sources);
            var modelFiles = ReadModelFiles(modelDir, sources);

            return new MachineInfo(osName, cpuName, cpuCores, totalRamGb, gpuName, vramGb,
                vramIsLowerBound, freeDiskGb, hasNvidiaSmi, llamaServerPath, portsInUse, modelFiles, sources);
        }

        // ---------- Operating system ----------

        private static string? ReadOsName(Dictionary<string, string> sources)
        {
            // This property never throws, so no try/catch is needed.
            sources["osName"] = "RuntimeInformation.OSDescription";
            return RuntimeInformation.OSDescription;
        }

        // ---------- CPU ----------

        private static string? ReadCpuName(Dictionary<string, string> sources)
        {
            try
            {
                // Registry.GetValue returns object?, so "as string" converts it.
                // A missing value gives null.
                var value = Registry.GetValue(
                    @"HKEY_LOCAL_MACHINE\HARDWARE\DESCRIPTION\System\CentralProcessor\0",
                    "ProcessorNameString", null) as string;

                sources["cpuName"] = "registry ProcessorNameString";
                return value;
            }
            catch (SecurityException ex)
            {
                // Thrown when the registry cannot be read due to permissions.
                sources["cpuName"] = $"failed: {ex.Message}";
                return null;
            }
        }

        private static int? ReadCpuCores(Dictionary<string, string> sources)
        {
            // Logical processor count, including hyperthreads.
            sources["cpuCores"] = "Environment.ProcessorCount (logical)";
            return Environment.ProcessorCount;
        }

        // ---------- RAM ----------

        private static double? ReadTotalRamGb(Dictionary<string, string> sources)
        {
            try
            {
                using var searcher = new ManagementObjectSearcher(
                    "SELECT TotalPhysicalMemory FROM Win32_ComputerSystem");

                // One ComputerSystem object exists, so this loop body runs once.
                foreach (ManagementObject obj in searcher.Get())
                {
                    // WMI returns the value as object, so convert it to a ulong (bytes).
                    var bytes = Convert.ToUInt64(obj["TotalPhysicalMemory"]);

                    sources["totalRamGb"] = "WMI Win32_ComputerSystem.TotalPhysicalMemory";
                    return bytes / Gb;
                }

                // The query returned no rows.
                sources["totalRamGb"] = "WMI returned no rows";
                return null;
            }
            catch (ManagementException ex)
            {
                // WMI query failed, for example because the WMI service is unavailable.
                sources["totalRamGb"] = $"WMI failed: {ex.Message}";
                return null;
            }
        }

        // ---------- GPU ----------

        // Returns four values: name, VRAM in GB, whether VRAM is only a lower bound, and whether nvidia-smi was used.
        private static (string? Name, double? VramGb, bool VramIsLowerBound, bool HasNvidiaSmi) ReadGpu(
            Dictionary<string, string> sources)
        {
            // nvidia-smi gives an exact VRAM figure, so try it first.
            var smi = ReadGpuFromNvidiaSmi(sources);
            if (smi is { } gpu)
                return (gpu.Name, gpu.VramGb, false, true);

            // Fallback: the registry. Its VRAM value is not marked as a lower bound here.
            var (name, vram) = ReadGpuFromRegistry(sources);
            return (name, vram, false, false);
        }

        private static (string Name, double VramGb)? ReadGpuFromNvidiaSmi(Dictionary<string, string> sources)
        {
            try
            {
                var psi = new ProcessStartInfo("nvidia-smi",
                    "--query-gpu=name,memory.total --format=csv,noheader,nounits")
                {
                    RedirectStandardOutput = true,  // capture the program's output
                    UseShellExecute = false,        // required when redirecting output
                    CreateNoWindow = true           // no console window
                };

                using var process = Process.Start(psi);
                if (process is null)
                {
                    sources["gpu"] = "nvidia-smi did not start";
                    return null;
                }

                string output = process.StandardOutput.ReadToEnd();

                // Wait at most 5 seconds so the scan cannot hang.
                process.WaitForExit(5000);

                if (process.ExitCode != 0)
                {
                    sources["gpu"] = $"nvidia-smi exit code {process.ExitCode}";
                    return null;
                }

                // Output for one GPU: "NVIDIA GeForce RTX 4090, 24564"
                // Only the first line is used, so this reads the first GPU.
                var parts = output.Split('\n')[0].Split(',');
                var name = parts[0].Trim();

                // nvidia-smi reports MiB. InvariantCulture keeps parsing independent of locale.
                var mib = double.Parse(parts[1].Trim(), CultureInfo.InvariantCulture);

                sources["gpu"] = "nvidia-smi";
                return (name, mib / 1024d);
            }
            catch (Win32Exception ex)
            {
                // Process.Start throws this when nvidia-smi is not installed.
                sources["gpu"] = $"nvidia-smi not found: {ex.Message}";
                return null;
            }
        }

        private static (string? Name, double? VramGb) ReadGpuFromRegistry(Dictionary<string, string> sources)
        {
            // Display adapter class key. Subkey 0000 is usually the primary GPU.
            const string keyPath =
                @"SYSTEM\CurrentControlSet\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000";

            try
            {
                using var key = Registry.LocalMachine.OpenSubKey(keyPath);
                if (key is null)
                {
                    sources["gpu"] = "registry display adapter key not found";
                    return (null, null);
                }

                var name = key.GetValue("DriverDesc") as string;

                // VRAM is stored in bytes as a 64-bit number under this value name.
                double? vram = key.GetValue("HardwareInformation.qwMemorySize") is long bytes
                    ? bytes / Gb
                    : null;

                sources["gpu"] = "registry display class 0000";
                return (name, vram);
            }
            catch (Exception ex) when (ex is SecurityException or UnauthorizedAccessException)
            {
                sources["gpu"] = $"registry failed: {ex.Message}";
                return (null, null);
            }
        }

        // ---------- Disk ----------

        private static double? ReadFreeDiskGb(string modelDir, Dictionary<string, string> sources)
        {
            try
            {
                // Finds the drive that holds the model folder, for example "D:\".
                var drive = new DriveInfo(Path.GetPathRoot(modelDir)!);

                sources["freeDiskGb"] = "DriveInfo.AvailableFreeSpace";
                return drive.AvailableFreeSpace / Gb;
            }
            catch (IOException ex)
            {
                // Thrown when the drive is not ready, or is missing.
                sources["freeDiskGb"] = $"failed: {ex.Message}";
                return null;
            }
        }

        // ---------- llama-server ----------

        private static string? FindLlamaServer(Dictionary<string, string> sources)
        {
            // PATH is a list of folders separated by ';' on Windows.
            var dirs = (Environment.GetEnvironmentVariable("PATH") ?? string.Empty)
                .Split(Path.PathSeparator, StringSplitOptions.RemoveEmptyEntries);

            // Returns the first folder that contains llama-server.exe, or null.
            var path = dirs
                .Select(d => Path.Combine(d.Trim('"'), "llama-server.exe"))
                .FirstOrDefault(File.Exists);

            sources["llamaServerPath"] = path is null ? "not found on PATH" : "searched PATH";
            return path;
        }

        // ---------- Network ----------

        private static IReadOnlyList<int> ReadListeningPorts(Dictionary<string, string> sources)
        {
            try
            {
                // Lists TCP endpoints that are accepting connections, then keeps unique, sorted port numbers.
                var ports = IPGlobalProperties.GetIPGlobalProperties()
                    .GetActiveTcpListeners()
                    .Select(ep => ep.Port)
                    .Distinct()
                    .OrderBy(p => p)
                    .ToList();

                sources["portsInUser"] = "IPGlobalProperties.GetActiveTcpListeners";
                return ports;
            }
            catch (NetworkInformationException ex)
            {
                sources["portsInUser"] = $"failed: {ex.Message}";
                return Array.Empty<int>();
            }
        }

        // ---------- Model files ----------

        private static IReadOnlyList<string> ReadModelFiles(string modelDir, Dictionary<string, string> sources)
        {
            try
            {
                // ToList() forces the lazy enumeration to run inside the try block.
                var files = Directory.EnumerateFiles(modelDir, "*.gguf").ToList();

                sources["modelFiles"] = "Directory.EnumerateFiles *.gguf";
                return files;
            }
            catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
            {
                // Covers a missing folder and a folder without permission.
                sources["modelFiles"] = $"failed: {ex.Message}";
                return Array.Empty<string>();
            }
        }
    }
}