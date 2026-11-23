# ModPolling Tool

[![Windows](https://img.shields.io/badge/Windows-10%2F11-0078D6?logo=windows&logoColor=white)](https://www.microsoft.com/windows) [![Built with Python](https://img.shields.io/badge/Built%20with-Python%203.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/) ![Modbus](https://img.shields.io/badge/Modbus-RTU%20%2F%20TCP-0a84ff) ![Portable](https://img.shields.io/badge/Install-None%20(portable)-success) ![Status](https://img.shields.io/badge/Status-Active-success)

A modern Windows GUI for polling Modbus devices, built around the proven [`modpoll`](https://www.modbusdriver.com/modpoll.html) command-line utility. Pick an equipment preset, hit **Start Polling**, and watch live color-coded responses — no need to memorize command-line flags.

<p align="center">
    <img width="900" src="https://i.ibb.co/yFKZTnGw/2026-01-09-12-58-10-modpoll-py-Modoll-Workspace-Cursor.png" alt="ModPolling Tool main window: equipment presets, connection settings, and live terminal output">
</p>

## Highlights 🚀

- **⚡ One-click polling** over serial (Modbus RTU) or Modbus TCP
- **🔎 Auto-Detect** — scans baudrates and parities automatically to find a device
- **🧩 70+ equipment presets** (Carel, AK-CC, Carlo Gavazzi, Schneider, Kamstrup, Regin, Corrigo, Dixell, Swegon, Flexit, …) that auto-fill baud, parity, data/stop bits
- **🧪 Live command preview** — see the exact `modpoll` command, edit it inline for advanced flags
- **🟢🟡🔴 Status indicator** — instant visual feedback: responding, checksum trouble, or dead air
- **🧠 Smart log parsing** — suppresses boilerplate, counts attempts, and translates errors into plain language
- **🔌 Thorough COM port discovery** via pySerial *and* the Windows registry (catches virtual ports from tools like NPort Administrator)
- **🗃️ Units tab (optional)** — auto-populate connection settings from a local IWMAC plant database
- **🔢 COM10+ handled automatically** using the `\\.\COMn` format Windows requires

## 📥 Download & Install

**[⬇ Download ModPollingTool.exe](https://github.com/hapnes-dev/ModpollingTool/releases/download/modpollv2/ModPollingTool.exe)** (~31 MB) — or browse all [releases](https://github.com/hapnes-dev/ModpollingTool/releases).

1. Download `ModPollingTool.exe` and place it anywhere you like.
2. Double-click to run — it's a **portable app**: no installer, no Python, no dependencies.
3. On first run the app makes sure `modpoll.exe` is available at `C:\iwmac\bin\modpoll.exe`, downloading it automatically if missing. You can also grab it manually: **[Download modpoll.exe](https://github.com/hapnes-dev/ModpollingTool/releases/download/modpollv2/modpoll.exe)**.

> **Note:** Windows SmartScreen may warn on first launch because the executable is unsigned. Choose **More info → Run anyway**.

## ⚡ Quick Start

1. Press **Refresh** to list COM ports (or type one manually).
2. Choose an equipment preset in the left pane — or set baud/parity yourself.
3. **RTU:** pick the COM port and set the slave address. **TCP:** enter `host[:port]` in the *Modbus TCP/IP* field.
4. Glance at the live **Command** preview to confirm what will run.
5. Click **Start Polling**. Green means the device is talking. **Stop Polling** ends the session.

Not sure about the serial settings? Enter the slave address and hit **AUTO DETECT** instead.

## 🧭 Using the App

### Serial (RTU) polling 🔌
- Select a COM port — ports ≥ 10 are automatically formatted as `\\.\COMn`.
- Set Address (`-a`), Baudrate (`-b`), and Parity (`-p`). Data bits (`-d`), stop bits (`-s`), start reference (`-r`), register count (`-c`), and register type (`-t`) live under the **Advanced** tab.
- Click **Start Polling**. The log shows attempts, parsed data lines like `[100]: 70`, and highlights timeouts or port errors.

### Modbus TCP 🌐
- Enter `host[:port]` in the *Modbus TCP/IP* field (port defaults to 502).
- Address and register settings (`-a`, `-r`, `-c`, `-t`) still apply.
- Click **Start Polling**.

### Equipment presets 🧩
- Search (`Ctrl+F`) and select an equipment model; the app fills in its typical baud/parity/data/stop settings.
- Presets are a starting point — you can override any field afterwards.

### Auto-Detect 🔎
Don't know the device's serial settings? Let the tool find them:

1. Enter the **slave address** (`-a`) in the Basic tab.
2. Click **AUTO DETECT**.
3. The tool tries each combination in order of real-world likelihood:
   `9600/none → 19200/none → 9600/even → 19200/even → 9600/odd → 19200/odd`
4. As soon as a device responds, the working settings are **applied to the UI automatically**.
5. Click **STOP** to cancel the scan early.

### Command preview & custom arguments 🧪
- The **Command** field always shows exactly what will be passed to `modpoll`.
- Edit it inline for advanced scenarios (flags not exposed in the UI) — your edited command is used as-is when you start polling.

### Status indicator 🟢🟡🔴
The circular indicator blinks while responses come in:

| Color | Meaning |
|-------|---------|
| 🟢 Green | Valid responses — including Modbus exceptions (illegal function / data address / value), which still prove the device is alive |
| 🟡 Yellow | Checksum errors — usually wiring noise or a baud/parity mismatch |
| 🔴 Red | Timeouts or port/socket errors |

The **STOP** button turns red whenever polling or auto-detect is active.

### Units tab (optional, IWMAC) 🗃️
On machines running an IWMAC plant server, **Get units data** queries the local database and lists every configured unit with its ID, name, driver, address, COM/IP, baudrate, and parity. Select a row (or **Apply Preset**) to fill the connection fields in one click — TCP-mode units fill the IP field instead of a COM port. A toggle filters the table to Modbus-capable units only.

## 🔍 Under the Hood

The app builds a `modpoll` command from your settings and runs it without spawning console windows:

```bash
# Serial RTU example
modpoll COM3 -b9600 -pnone -a1 -r100 -c1 -t3

# Modbus TCP example
modpoll -m tcp 192.168.1.50:502 -b9600 -peven -d8 -s1 -a1 -r100 -c10 -t3
```

Rather than one long-running process, the tool runs `modpoll` in **one-shot mode (`-1`) roughly once per second**. On Windows, a continuously running modpoll buffers its piped output unpredictably — one-shot mode guarantees every poll's result reaches the log immediately, giving a true 1-second cadence.

The log intentionally hides modpoll's headers and boilerplate so you only see data lines, attempt counts, and actionable errors.

### Argument reference

| Flag | Meaning | UI field |
|------|---------|----------|
| *(positional)* | COM port or `\\.\COMxx` | COM Port |
| `-b` | Baud rate | Baudrate |
| `-p` | Parity (`none`/`even`/`odd`) | Parity |
| `-d` | Data bits (7 or 8) | Data Bits *(Advanced)* |
| `-s` | Stop bits (1 or 2) | Stop Bits *(Advanced)* |
| `-a` | Slave address (1–247) | Address |
| `-r` | Start reference (register number) | Start Reference *(Advanced)* |
| `-c` | Number of registers to read | Number of Registers *(Advanced)* |
| `-t` | Register data type (see below) | Register Data Type *(Advanced)* |
| `-m tcp` | Modbus TCP mode | Modbus TCP/IP field |
| `-1` | One-shot poll | added automatically |

**Register data types (`-t`):**

| Value | Table |
|-------|-------|
| `0` | Coil |
| `1` | Discrete input |
| `3` | 16-bit input register *(default)* |
| `4` | 16-bit holding register |

## 🧰 Troubleshooting

| Problem | What to check |
|---------|---------------|
| **modpoll.exe not found** | Make sure the app can write to `C:\iwmac\bin`, or place `modpoll.exe` there manually. If the auto-download fails, check network/proxy settings. |
| **Serial port already open** | Another program owns the COM port — stop any service using it (e.g. an IWMAC plant server) and try again. |
| **Reply time-out** | Wiring, address, baudrate, or parity is off. Many devices need exact settings — try the vendor preset or **AUTO DETECT**. |
| **Checksum error** | Usually electrical noise or a parity/baud mismatch. Re-check cable shielding and grounding. |
| **COM port missing** | Click **Refresh** or type the port manually. Verify it exists in Device Manager; for virtual ports, make sure the vendor tool actually created it. |
| **SmartScreen blocks the app** | The exe is unsigned — choose **More info → Run anyway**. |
