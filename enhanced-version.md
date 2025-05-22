# Enhanced Mock C2 Server with PowerShell Beacon

## Overview

This project simulates an enhanced Command and Control (C2) infrastructure for cybersecurity testing and research. It includes improvements over the basic version by collecting more detailed system information and improving data parsing on the C2 server.

> ⚠️ **Disclaimer:** This project is for educational and research purposes only. Do not deploy or use on unauthorized systems or networks.

## Enhancements over Basic Version

### PowerShell Beacon (`enhanced_beacon.ps1`)
- **Expanded System Info**: Collects additional data including:
  - System uptime
  - Installed antivirus products
  - Firewall status
  - Top CPU-consuming processes
  - Installed software
- **More Detailed Logon Beacon**: Scheduled task includes all the above details, enhancing persistence and visibility into the target system.
- **Improved Logging**: Timestamps and more verbose data structure sent to C2.

### Python C2 Server (`enhanced_c2_server.py`)
- **POST Data Decoding**: Uses `urllib.parse` to parse and decode incoming POST form data.
- **Formatted Output**: Displays received fields cleanly and in a human-readable format.
- **Modular Code**: Improved structure with function encapsulation and safe defaults.

## Components

### 1. `enhanced_beacon.ps1`

A PowerShell script that gathers a rich set of system data and sets up a scheduled task for logon persistence.

### 2. `enhanced_c2_server.py`

Python HTTP server that decodes, parses, and prints structured POST data sent from the beacon.

## Usage

### Step 1: Start the C2 Server

```bash
python enhanced_c2_server.py
```

### Step 2: Deploy the PowerShell Script

Run `enhanced_beacon.ps1` on the target system. It will:
- Send an initial beacon immediately.
- Create a scheduled task that runs on every user logon.

![WhatsApp Image 2025-05-22 at 16 57 27_fef949de](https://github.com/user-attachments/assets/44c198af-eb70-460c-9fa5-cb8bc2df7740)

## Notes
- Ensure port 80 is open and accessible.
- Administrator privileges are required to register tasks and gather full system data.
- Test in a safe, isolated lab environment.
