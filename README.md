# ISDIA - DIA Mass Spectrometry Detection Tool


## Overview

ISDIA is a professional tool for detecting DIA (Data-Independent Acquisition) mode in mass spectrometry data files. It supports multiple file formats including mzML and Bruker timsTOF (.d) files.

The tool comes in two versions:

- **GUI Version**: PyQt5-based graphical interface 
- **CLI Version**: Command-line interface for batch processing and automation

## Features

- ✅ Support for multiple file formats (mzML, .d, .d.zip)
- ✅ Batch file processing
- ✅ Intelligent DIA pattern detection
- ✅ Pulse DIA mode support
- ✅ Real-time progress tracking
- ✅ CSV result export
- ✅ User-friendly GUI or powerful CLI

## MSConvert parameters (if you need to convert .raw or .wiff to mzML)
The parameters were: --inten64 --mzML --zlib --filter "peakPicking true [1,2]"

## System Requirements

- **Operating System**: Windows 10/11 or Linux
- **Python**: 3.11+ (for building from source)
- **Memory**: 4GB+ RAM recommended
- **Disk Space**: 200MB for installation

## Quick Start

### GUI Version (Recommended)

1. **Download**: Get `ISDIA_gui.exe` (60.01 MB) from the release
2. **Run**: Double-click the executable
3. **Select Files**: Choose input method (direct selection or file list)
4. **Configure Parameters**: Adjust detection settings if needed
5. **Start**: Click "开始检测" (Start Detection)
6. **View Results**: Results are saved to CSV file

**Interface Features**:

- Professional deep blue dark theme
- Unified circular controls (20px) with clear visibility
- Real-time progress tracking with visual feedback
- Comprehensive logging system

### CLI Version

#### Windows

```powershell
.\dist\ISDIA.exe --help
.\dist\ISDIA.exe file_list.txt --output results.csv
```

#### Linux

```bash
./dist/ISDIA --help
./dist/ISDIA file_list.txt --output results.csv
```

## Building from Source

### Prerequisites

Install Python dependencies:

```bash
pip install -r requirements.txt
```

### Build GUI Version (Windows)

```powershell
.\build_windows_gui_pyqt5.ps1
```

Output: `dist\ISDIA_gui.exe` (60.01 MB)

### Build CLI Version

**Windows:**

```powershell
.\build_windows.ps1
```

**Linux:**

```bash
chmod +x build_linux.sh
./build_linux.sh
```

Output: `dist/ISDIA.exe` (28.82 MB on Windows) or `dist/ISDIA` (Linux)

### Build Both Versions

**Important**: You can build both GUI and CLI versions independently without overwriting each other. The build scripts only clean temporary `build/` directories while preserving the final `dist/` output.

```powershell
# Build both versions on Windows
.\build_windows_gui_pyqt5.ps1    # Creates ISDIA_gui.exe
.\build_windows.ps1              # Creates ISDIA.exe

# Both executables will coexist in dist/ folder
```

This allows you to:

- Build either version independently
- Distribute both versions together
- Update one version without affecting the other

## GUI Version Guide

### Interface Overview

```
┌─────────────────────────────────────────┐
│  🔬 DIA Mass Spectrometry Analysis      │
├─────────────────────────────────────────┤
│  1. File Input                          │
│     ○ Direct File Selection             │
│     ○ File List (txt)                   │
│                                         │
│  2. Detection Parameters                │
│     Max MS2 Count: [11]                 │
│     Consecutive Threshold: [3]          │
│     Tolerance: [0.1]                    │
│     ☑ Enable Pulse DIA Mode             │
│                                         │
│  3. Execution                           │
│     [Start Detection] [View Results]    │
│     Progress: ████████░░ 80%            │
│                                         │
│  4. Processing Log                      │
│     [Log output area...]                │
└─────────────────────────────────────────┘
```

### Parameters

| Parameter             | Description                          | Default | Range  |
| --------------------- | ------------------------------------ | ------- | ------ |
| Max MS2 Count         | Maximum MS2 scan count threshold     | 11      | 1-100  |
| Consecutive Threshold | Consecutive scan detection threshold | 3       | 1-10   |
| Tolerance             | m/z tolerance for comparison         | 0.1     | 0-1    |
| Pulse DIA             | Enable Pulse DIA detection mode      | Off     | On/Off |
| Pulse Tolerance       | Tolerance for Pulse DIA mode         | 0.01    | 0-0.1  |

### File Input Methods

#### Method 1: Direct Selection

1. Select "直接选择文件" (Direct File Selection)
2. Click "选择文件..." (Select Files)
3. Choose one or more mzML/.d files
4. Files appear in the list below

#### Method 2: File List

1. Select "使用文件列表 (txt)" (Use File List)
2. Prepare a text file with file paths (one per line):
   ```
   data/sample1.mzML
   data/sample2.d
   data/sample3.d.zip
   ```
