
# Mock C2 Server with PowerShell Beacon

## Overview

This project simulates a basic Command and Control (C2) infrastructure for educational or lab-based cybersecurity demonstrations. It involves a Python-based C2 server and a PowerShell-based beacon script that runs on the target machine, collecting system information and sending it back to the C2 server.

> ⚠️ **Disclaimer:** This project is strictly for educational and research purposes in controlled environments. Do **not** use on unauthorized systems or networks.

---

## Project Structure

```
.
├── beacon.ps1         # PowerShell beacon script
├── c2_server.py       # Python HTTP C2 server
└── README.md          # Documentation
```

---

## Prerequisites

### On the C2 Server Machine (Attacker)

- Python 3.x installed
- Port 80 open and not in use (e.g., no Apache/Nginx running)
- Optionally, run with `sudo` or Administrator if needed to bind to port 80

### On the Target Machine (Victim)

- Windows OS
- PowerShell enabled
- Administrator privileges to schedule tasks

---

## Setup & Execution

### Step 1: Start the C2 Server

1. Place `c2_server.py` on the server or attack machine.
2. Run the C2 server:

```bash
python3 c2_server.py
```

You should see:
```
[+] C2 Server Running on Port 80...
```

> **Tip:** Use a static IP for the attacker machine (e.g., `192.168.1.193`).

---

### Step 2: Deploy the PowerShell Script on Target

1. Open `beacon.ps1` and make sure the IP address in the script matches the C2 server's IP.
2. On the Windows target machine, run the script with elevated privileges:

```powershell
powershell.exe -ExecutionPolicy Bypass -File .eacon.ps1
```

What it does:
- Immediately collects and sends system info to the C2 server.
- Creates a scheduled task that re-runs the beacon on every logon.

---

## Data Collection

You will see logs like the following on the server console:

```
[+] Beacon Data Received:
Hostname: DESKTOP-XYZ
IP: 192.168.1.105
User: alice
OS: Microsoft Windows 10 Pro
...
```

---

## Indicators of Compromise (IoC)

The script sets up a scheduled task named:

```
MicrosoftEdgeUpdaterService
```

Logs may be created in the Temp directory named `log.txt` (depending on the script version).

You may also notice HTTP POST traffic to the C2 server’s IP on port 80.

![WhatsApp Image 2025-05-22 at 16 58 32_5d109823](https://github.com/user-attachments/assets/33b7a245-b152-46c9-9746-a763064779af)

---

## Teardown / Cleanup

### On Victim Machine:
- Open Task Scheduler → Delete the task: `MicrosoftEdgeUpdaterService`
- Delete the beacon script if still present

### On Server:
- Stop the Python script manually (Ctrl+C)

---

