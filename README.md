# Mock C2 Server with PowerShell Beacon

## Overview
This project demonstrates a basic simulation of a Command and Control (C2) infrastructure. A PowerShell script runs on a victim machine, gathers system information, and sends it to a mock C2 server written in Python. This can be useful for educational and testing purposes in cybersecurity labs.

> ⚠️ **Disclaimer:** This project is for educational and research purposes only. Do not deploy or use on unauthorized systems or networks.

## Components

### 1. `beacon.ps1`

This PowerShell script does the following:
- Collects the machine's hostname, IP address, username, and OS.
- Sends a one-time beacon to the C2 server.
- Sets up a scheduled task that executes the beacon script on every user logon.
- Uses the Windows Task Scheduler COM API to persist via a hidden task.

#### Features:
- Uses base64-encoded PowerShell for stealth.
- Silent failure handling.
- Logs beacon executions to a temp log file.

### 2. `c2_server.py`

A minimal Python HTTP server that listens for incoming POST requests from the beacon. It logs received data to the console.

#### Features:
- Lightweight, no external dependencies.
- Listens on port 80 for incoming beacon data.

## Usage

### Step 1: Start the C2 Server

```bash
python3 c2_server.py
```

### Step 2: Deploy the PowerShell Script

Run `beacon.ps1` on the target system to initiate the beacon and schedule future beacons on user logon.

## IoC
![WhatsApp Image 2025-05-22 at 16 58 32_f7d97f4b](https://github.com/user-attachments/assets/ce1d7627-1741-4721-a997-6d708608264e)

## Notes
- You may need administrator privileges to register scheduled tasks and bind to port 80.
