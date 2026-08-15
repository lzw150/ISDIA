<p align="center" style="margin-bottom: 0px !important;">
   <img width="120" height="127" alt="ISDIA软件logo1_lowsize" src="https://github.com/user-attachments/assets/3e7779ef-02c3-4482-b352-0aa9e3040d98" />
</p>

# ISDIA v2.0 - DIA Mass Spectrometry Detection Tool

## Overview

ISDIA is a professional tool for detecting DIA (Data-Independent Acquisition) mode in mass spectrometry data files.

**v2.0 New Features**:
- ✅ Direct support for **Thermo .raw** files (via msconvert)
- ✅ Direct support for **SCIEX .wiff** files (via msconvert)
- ✅ New **.mzXML** format support (built-in XML parser)
- ✅ Pluggable reader architecture — easy to add more formats
- ✅ Self-contained distribution — copy and run, no Python installation needed

## Supported Formats

| Format | Extension | Reader | Platform |
|--------|-----------|--------|----------|
| mzML | `.mzML` | Built-in XML | Windows / Linux |
| mzXML | `.mzXML`, `.xml` | Built-in XML | Windows / Linux |
| Bruker timsTOF | `.d`, `.d.zip` | Built-in SQLite | Windows / Linux |
| Thermo RAW | `.raw` | msconvert (ProteoWizard) | Windows / Linux |
| SCIEX WIFF | `.wiff`, `.wiff2` | msconvert (ProteoWizard) | Windows / Linux |

Check format availability:
```bash
ISDIA --list-formats
```

## Quick Start

### Just Copy and Use! (Windows)

The release package contains both versions — pick whichever you prefer:

