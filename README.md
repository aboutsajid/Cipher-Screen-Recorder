# Cipher Screen Recorder
<img width="1917" height="1030" alt="Screenshot 2026-02-28 035425" src="https://github.com/user-attachments/assets/cb4d32f8-eadb-4848-a269-61e57f221e75" />

Windows desktop screen recorder built with Python, PyQt6, OpenCV, and FFmpeg.

## Overview

Cipher Screen Recorder is a GUI-first recorder focused on practical workflows:
- record full screen, selected region, camera, or screen plus camera overlay
- capture microphone and optional system audio (WASAPI loopback)
- apply audio sync offsets and auto-calibrate from a clap test
- trim and GIF-export recordings after capture
- run fully from tray/global hotkeys without keeping the main window open
- use instant `Quick Record` mode optimized for speed and low-click workflows

## Feature Set

### Capture modes
- `Only Screen`
- `Screen + Camera` (camera overlay composited on recorded screen)
- `Only Camera`

### Video capture controls
- monitor selection (multi-monitor aware)
- region selection via drag overlay
- camera device selection
- camera overlay corner, size, rounded corners, and border
- FPS presets (`24`, `30`, `60`)
- quality presets mapped to CRF (`High`, `Balanced`, `Compact`)

### Audio controls
- no audio / microphone / system + mic / system only
- microphone device picker
- system loopback device picker (WASAPI)
- built-in audio mixer for mic + system mode
- manual sync offset (`audio_sync_offset_ms`)
- one-click starting offset (`-700ms`) helper
- auto sync calibration from recorded file (`Auto Calibrate`)

### Recording lifecycle
- countdown before start
- auto stop after N minutes
- pause/resume while recording
- quick record mode (instant start, no countdown, no preview dependency)
- low-friction stop flow (no confirmation dialog unless error)
- runtime stats in status bar:
  - encoded FPS
  - dropped frames
  - queue depth
  - CPU/RAM (when `psutil` is available)

### Post recording tools
- open last recording
- copy last recording path
- trim using start/end timecode
- export MP4 to GIF (palette-based high quality)
- in-app gallery list from output folder
- single-click screenshot capture

### Tray and hotkeys
- tray menu actions:
  - start recording
  - stop recording (or cancel countdown)
  - pause/resume
  - open last recording
  - show app
  - exit
- tray icon state indicator (idle / recording / paused)
- double-click tray icon toggles recording
- start minimized to tray (configurable)
- local in-window hotkeys:
  - `F8` screenshot
  - `F9` start/stop
  - `F10` pause/resume
- global system-wide hotkeys (work even when app is unfocused):
  - `Ctrl + Alt + R` toggle start/stop
  - `Ctrl + Alt + P` pause/resume
  - `Ctrl + Alt + S` screenshot

### Speed-first workflow features
- quick record uses last-used capture/audio settings
- output auto-saved to default folder
- quick-record filename format: `YYYY-MM-DD_HH-MM-SS.mp4`
- optional post-stop actions:
  - auto-open output folder
  - auto-copy output path to clipboard
- tray mode and hotkeys stay in sync with recording state

### FFmpeg runtime behavior
- FFmpeg resolution order:
  1. `CIPHERSR_FFMPEG` environment variable
  2. bundled binary in PyInstaller build
  3. local `ffmpeg/` folder
  4. system `PATH`
  5. common Windows install locations

## Project Structure

```text
main.py
ui/
  main_window.py
  global_hotkeys.py
  tray_manager.py
  styles.py
  timer_widget.py
  recording_overlay.py
capture/
  screen_capture.py
  camera_capture.py
  region_selector.py
  screenshot.py
audio/
  mic_capture.py
  system_audio.py
controller/
  recording_controller.py
encoder/
  ffmpeg_encoder.py
utils/
  config.py
  settings_store.py
  qt_settings.py
  ffmpeg_runtime.py
  av_sync_calibration.py
tests/
  test_controller.py
  test_encoder.py
  test_mux_robustness.py
  test_av_sync_calibration.py
  test_screen_capture.py
```

## Requirements

- Windows 10/11
- Python 3.11+
- FFmpeg (installed or bundled)

Python dependencies are listed in `requirements.txt`:
- PyQt6
- mss
- numpy
- sounddevice
- opencv-python
- psutil
- keyboard (Windows global hotkeys)
- pyaudiowpatch (Windows marker)

## Setup

1. Install dependencies:

```powershell
pip install -r requirements.txt
```

2. Ensure FFmpeg is available:
- install via WinGet:

```powershell
winget install ffmpeg
```

- or place binaries in local `ffmpeg/` folder:
  - `ffmpeg/ffmpeg.exe`
  - `ffmpeg/ffprobe.exe`

3. Run:

```powershell
python main.py
```

Optional (run without terminal window):

```powershell
pythonw main.py
```

## Configuration and Persistence

- defaults are defined in `utils/config.py`
- base runtime settings are persisted in `settings.json`
- workflow preferences are persisted via `QSettings` (`utils/qt_settings.py`)
- configurable options include:
  - capture mode and region
  - monitor and camera device
  - audio modes and devices
  - output directory and filename template
  - tray mode, start minimized, countdown, auto stop
  - sync offset and overlay settings
  - quick record mode
  - auto-open output folder after recording
  - auto-copy output path after recording

Default output name template:
- `cipher_screen_recorder_{datetime}_{mode}`

Quick record output template:
- `{iso_datetime}.mp4` (for example `2026-02-28_02-15-09.mp4`)

## Build

### Build EXE (PyInstaller)

```powershell
python -m PyInstaller CipherScreenRecorder.spec --noconfirm
```

Output:
- `dist/Cipher Screen Recorder.exe`

### Build installer (Inno Setup)

```powershell
ISCC.exe installer\CipherScreenRecorder.iss
```

Output:
- `dist/installer/Cipher-Screen-Recorder-Setup.exe`

## Tests

Run automated tests:

```powershell
python -m pytest -q
```

## Troubleshooting

### FFmpeg not found
- verify `ffmpeg -version`
- set `CIPHERSR_FFMPEG` to explicit `ffmpeg.exe` path if needed
- if packaged, confirm bundled FFmpeg files exist

### System audio does not start
- system audio capture requires `pyaudiowpatch` and Windows WASAPI loopback
- select `System` or `System + Mic` and verify loopback device list

### Trim or GIF export fails
- ensure FFmpeg is accessible
- confirm source file is not open in another app
- check free disk space and write permission to output folder

### Audio out of sync
- run a short clap test
- use `Auto Calibrate`
- fine tune `audio_sync_offset_ms` by 50-150ms steps

### Global hotkeys not firing
- verify `keyboard` is installed (`pip install -r requirements.txt`)
- on some Windows setups, global hooks may require running the app with elevated permission
- avoid other apps binding the same shortcuts (`Ctrl+Alt+R/P/S`)
