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
        // mainFolder is the current config's main folder (on a first run, %USERPROFILE%\LocalAgent).
        // The scan reads its drive, its models folder, and its llama.cpp folder.
        public static MachineInfo Scan(string mainFolder)
        {
            var sources = new Dictionary<string, string>();

            var osName = ReadOsName(sources);
            var cpuName = ReadCpuName(sources);
            var cpuCores = ReadCpuCores(sources);
            var totalRamGb = ReadTotalRamGb(sources);

            // Deconstructs the four values returned by ReadGpu.
            var (gpuName, vramGb, vramIsLowerBound, hasNvidiaSmi) = ReadGpu(sources);

            var freeDiskGb = ReadFreeDiskGb(mainFolder, sources);
            var llamaServerPath = FindLlamaServer(mainFolder, sources);
            var portsInUse = ReadListeningPorts(sources);
            var modelFiles = ReadModelFiles(mainFolder, sources);
            var ollamaFolder = FindOllamaFolder(sources);

            return new MachineInfo(osName, cpuName, cpuCores, totalRamGb, gpuName, vramGb,
                vramIsLowerBound, freeDiskGb, hasNvidiaSmi, llamaServerPath, portsInUse, modelFiles,
                ollamaFolder, sources);
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

                // Start reading before waiting. ReadToEnd on its own blocks until the program
                // exits, so a nvidia-smi that hangs would hang the scan before the timeout applies.
                var outputTask = process.StandardOutput.ReadToEndAsync();

                // Wait at most 5 seconds so the scan cannot hang.
                if (!process.WaitForExit(5000))
                {
                    process.Kill(entireProcessTree: true);
                    sources["gpu"] = "nvidia-smi timed out after 5 seconds";
                    return null;
                }

                string output = outputTask.GetAwaiter().GetResult();

                if (process.ExitCode != 0)
                {
                    sources["gpu"] = $"nvidia-smi exit code {process.ExitCode}";
                    return null;
                }

                // Output for one GPU: "NVIDIA GeForce RTX 4090, 24564"
                // Only the first line is used, so this reads the first GPU.
                var parts = output.Split('\n')[0].Split(',');

                // Parse, do not trust: some drivers print "[Not Supported]" or "[N/A]".
                // nvidia-smi reports MiB. InvariantCulture keeps parsing independent of locale.
                if (parts.Length < 2 ||
                    !double.TryParse(parts[1].Trim(), NumberStyles.Float, CultureInfo.InvariantCulture, out var mib))
                {
                    sources["gpu"] = $"nvidia-smi output not understood: {output.Trim()}";
                    return null;
                }

                var name = parts[0].Trim();

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

        private static double? ReadFreeDiskGb(string mainFolder, Dictionary<string, string> sources)
        {
            try
            {
                // Finds the drive that holds the main folder, for example "D:\".
                // Downloaded models go to its models folder, so this is the drive that must have room.
                var drive = new DriveInfo(Path.GetPathRoot(mainFolder)!);

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

        // PATH first, then the main folder's llama.cpp folder, where the install guide tells the
        // user to unzip it. Checks that the file exists. Never runs it (README section 3.4).
        // Anywhere else, the user types the path in InstalledPrompt (Exercise 2, Part F).
        private static string? FindLlamaServer(string mainFolder, Dictionary<string, string> sources)
        {
            // PATH is a list of folders separated by ';' on Windows.
            var dirs = (Environment.GetEnvironmentVariable("PATH") ?? string.Empty)
                .Split(Path.PathSeparator, StringSplitOptions.RemoveEmptyEntries);

            // The first folder that contains llama-server.exe, or null.
            var onPath = dirs
                .Select(d => Path.Combine(d.Trim('"'), "llama-server.exe"))
                .FirstOrDefault(File.Exists);

            if (onPath is not null)
            {
                sources["llamaServerPath"] = "PATH";
                return onPath;
            }

            var inMainFolder = Path.Combine(mainFolder, "llama.cpp", "llama-server.exe");
            if (File.Exists(inMainFolder))
            {
                sources["llamaServerPath"] = "main folder";
                return inMainFolder;
            }

            sources["llamaServerPath"] = "not found on PATH or in the main folder";
            return null;
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

                sources["portsInUse"] = "IPGlobalProperties.GetActiveTcpListeners";
                return ports;
            }
            catch (NetworkInformationException ex)
            {
                sources["portsInUse"] = $"failed: {ex.Message}";
                return Array.Empty<int>();
            }
        }

        // ---------- Model files ----------

        // The known model folders from README section 3.4, as (source name, folder), in the
        // order of that table. A variable that is not set is left out, and so is a folder
        // already in the list. The folders do not have to exist: ReadModelFiles skips the
        // missing ones. Public, because InstalledPrompt lists them when nothing was found.
        public static IReadOnlyList<(string Source, string Folder)> ModelFolders(string mainFolder)
        {
            var profile = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
            var localAppData = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
            var hfHome = Env("HF_HOME");

            var candidates = new (string Source, string? Folder)[]
            {
                ("this agent", Path.Combine(mainFolder, "models")),
                // llama-server -hf downloads. Newer builds use the Hugging Face cache,
                // older builds their own folder. LLAMA_CACHE overrides both.
                ("llama.cpp", Env("LLAMA_CACHE")),
                ("llama.cpp", Env("HF_HUB_CACHE")),
                ("llama.cpp", hfHome is null ? null : Path.Combine(hfHome, "hub")),
                ("llama.cpp", Path.Combine(profile, ".cache", "huggingface", "hub")),
                ("llama.cpp", Path.Combine(localAppData, "llama.cpp")),
                // LM Studio: the current folder, then the one older versions used.
                ("LM Studio", Path.Combine(profile, ".lmstudio", "models")),
                ("LM Studio", Path.Combine(profile, ".cache", "lm-studio", "models")),
            };

            // Folder names on Windows ignore case, so "C:\Models" and "c:\models\" are one folder.
            var seen = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
            var folders = new List<(string Source, string Folder)>();
            foreach (var (source, folder) in candidates)
            {
                if (folder is null)
                    continue;

                var trimmed = folder.TrimEnd('\\', '/');
                if (seen.Add(trimmed))
                    folders.Add((source, trimmed));
            }

            return folders;
        }

        // Every *.gguf file in each known folder that exists, including its subfolders.
        // FoundModel is defined next to MachineInfo (Exercise 2, Part B).
        private static IReadOnlyList<FoundModel> ReadModelFiles(string mainFolder, Dictionary<string, string> sources)
        {
            var found = new List<FoundModel>();
            var searched = new List<string>();

            // Search subfolders, and skip a subfolder that cannot be read instead of stopping.
            // AttributesToSkip = 0 also lists hidden files; the default skips them.
            // The full name is needed: System.Management has an EnumerationOptions class too.
            var options = new System.IO.EnumerationOptions
            {
                RecurseSubdirectories = true,
                IgnoreInaccessible = true,
                AttributesToSkip = 0
            };

            foreach (var (source, folder) in ModelFolders(mainFolder))
            {
                // A missing folder is normal: that tool is not installed.
                // Directory.EnumerateFiles on a missing folder throws, so check first. Do not
                // use Get-ChildItem -Recurse here: on a missing folder it searches the parent.
                if (!Directory.Exists(folder))
                    continue;

                try
                {
                    // ToList() forces the lazy enumeration to run inside the try block.
                    var paths = Directory.EnumerateFiles(folder, "*.gguf", options).ToList();

                    foreach (var path in paths)
                    {
                        // mmproj files are vision add-ons for a model, not models (README section 5.12).
                        if (Path.GetFileName(path).StartsWith("mmproj", StringComparison.OrdinalIgnoreCase))
                            continue;

                        found.Add(new FoundModel(path, ReadRealSizeGb(path), source));
                    }

                    searched.Add(folder);
                }
                catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
                {
                    // One unreadable folder does not stop the others.
                    sources[$"modelFiles {folder}"] = $"failed: {ex.Message}";
                }
            }

            sources["modelFiles"] = searched.Count == 0
                ? "no known model folder exists"
                : $"searched {string.Join(", ", searched)}";
            return found;
        }

        // The size of the real file, in GB, or null if it cannot be read.
        // In the Hugging Face cache, each .gguf file is a link into a "blobs" folder, and the
        // link itself has a length of 0 (README section 5.12). So follow the link first.
        private static double? ReadRealSizeGb(string path)
        {
            try
            {
                var file = new FileInfo(path);

                // ResolveLinkTarget returns null when the file is not a link.
                // returnFinalTarget: true follows a chain of links to the real file.
                var target = file.ResolveLinkTarget(returnFinalTarget: true) as FileInfo ?? file;

                return target.Exists ? target.Length / Gb : null;
            }
            catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
            {
                // A broken link, or a target the app may not read.
                return null;
            }
        }

        // Ollama stores its models as blobs without model names, so they are not listed
        // (README section 3.4). Only the folder is reported, so the wizard can say why.
        private static string? FindOllamaFolder(Dictionary<string, string> sources)
        {
            var folder = Env("OLLAMA_MODELS")
                ?? Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.UserProfile), ".ollama", "models");

            if (Directory.Exists(folder))
            {
                sources["ollamaFolder"] = "found; its models are not listed";
                return folder;
            }

            sources["ollamaFolder"] = "not found";
            return null;
        }

        // An environment variable's value, or null if it is not set or is empty.
        private static string? Env(string name) =>
            Environment.GetEnvironmentVariable(name) is { Length: > 0 } value ? value : null;
    }
}