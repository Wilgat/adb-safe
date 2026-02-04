# adb-safe

**Reliable ADB wrapper that prevents daemon startup failures on Linux/macOS**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Shell](https://img.shields.io/badge/Shell-POSIX-green.svg)](https://www POSIX.org)

`adb-safe` is a simple, lightweight wrapper around the Android Debug Bridge (`adb`) command that solves one of the most frustrating issues on Linux (and sometimes macOS):

```
failed to start daemon
ADB server didn't ACK
cannot connect to daemon
```

It automatically retries starting the ADB daemon, adds small delays, kills and restarts the server when needed, and provides clear feedback — so you spend less time fighting ADB and more time developing.

## Features

- Automatically recovers from ADB daemon startup race conditions
- Retries up to 5 times with intelligent backoff
- Shows helpful recovery messages
- Same syntax as plain `adb` — zero learning curve
- One-liner global installation (like many popular CLI tools)
- Detects missing `adb` and suggests OS-specific installation commands
- Detailed `--help` output with examples and troubleshooting tips
- Works on Linux (Ubuntu, Fedora, Arch, etc.) and macOS

## Installation

### Recommended (one-liner)

```bash
curl -fsSL https://raw.githubusercontent.com/Wilgat/adb-safe/main/adb-safe | sh
```

If you get a permission error:

```bash
sudo curl -fsSL https://raw.githubusercontent.com/Wilgat/adb-safe/main/adb-safe | sudo sh
```

This installs `adb-safe` to `/usr/local/bin/adb-safe`.

### Manual install

1. Download the script:
   ```bash
   curl -fsSL -o adb-safe https://raw.githubusercontent.com/Wilgat/adb-safe/main/adb-safe
   ```

2. Make it executable and move to PATH:
   ```bash
   chmod +x adb-safe
   sudo mv adb-safe /usr/local/bin/
   ```

### Optional: Alias (recommended)

Add this to your `~/.bashrc`, `~/.zshrc` or equivalent:

```bash
alias adb='adb-safe'
```

Then you can keep using `adb` normally while getting the safety benefits.

## Usage

Just replace `adb` with `adb-safe`:

```bash
adb-safe devices -l
adb-safe shell
adb-safe logcat
adb-safe install myapp.apk
adb-safe push localfile.txt /sdcard/
adb-safe reboot
```

Special commands:

```bash
adb-safe help          # detailed help & troubleshooting
adb-safe version       # show version and real adb version
```

## If ADB is not installed

When you first run `adb-safe` and `adb` is missing, it will:

- Detect your OS/distribution
- Show the recommended installation command
- Ask if you want to install automatically (uses `sudo` when needed)

Examples:

- Ubuntu/Debian: `sudo apt install adb`
- Fedora: `sudo dnf install android-tools`
- Arch/Manjaro: `sudo pacman -S android-tools`
- macOS: `brew install --cask android-platform-tools`

## Why this happens (and why adb-safe helps)

On Linux, the ADB server (daemon) sometimes takes 0.5–3 seconds to fully start after `adb start-server` is called — especially when USB permissions are being applied via udev rules. The client gives up too quickly → classic "didn't ACK" error.

`adb-safe` adds patience + automatic recovery:

- Checks if daemon is alive
- Retries with delays
- Kills and restarts on later attempts
- Gives clear feedback instead of cryptic errors

## Troubleshooting

| Problem                          | Possible Fix / Check                                                                 |
|----------------------------------|--------------------------------------------------------------------------------------|
| Still "failed to start daemon"   | Run `adb-safe kill-server` then `adb-safe start-server`                             |
| Device shows "unauthorized"      | Revoke USB debugging authorizations on the device → reconnect                       |
| Device "offline"                 | Unplug/replug USB cable, or run `adb-safe reconnect`                                |
| No devices listed                | Check USB debugging is enabled + accept RSA prompt on device                        |
| Permission denied (Linux)        | Ensure you're in `plugdev` group + have udev rules for your device vendor           |
| Port 5037 conflict               | `ss -tulnp | grep 5037` or `lsof -i :5037` — kill conflicting process if needed      |

For udev rules example (common for Rockchip, Google, Samsung devices):

```bash
sudo nano /etc/udev/rules.d/51-android.rules
# Add: SUBSYSTEM=="usb", ATTR{idVendor}=="XXXX", MODE="0666", GROUP="plugdev"
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## Contributing

Contributions are welcome!

1. Fork the repo
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

MIT License

See [LICENSE](LICENSE) for full text.
