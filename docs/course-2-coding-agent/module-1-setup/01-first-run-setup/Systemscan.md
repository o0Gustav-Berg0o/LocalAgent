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

        // The full scan. Calls one reader per fact and assembles the result.
        // Each reader records its outcome in "sources".
        //
        // mainFolder is the main folder the wizard already knows: the saved one, or the one from
        // the earlier scan. On a first run nobody has chosen one yet, so it is null. Never pass a
        // default as if the user had chosen it (README section 5.13). With null, the agent's own
        // models and llama.cpp folders are skipped; CheckMainFolder adds them after the question.
        //
        // progress gets one tick line per probe. The caller prints it, so this class never calls
        // Console (README section 3.10, rule 3). The building blocks below do not report progress.
        public static MachineInfo Scan(string? mainFolder, Action<string>? progress = null)
        {
            var sources = new Dictionary<string, string>();

            // "[ok]" when a value was read, "[--]" when it was not, then the value and its source.
            void Tick(string key, bool found, string text) =>
                progress?.Invoke($"[{(found ? "ok" : "--")}] {text} ({sources.GetValueOrDefault(key, "no source")})");

            var osName = ReadOsName(sources);
            Tick("osName", osName is not null, $"OS: {osName ?? "unknown"}");

            var cpuName = ReadCpuName(sources);
            var cpuCores = ReadCpuCores(sources);
            Tick("cpuName", cpuName is not null, $"CPU: {cpuName ?? "unknown"}, {cpuCores} logical cores");

            var totalRamGb = ReadTotalRamGb(sources);
            Tick("totalRamGb", totalRamGb is not null, $"RAM: {(totalRamGb is { } ram ? $"{ram:F0} GB" : "unknown")}");

            // Deconstructs the four values returned by ReadGpu.
            var (gpuName, vramGb, vramIsLowerBound, hasNvidiaSmi) = ReadGpu(sources);
            Tick("gpu", gpuName is not null,
                $"GPU: {gpuName ?? "none found"}, VRAM {(vramGb is { } vram ? $"{vram:F1} GB" : "unknown")}");

            // Every drive, because the main folder may not be chosen yet.
            var freeDisk = FreeDiskByDrive(sources);
            Tick("freeDiskGb", freeDisk.Count > 0,
                $"free disk: {string.Join(", ", freeDisk.Select(d => $"{d.Key.TrimEnd('\\')} {d.Value:F0} GB"))}");

            var llamaServerPath = FindLlamaServer(mainFolder, sources);
            Tick("llamaServerPath", llamaServerPath is not null, $"llama-server: {llamaServerPath ?? "not found"}");

            var portsInUse = ReadListeningPorts(sources);
            Tick("portsInUse", true, $"ports in use: {portsInUse.Count}");

            var modelFiles = FindModels(ModelFolders(mainFolder), sources);
            Tick("modelFiles", modelFiles.Count > 0, $"model files: {modelFiles.Count}");

            var ollamaFolder = FindOllamaFolder(sources);
            Tick("ollamaFolder", ollamaFolder is not null, $"Ollama: {(ollamaFolder is null ? "not found" : "found")}");

            return new MachineInfo(osName, cpuName, cpuCores, totalRamGb, gpuName, vramGb,
                vramIsLowerBound, freeDisk, hasNvidiaSmi, llamaServerPath, portsInUse, modelFiles,
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

        // ======================================================================
        // Building blocks. Public, so the wizard and later lessons can ask one question
        // without a full scan (README section 3.4): after the main-folder question, when
        // the user types a folder, or in a later lesson. "sources" is optional for that
        // reason: a caller that does not show sources passes nothing.
        // ======================================================================

        // ---------- Disk ----------

        // Free GB per drive root ("C:\"), for every fixed drive that is ready.
        // Every drive, because on a first run the main folder is not chosen yet (README section 5.13).
        public static IReadOnlyDictionary<string, double> FreeDiskByDrive(IDictionary<string, string>? sources = null)
        {
            var free = new Dictionary<string, double>(StringComparer.OrdinalIgnoreCase);

            foreach (var drive in DriveInfo.GetDrives())
            {
                try
                {
                    // IsReady is false for an empty card reader or a disconnected drive.
                    if (drive.DriveType == DriveType.Fixed && drive.IsReady)
                        free[drive.Name] = drive.AvailableFreeSpace / Gb;
                }
                catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
                {
                    // A drive that fails while it is read is left out. The others still count.
                }
            }

            Note(sources, "freeDiskGb", free.Count == 0
                ? "no fixed drive could be read"
                : "DriveInfo.AvailableFreeSpace, per drive");
            return free;
        }

        // The free GB on the drive that holds folder, from the scan's per-drive list.
        // null if that drive was not read, for example a network drive.
        // The comparison ignores case, because a list read back from JSON has lost its comparer.
        public static double? FreeDiskGbFor(MachineInfo info, string folder)
        {
            var root = Path.GetPathRoot(Path.GetFullPath(folder));
            if (root is null)
                return null;

            foreach (var (drive, gb) in info.FreeDiskGbByDrive)
            {
                if (string.Equals(drive, root, StringComparison.OrdinalIgnoreCase))
                    return gb;
            }

            return null;
        }

        // ---------- llama-server ----------

        // PATH first, then the main folder's llama.cpp folder when a main folder is known. That is
        // where the install guide tells the user to unzip it. Checks that the file exists. Never
        // runs it (README section 3.4). Anywhere else, the user types the path in InstalledPrompt.
        public static string? FindLlamaServer(string? mainFolder, IDictionary<string, string>? sources = null)
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
                Note(sources, "llamaServerPath", "PATH");
                return onPath;
            }

            if (mainFolder is not null && File.Exists(LlamaServerIn(mainFolder)))
            {
                Note(sources, "llamaServerPath", "main folder");
                return LlamaServerIn(mainFolder);
            }

            Note(sources, "llamaServerPath", mainFolder is null
                ? "not found on PATH"
                : "not found on PATH or in the main folder");
            return null;
        }

        // Where the install guide tells the user to unzip llama-server.
        private static string LlamaServerIn(string mainFolder) =>
            Path.Combine(mainFolder, "llama.cpp", "llama-server.exe");

        // ---------- Model files ----------

        // The known model folders from README section 3.4, as (source name, folder), in the
        // order of that table. The agent's own models folder only when a main folder is known.
        // A variable that is not set is left out, and so is a folder already in the list.
        // The folders do not have to exist: FindModels skips the missing ones.
        // InstalledPrompt lists them when nothing was found.
        public static IReadOnlyList<(string Source, string Folder)> ModelFolders(string? mainFolder)
        {
            var profile = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
            var localAppData = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
            var hfHome = Env("HF_HOME");

            var candidates = new (string Source, string? Folder)[]
            {
                ("this agent", mainFolder is null ? null : Path.Combine(mainFolder, "models")),
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

        // Every *.gguf file in each of these folders that exists, including its subfolders.
        // The scan passes ModelFolders(...). ScanRows.Apply passes one folder the user typed,
        // with source "you". CheckMainFolder passes the main folder's models folder.
        // FoundModel is defined next to MachineInfo (Exercise 2, Part B).
        public static IReadOnlyList<FoundModel> FindModels(
            IEnumerable<(string Source, string Folder)> folders, IDictionary<string, string>? sources = null)
        {
            var found = new List<FoundModel>();
            var searched = new List<string>();

            // Two folder names can lead to the same file, so each file is listed once.
            var seenFiles = new HashSet<string>(StringComparer.OrdinalIgnoreCase);

            // Search subfolders, and skip a subfolder that cannot be read instead of stopping.
            // AttributesToSkip = 0 also lists hidden files; the default skips them.
            // The full name is needed: System.Management has an EnumerationOptions class too.
            var options = new System.IO.EnumerationOptions
            {
                RecurseSubdirectories = true,
                IgnoreInaccessible = true,
                AttributesToSkip = 0
            };

            foreach (var (source, folder) in folders)
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

                        if (seenFiles.Add(path))
                            found.Add(new FoundModel(path, RealSizeGb(path), source));
                    }

                    searched.Add(folder);
                }
                catch (Exception ex) when (ex is IOException or UnauthorizedAccessException)
                {
                    // One unreadable folder does not stop the others.
                    Note(sources, $"modelFiles {folder}", $"failed: {ex.Message}");
                }
            }

            Note(sources, "modelFiles", searched.Count == 0
                ? "no known model folder exists"
                : $"searched {string.Join(", ", searched)}");
            return found;
        }

        // The size of the real file, in GB, or null if it cannot be read.
        // In the Hugging Face cache, each .gguf file is a link into a "blobs" folder, and the
        // link itself has a length of 0 (README section 5.12). So follow the link first.
        // Public, because ScanRows.Apply needs it for a single file the user typed.
        public static double? RealSizeGb(string path)
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

        // ---------- Ollama ----------

        // Ollama stores its models as blobs without model names, so they are not listed
        // (README section 3.4). Only the folder is reported, so the wizard can say why.
        public static string? FindOllamaFolder(IDictionary<string, string>? sources = null)
        {
            var folder = Env("OLLAMA_MODELS")
                ?? Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.UserProfile), ".ollama", "models");

            if (Directory.Exists(folder))
            {
                Note(sources, "ollamaFolder", "found; its models are not listed");
                return folder;
            }

            Note(sources, "ollamaFolder", "not found");
            return null;
        }

        // ---------- After the main-folder question ----------

        // Checks only the chosen main folder, without a full scan: its models folder, and its
        // llama.cpp folder when no llama-server is known yet. Returns the facts with what it found
        // added, and one line per addition for the wizard to print. A model already in the list
        // is not added again, so calling it twice adds nothing the second time.
        public static (MachineInfo Info, IReadOnlyList<string> Added) CheckMainFolder(MachineInfo info, string mainFolder)
        {
            var added = new List<string>();

            var known = info.ModelFiles.Select(m => m.Path).ToHashSet(StringComparer.OrdinalIgnoreCase);
            var newModels = FindModels([("this agent", Path.Combine(mainFolder, "models"))])
                .Where(m => !known.Contains(m.Path))
                .ToList();

            foreach (var model in newModels)
                added.Add($"model {Path.GetFileName(model.Path)}, found in the main folder");

            var llamaServerPath = info.LlamaServerPath;
            if (llamaServerPath is null && File.Exists(LlamaServerIn(mainFolder)))
            {
                llamaServerPath = LlamaServerIn(mainFolder);
                added.Add($"llama-server, found in {Path.GetDirectoryName(llamaServerPath)}");
            }

            var updated = info with
            {
                ModelFiles = [.. info.ModelFiles, .. newModels],
                LlamaServerPath = llamaServerPath
            };

            return (updated, added);
        }

        // ---------- Helpers ----------

        // Records a source when the caller wants sources. A building block called on its own
        // gets null and records nothing.
        private static void Note(IDictionary<string, string>? sources, string key, string value)
        {
            if (sources is not null)
                sources[key] = value;
        }

        // An environment variable's value, or null if it is not set or is empty.
        private static string? Env(string name) =>
            Environment.GetEnvironmentVariable(name) is { Length: > 0 } value ? value : null;
    }
}