1. **Download**: Get the latest release from [GitHub Releases](https://github.com/guomics-lab/ISDIA/releases)
2. **Extract**: Unzip to any folder
3. **Run**: 
   - **GUI**: Double-click `ISDIA_gui.exe` (graphical interface, recommended)
   - **CLI**: Open terminal, run `ISDIA.exe <input>` (command line, suitable for batch processing)
4. **Done** — no Python, no installation required!

> **Note**: `.raw` / `.wiff` / `.wiff2` support requires external converter tools. See [Converter Setup](#converter-setup-for-raw--wiff) below to configure them.
>
> If the target computer shows an error about missing `msvcp140.dll` or `vcruntime140.dll`, install the [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe) (most Windows 10/11 machines already have this).
>
> **Temporary files**: When reading `.raw`, `.wiff`, or `.wiff2` files, ISDIA temporarily converts them to `.mzML` format internally. These temporary files are **automatically deleted** immediately after processing — no cleanup needed, no disk space wasted.

### Windows GUI
<img width="2350" height="1469" alt="GUI_shortcut" src="https://github.com/user-attachments/assets/026bf8fa-0e67-489a-a402-a24059b4a68d" />

### CLI Usage (Windows Command Line)

```bash
# Folder — auto-scan all supported files
ISDIA D:\data\raw_files

# Glob pattern (use quotes around the path!)
ISDIA "D:\data\*.raw"
ISDIA "D:\data\*.d.zip"

# File list (.txt)
ISDIA file_list.txt

# Single file
ISDIA D:\data\sample.raw

# Custom output
ISDIA D:\data --output results.csv

# With custom parameters
ISDIA D:\data --max-ms2 15 --threshold 5

# Disable Pulse DIA detection
ISDIA D:\data --no-pulse-dia

# Specify converter directory (default: ./converters/)
ISDIA D:\data --converter-dir "D:\Program Files\ProteoWizard"

# List supported formats
ISDIA --list-formats
```

### File List Format

Create a text file with one file path per line:
```
sample001.mzML
sample002.mzXML
sample003.d
sample004.d.zip
sample005.raw
sample006.wiff
sample007.wiff2
D:\data\sample008.raw
```

**Generate file_list.txt from command line:**

Windows (PowerShell):
```powershell
# All supported files in a folder
Get-ChildItem D:\data -Recurse -Include *.mzML,*.mzXML,*.d,*.raw,*.wiff,*.wiff2 | ForEach-Object FullName > file_list.txt

# or using CMD
dir /b /s D:\data\*.mzML D:\data\*.raw > file_list.txt
```

Linux:
```bash
# All supported files in a folder
find /d/west -type f \( -name "*.mzML" -o -name "*.mzXML" -o -name "*.raw" -o -name "*.wiff" -o -name "*.wiff2" \) > file_list.txt

# timsTOF .d folders
find /d/west -type d -name "*.d" > file_list.txt
```

> **Tip**: In most cases you don't need to create `file_list.txt` manually. Just pass the folder path directly — ISDIA scans it automatically.

### Output Format

CSV file with columns:
- `FileName`: Input file name
- `FileExists`: Whether the file was found
- `MS2Count`: Number of MS2 scans detected
- `MS2isoContinue`: Number of consecutive isolation windows
- `ISDIA`: DIA detection result (Yes / No / Unknown)
- `Format`: File format detected

## Algorithm Details

ISDIA determines DIA mode by analyzing the isolation window patterns of MS2 scans:

1. **MS2 Scan Collection**: Extract the first N MS2 scans from the file
2. **Isolation Window Extraction**: Get target m/z, lower offset, upper offset
3. **Continuity Check**: Check if consecutive windows satisfy:
   - `targetMZ(i) + 1 <= targetMZ(i+1)`
   - `iso_upper(i) + tolerance >= iso_lower(i+1)`
4. **Classification**: >= 3 consecutive windows → DIA; otherwise → DDA
5. **Pulse DIA** (optional): Check for periodic window patterns with 3 equal m/z differences

### Converter Setup for .raw / .wiff

**Windows**: Install [ProteoWizard](https://proteowizard.sourceforge.io/download.html), then copy **all files** in the installation directory (including `msconvert.exe`, vendor DLLs and other dependencies) to `converters/msconvert/`. Alternatively, use `--converter-dir` to point to the ProteoWizard installation path directly.

**Linux (cluster without sudo)**:

Step 1 — install a container runtime (ask your admin, or install yourself via conda):

```bash
# Apptainer (Singularity successor) via conda
conda install -c conda-forge apptainer -y

# or the classic SingularityCE
conda install -c conda-forge singularityce -y

# verify
which apptainer || which singularity
```

> **Note**: If neither is available, alternatives are (a) `conda install -c conda-forge wine` to run msconvert.exe under Wine, or (b) pre-convert `.raw`/`.wiff` to `.mzML` on a Windows machine.

Step 2 — build the ProteoWizard Singularity container from Docker Hub:

```bash
singularity pull pwiz.sif docker://proteowizard/pwiz-skyline-i-agree-to-the-vendor-licenses
```

- This downloads `pwiz.sif` (~2 GB). Place it in `converters/msconvert/`. ISDIA auto-detects it — no `--msconvert-container` flag needed. (Explicit flag still works and takes priority.)

### Linux Usage

The Linux binary is self-contained (no Python/pip required).

```bash
# Make executable
chmod +x ISDIA

# Folder — auto-scan all supported files
./ISDIA /data/raw_files

# Glob pattern (use quotes!)
./ISDIA "/data/*.raw"
./ISDIA "/data/*.d.zip"

# File list (.txt)
./ISDIA file_list.txt

# Custom output and parameters
./ISDIA /data/raw_files --output results.csv --max-ms2 15 --threshold 5

# Disable Pulse DIA detection
./ISDIA /data/raw_files --no-pulse-dia

# With Singularity container (cluster without sudo) — auto-detected from converters/
./ISDIA /data/raw_files

# Explicit container path (optional, overrides auto-detection)
./ISDIA --msconvert-container ./converters/msconvert/pwiz.sif /data/raw_files

# List supported formats
./ISDIA --list-formats
```

## Performance

| Operation | Speed |
|-----------|-------|
| .mzML parsing | ~1-2s / file (depends on file size) |
| .d (timsTOF) | ~0.1s / file (SQLite query) |
| .raw (msconvert, progressive) | ~2-10s / file (first 50 scans) |
| .wiff (msconvert, progressive) | ~2-10s / file (first 50 scans) |

## Troubleshooting

### "Converter not found" error

Use `--list-formats` to check converter status. Install missing tools as described above.

### Missing DLL error (msvcp140.dll / vcruntime140.dll)

Install [Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe). Most Windows 10/11 systems already include these.

### .raw file reads slowly

ISDIA uses progressive scan range (50→500→5000→50000). If all limits are reached, the file may have very few MS2 scans.

### .raw / .wiff on Linux

Options:
1. **Container (recommended)** — install Apptainer/Singularity (`conda install -c conda-forge apptainer`), download the `.sif` file, use `--msconvert-container`. No sudo required.
2. **Wine** — `sudo apt install wine` (or `conda install -c conda-forge wine`), then copy `msconvert.exe` and vendor DLLs from a Windows ProteoWizard installation into `converters/msconvert/`.

Or pre-convert on a Windows machine.

### Temporary files filling disk

No need to worry — all temporary `.mzML` files from `.raw`/`.wiff`/`.wiff2` conversion are **automatically deleted** by ISDIA immediately after processing. If a crash occurs mid-conversion, leftover temp files can be found in the system temp directory (usually `%TEMP%`) under names starting with `isdia_` — you can safely delete them manually.

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--output`, `-o` | `isdia_results.csv` | Output CSV file path. |
| `--max-ms2` | 11 | Maximum number of MS2 scans to extract. The default is an empirical value sufficient for acquisition mode judgment; larger values increase processing time. |
| `--threshold` | 3 | Number of consecutive isolation windows required to classify as DIA. Increase to reduce false positives. |
| `--tolerance` | 0.1 | m/z tolerance for comparing adjacent isolation window boundaries. Larger values are more permissive. |
| `--pulse-dia` | on | Pulse DIA detection mode. Runs automatically as a secondary check; has no effect on standard DIA detection. |
| `--no-pulse-dia` | — | Disable Pulse DIA detection. |
| `--pulse-tol` | 0.01 | Tolerance for detecting equal m/z differences in Pulse DIA mode. Smaller values require stricter periodicity. |
| `--converter-dir` | `./converters/` | Path to msconvert.exe and vendor DLLs (Windows). |
| `--msconvert-container` | — | Path to Singularity .sif container with Wine + msconvert (Linux clusters). |
| `--list-formats` | — | Display all supported file formats and converter availability. |

### Parameter Guidance

- **DDA vs DIA discrimination**: Default parameters (`--max-ms2 11 --threshold 3 --tolerance 0.1`) work well for most datasets. If false positives occur, increase `--threshold` or decrease `--tolerance`.
- **Pulse DIA**: Enabled by default. Pulse DIA check only activates when standard detection fails, acting as a safety net with no impact on normal DIA detection. Disable with `--no-pulse-dia` if needed.

## Terms of Use

This software is provided for **academic research use only**. Redistribution must include all original files including the `converters/` directory. The software is protected by Chinese software copyright (Registration No. 2026SR0602039).

## Citation

```
Tiannan Guo, Zhiwei Liu, Wenjie Zhang. ISDIA software [Z]. 2026SR0602039. 2026.
```

## Contact

- Email: liuzhiwei@westlake.edu.cn
- Issues: GitHub Issues

For commercial cooperation, please contact us.
