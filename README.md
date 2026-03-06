# NanoKVM USB — Native macOS Client

A lightweight, native macOS client for the [Sipeed NanoKVM USB](https://wiki.sipeed.com/nanokvm) HDMI-to-USB capture device. The entire application is a single Swift file with zero external dependencies — just compile and run.

![macOS 12+](https://img.shields.io/badge/macOS-12.0%2B-blue)
![Swift](https://img.shields.io/badge/Swift-single--file-orange)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

### Video
- **USB capture device support** — auto-detects external HDMI capture devices, filters out built-in cameras
- **Multiple formats** — MJPEG, H.264, H.265, and raw NV12 with automatic quality scoring
- **Resolution selection** — browse and switch between all supported resolutions and frame rates
- **GPU-accelerated preview** — renders via `AVCaptureVideoPreviewLayer` with correct aspect ratio and letterboxing
- **Fullscreen** — toggle with `Cmd+F`; menu bar and toolbar auto-hide
- **Settings persistence** — last-used resolution and format restored on launch

### Keyboard
- **Full HID passthrough** — translates macOS virtual keycodes to USB HID scan codes over serial
- **All standard keys** — letters, numbers, punctuation, function keys (F1–F12), navigation, keypad
- **Modifier support** — Ctrl, Shift, Option, Command
- **Preset combos** — Ctrl+Alt+Del, Win+Tab, Alt+F4, Ctrl+Esc available from the toolbar menu
- **Paste as typing** — pastes clipboard content character-by-character with correct shift handling

### Mouse
- **Absolute mode** (default) — pointer position mapped to remote screen coordinates (0–4095)
- **Relative mode** — sends delta movement, useful for certain remote applications
- **Full button support** — left, right, middle, and extra buttons
- **Scroll wheel** — natural or inverted direction, four speed presets (Slow / Normal / Fast / Very Fast)
- **Mouse jiggler** — prevents screensaver on the remote machine (30-second micro-movements)
- **Cursor hide** — optionally hides the local cursor while the window is focused

### Audio
- **CoreAudio pass-through** — routes audio from the USB capture device directly to your speakers via HAL audio units
- **Lock-free ring buffer** — 48 kHz stereo Float32 with power-of-2 masking for glitch-free real-time playback
- **Mute toggle** — silence playback without stopping capture
- **Audio-triggered refresh** — when the window is unfocused, a sound on the remote machine automatically wakes the display (2-second throttle)

### Recording & Screenshots
- **Video recording** — record to `.mov` with H.264 or H.265 (HEVC) codec
- **Screenshots** — save as PNG, JPEG, or HEIC with configurable quality (50–100%)
- **Toolbar indicator** — record button turns red while recording
- **All settings persisted** — codec and screenshot format/quality remembered across sessions

### Background Refresh
When the window loses focus, the capture session pauses to save CPU. A configurable timer periodically wakes it to grab a single frame:

| Interval | Behaviour |
|----------|-----------|
| **Live** | Session never pauses (continuous) |
| **1 s – 5 min** | Periodic single-frame refresh |

- **Frozen-frame overlay** — last captured frame displayed as a static layer while paused
- **Audio-triggered wake** — sound activity on the remote triggers an immediate refresh regardless of timer
- **Session watchdog** — if the capture session fails to produce a frame within 3 seconds of refocusing, it is automatically force-restarted

### Serial Communication
Communicates with the NanoKVM's CH552 HID chip over USB serial:

- Auto-discovers `/dev/cu.usbmodem*` and `/dev/cu.usbserial*` ports
- Checksummed packet protocol (header `0x57 0xAB`)
- Firmware version query
- Graceful disconnect/reconnect handling

## Requirements

- **macOS 12.0** or later
- **Sipeed NanoKVM USB** (or compatible USB HDMI capture device + CH552 serial)
- **Xcode Command Line Tools** (for `swiftc`)
  ```
  xcode-select --install
  ```

## Build & Run

```bash
git clone https://github.com/mrowlinson/NanoKVM-USB-Swift.git
cd NanoKVM-USB-Swift
bash build.command
open NanoKVM.app
```

That's it. `build.command` compiles `NanoKVM.swift` into a self-contained app bundle with a single `swiftc` invocation — no Xcode project, no package manager, no dependencies.

### What `build.command` does

```
swiftc NanoKVM.swift -o NanoKVM_bin \
  -framework AppKit \
  -framework AVFoundation \
  -framework CoreMedia \
  -framework AudioToolbox \
  -framework VideoToolbox \
  -framework UniformTypeIdentifiers \
  -framework Metal \
  -O
```

Then bundles the binary with `Info.plist` and `AppIcon.icns` into `NanoKVM.app`.

## Permissions

On first launch macOS will prompt for:

| Permission | Why |
|---|---|
| **Camera** | Required to access the USB HDMI capture device video feed |
| **Microphone** | Required to capture audio from the USB HDMI capture device |

These are standard `AVCaptureDevice` permissions. The app does not access your Mac's built-in camera or microphone — it specifically filters for external USB capture devices.

## Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Fullscreen | `Cmd+F` |
| Quit | `Cmd+Q` |

Additional key combos are available from the **Keyboard** toolbar menu:

- **Ctrl+Alt+Del** — send to remote
- **Win+Tab** — Windows task switcher
- **Alt+F4** — close active window on remote
- **Ctrl+Esc** — open Start menu
- **Paste** — type clipboard contents
- **Release All** — force-release all held keys

## Architecture

```
NanoKVM.swift          ← entire application (single file)
build.command          ← build script
Info.plist             ← app bundle metadata
AppIcon.icns           ← app icon
```

The app uses only Apple's built-in frameworks:

| Framework | Purpose |
|-----------|---------|
| AppKit | Window, menus, toolbar, event handling |
| AVFoundation | Video/audio capture session, recording |
| CoreMedia | Sample buffer handling |
| AudioToolbox | CoreAudio HAL units for audio pass-through |
| VideoToolbox | Pixel buffer → CGImage conversion |
| UniformTypeIdentifiers | File type handling for save dialogs |
| Metal | GPU layer backing for the preview |

### Why single-file?

The NanoKVM USB client is fundamentally simple — it reads video from a capture device, sends keyboard/mouse over serial, and passes audio through. Splitting this across files, adding a package manager, or using an Xcode project would add complexity without adding value. A single `swiftc` invocation produces a working app in under 5 seconds.

## Related

- [Sipeed NanoKVM USB](https://github.com/sipeed/NanoKVM-USB) — official Electron/browser client
- [Sipeed NanoKVM](https://wiki.sipeed.com/nanokvm) — product documentation

## License

MIT