3. Browse and select the file list
4. Set base directory if using relative paths

### Output Format

CSV file with columns:

- `filename`: Input file name
- `is_dia`: True/False (DIA detection result)
- `ms2_count`: Number of MS2 scans detected
- `consecutive_windows`: Number of consecutive isolation windows
- `detection_method`: Detection algorithm used

## CLI Version Guide

### Basic Usage

```bash
ISDIA <file_list.txt> [options]
```

### Command-Line Arguments

```
positional arguments:
  file_list             Path to text file containing list of files to process

options:
  -h, --help            Show help message
  -o, --output OUTPUT   Output CSV file path (default: isdia_results.csv)
  -d, --base-dir DIR    Base directory for file paths (default: .)
  --max-ms2 N          Maximum MS2 count threshold (default: 11)
  --threshold N         Consecutive threshold (default: 3)
  --tolerance FLOAT     m/z tolerance (default: 0.1)
  --pulse-dia           Enable Pulse DIA mode
  --pulse-tol FLOAT     Pulse DIA tolerance (default: 0.01)
```

### Examples

**Example 1: Basic usage**

```bash
ISDIA file_list.txt
```

**Example 2: Custom output and parameters**

```bash
ISDIA file_list.txt \
  --output my_results.csv \
  --max-ms2 15 \
  --threshold 5
```

**Example 3: Enable Pulse DIA mode**

```bash
ISDIA file_list.txt \
  --pulse-dia \
  --pulse-tol 0.02
```

**Example 4: Specify base directory**

```bash
ISDIA file_list.txt \
  --base-dir /path/to/data \
  --output results.csv
```

### File List Format

Create a text file (e.g., `file_list.txt`) with one file path per line:

```
sample001.mzML
sample002.d
sample003.d.zip
subfolder/sample004.mzML
```

Supports:

- Relative paths (relative to `--base-dir`)
- Absolute paths
- Mixed mzML and .d formats

## Algorithm Details

### DIA Detection Logic

1. **MS2 Scan Analysis**:

   - Count MS2 scans in each file
   - Check if count exceeds `max-ms2` threshold
2. **Isolation Window Detection**:

   - Extract isolation windows from MS2 scans
   - Sort by m/z value
   - Calculate m/z differences between consecutive windows
3. **Consecutive Window Check**:

   - Compare differences with `tolerance`
   - Count consecutive windows within tolerance
   - Report as DIA if count ≥ `threshold`
4. **Pulse DIA Mode** (Optional):

   - Additional check for Pulse DIA patterns
   - Uses stricter `pulse-tolerance`
   - Detects periodic window patterns

## Troubleshooting

### Issue: GUI window is blank or unresponsive

**Solution**: Update graphics drivers or try running in compatibility mode

### Issue: "Failed to load file" error

**Solution**:

- Verify file format (mzML, .d, .d.zip)
- Check file permissions
- Ensure file is not corrupted

### Issue: Progress bar stuck

**Solution**:

- Check log output for detailed error messages
- Verify sufficient disk space
- Try with a smaller file first

### Issue: Wrong detection results

**Solution**:

- Adjust `max-ms2` parameter (increase for more stringent detection)
- Modify `threshold` value (increase to require more consecutive windows)
- Try enabling Pulse DIA mode if analyzing Pulse DIA data

## Performance

- **Processing Speed**: ~5-10 files/second (depends on file size)
- **Memory Usage**: ~200-500 MB per instance
- **Disk I/O**: Optimized for SSD (HDD may be slower)

## Version History

### v2.1 (Current)

- ✅ Optimized build scripts for dual-version coexistence
- ✅ Separate executable names (GUI: `_gui.exe`, CLI: `.exe`)
- ✅ Unified circular control styling (20px, 3px border)
- ✅ Improved control visibility and consistency
- ✅ Window title updated to "ISDIA"
- ✅ Comprehensive bilingual documentation

### v2.0

- ✅ PyQt5 GUI with professional dark theme
- ✅ Improved UI controls (radio buttons, checkboxes)
- ✅ Real-time progress updates
- ✅ Enhanced error handling

### v1.0

- Initial release with CLI support
- Basic DIA detection
- Tkinter GUI (deprecated)

## License

MIT License - See LICENSE file for details

## Citation

If you use this tool in your research, please cite:

```
Tiannan Guo, Zhiwei Liu, Wenjie Zhang. ISDIA software [Z]. 2026SR0602039. 2026.
```

## Contact

- **Issues**: Report bugs on GitHub Issues
- **Email**: liuzhiwei@westlake.edu.cn

## Acknowledgments

Developed at Westlake Laboratory for mass spectrometry data analysis.

---

**Note**: This tool is for research use only. Always validate results with domain knowledge.
