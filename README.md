ChuniVController Plus
---

A modern touch-based virtual controller for Chunithm with support for LAN play and webcam-based air note detection.

## Features

- **16-key Touchpad**: Maps keys A–P on your screen for slider input
- **Air Sensor Support**: Click/touch the sensor area above the keyboard, or use a webcam for hand detection
- **LAN Support**: Play on a different machine than your server over local network
- **Webcam IR Detection**: Automatic hand/motion detection via webcam for air notes (experimental)
- **Customizable Layout**: Adjust key width, height, and sensor size on the fly
- **Window Controls**: Opacity adjustment, window lock, and more

## Quick Start

1. **Get the DLL**: Replace your game's `chuniio.dll` with the one from this project
2. **Run the EXE**: Execute `ChuniVController Plus.exe` on the client machine
3. **Start the game**: Run your Chunithm game server (with the custom DLL loaded)

The client will connect to `127.0.0.1:24864` by default. See [LAN Setup](#lan-setup) to connect over a network.

## Usage

### Basic Controls

| Action | Method |
|--------|--------|
| **Slider keys** | Click/touch the 16 rectangles (A–P) |
| **Air sensors** | Click/touch the sensor area at the top (or use webcam) |
| **Coin insert** | Click the "Coin" button |
| **Test/Service** | Use the buttons in the control panel |

### Control Panel Options

| Option | Description |
|--------|-------------|
| Key width | Adjust slider key width |
| Key height | Adjust slider key height |
| Sensor height | Adjust IR sensor area height |
| Apply | Apply layout changes |
| Opacity | Adjust window transparency (Alt+O) |
| Allow Mouse | Enable mouse interaction (disabled by default; uses touch) |
| Lock Window | Prevent window dragging (Alt+L) |
| Coin | Insert a coin |
| Service | Press service button |
| Test | Press test button |
| S+T | Press service + test simultaneously |
| Exit | Close the application |

## LAN Setup

To connect to the game server on a different machine:

1. Ensure the server machine is running the modified `chuniio.dll`
2. On the client machine, edit [ChuniVController/ChuniVController/App.config](ChuniVController/ChuniVController/App.config):

```xml
<appSettings>
  <add key="ServerIp" value="192.168.1.100" />
  <add key="ServerPort" value="24864" />
</appSettings>
```

3. Replace `192.168.1.100` with your server's LAN IP
4. Run `ChuniVController Plus.exe`
5. On the server, allow inbound UDP on port 24864 in Windows Firewall

## Webcam Air Notes (Experimental)

Use a webcam to automatically detect hand motions for air notes instead of clicking/touching.

### Setup

1. Enable in [ChuniVController/ChuniVController/App.config](ChuniVController/ChuniVController/App.config):

```xml
<add key="WebcamEnabled" value="true" />
<add key="WebcamDeviceIndex" value="0" />
<add key="WebcamMotionThreshold" value="20" />
<add key="WebcamBottomDeadzonePercent" value="15" />
```

2. Position your webcam above the play area, pointing straight down
3. Run the controller — a popup confirms webcam detection started
4. Wave your hands over the 6 zones to trigger air notes

### Configuration

| Setting | Range | Default | Notes |
|---------|-------|---------|-------|
| `WebcamEnabled` | true/false | false | Enable/disable webcam detection |
| `WebcamDeviceIndex` | 0–N | 0 | Camera index (0 = first webcam) |
| `WebcamMotionThreshold` | 10–30 | 20 | Sensitivity to pixel changes (lower = more sensitive) |
| `WebcamBottomDeadzonePercent` | 0–50 | 15 | Ignore bottom % of frame to reduce false triggers |

**Tips:**
- Lower `WebcamMotionThreshold` (10–15) for faster response
- Increase `WebcamBottomDeadzonePercent` (20–30) if your body triggers false detections
- Good lighting and a plain background improve accuracy
- The frame is divided into 6 vertical zones; motion triggers the corresponding IR sensor

### Technical Details

- Uses AForge.NET for DirectShow webcam capture (.NET Framework 4.7.2 compatible)
- Motion detection via frame differencing (no machine learning)
- Runs at ~30 FPS for responsive detection
- Sends `IR_BLOCKED`/`IR_UNBLOCKED` UDP messages to the game server

## Building

### Minimum Requirements

- Visual Studio Build Tools 2022 (or Visual Studio Community)
  - ".NET Desktop Development" workload
  - .NET Framework 4.7.2 Developer Pack
- NuGet CLI (for package restoration)

### Build Command

```powershell
# Find MSBuild and compile
$vs = & "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.Component.MSBuild -property installationPath
& "$vs\MSBuild\Current\Bin\MSBuild.exe" "ChuniVController\ChuniVController\ChuniVController.csproj" /p:Configuration=Release
```

**Output:** [ChuniVController/ChuniVController/bin/Release/ChuniVController Plus.exe](ChuniVController/ChuniVController/bin/Release/)

### Build Without Visual Studio

If you want to build only the server DLL (no UI), use MSYS2 + MinGW:

1. Install MSYS2 (https://www.msys2.org/)
2. Open "MSYS2 MinGW x64" and run:

```bash
pacman -S --needed mingw-w64-x86_64-gcc
cd /d/Webbivelhoilut/chunithm-vcontroller-lan/ChuniVController/ChuniIO
make -f Makefile.mingw
```

Output: `CHUNIIO.dll` in the same folder

## How It Works

- **Server DLL (`chuniio.dll`)**: Listens on UDP port 24864, receives control messages from the client, and relays them to the game
- **Client EXE**: Captures your input (touches, clicks, or webcam motion) and sends UDP messages to the server
- **Protocol**: Simple binary format with message type, target (key/sensor), and data (LED color for feedback)

See [ChuniVController/ChuniIO/chuniio.h](ChuniVController/ChuniIO/chuniio.h) for protocol details.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails with "MSBuild not found" | Install Visual Studio Build Tools 2022 |
| Webcam not detected | Check `WebcamDeviceIndex` (start at 0); try increasing by 1 if you have multiple cameras |
| Webcam triggers too easily | Increase `WebcamMotionThreshold` or `WebcamBottomDeadzonePercent` |
| LAN connection fails | Verify server IP, ensure firewall allows UDP 24864, check network connectivity |
| DLL not loading | Confirm you replaced the original `chuniio.dll` in your game directory |

## License

This project is licensed under the GNU General Public License v2.0 (GPLv2).

See the [LICENSE](LICENSE) file for the full license text.

## Forks & Attribution

If you fork or modify this project, please kindly mention this work and the original project in your fork's README or release notes. A short attribution like the following is appreciated:

"Based on ChuniVController Plus — original project: Nat-Lab/chunithm-vcontroller (plus edition enhancements)."

Thank you it helps others find the original source and preserves credit for improvements.

## Credits

Original project: [Nat-Lab/chunithm-vcontroller](https://github.com/Nat-Lab/chunithm-vcontroller)

Plus Edition: Added LAN support, webcam air note detection, and improved configurability
