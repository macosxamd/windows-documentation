

---

# Remote Desktop Connections  
**Complete Technical Documentation (Windows 2000 → Windows 11)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

Remote Desktop Protocol (RDP):

- Allows remote graphical access to Windows systems  
- Uses **TCP 3389** (and UDP 3389 on newer versions)  
- Supports encryption, NLA, certificates, and redirection  
- Works across LAN, WAN, VPN, and domain environments  

RDP is used for:

- Server administration  
- Remote support  
- VDI environments  
- Multi‑site management  
- Secure remote access  

---

# 2. Requirements

## 2.1 OS Requirements

| OS | Supports RDP Server | Supports RDP Client |
|----|---------------------|----------------------|
| Windows 2000 Pro | ✘ | ✔ |
| Windows 2000 Server | ✔ | ✔ |
| Windows XP Pro | ✔ | ✔ |
| Windows XP Home | ✘ | ✔ |
| Windows Vista → 11 | ✔ | ✔ |
| Windows Server 2003 → 2022 | ✔ | ✔ |

---

## 2.2 Network Requirements

- TCP 3389 open  
- UDP 3389 (optional, improves performance)  
- DNS resolution  
- Time synchronization (Kerberos)  
- Firewall rules enabled  

### CMD

```cmd
ping server01
telnet server01 3389
```

### PowerShell

```powershell
Test-NetConnection -ComputerName server01 -Port 3389
```

---

# 3. Enabling Remote Desktop

## 3.1 GUI Method (All Versions)

1. Open **System Properties**  
2. Click **Remote** tab  
3. Enable:  
   - **Allow users to connect remotely** (XP/2003)  
   - **Allow Remote Desktop** (Vista → 11)  
4. Add users to **Remote Desktop Users** group  

---

# 4. Enabling RDP (CMD)

### Windows XP → Windows 11

Enable RDP:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

Add user to RDP group:

```cmd
net localgroup "Remote Desktop Users" username /add
```

Enable firewall rule:

```cmd
netsh advfirewall firewall set rule group="remote desktop" new enable=Yes
```

---

# 5. Enabling RDP (PowerShell)

### Windows 7 → Windows 11

Enable RDP:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0
```

Add user:

```powershell
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "username"
```

Enable firewall:

```powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

---

# 6. Network Level Authentication (NLA)

NLA requires:

- Domain credentials  
- Secure authentication  
- Supported OS (Vista → 11)  

### CMD (Enable NLA)

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 1 /f
```

### PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1
```

---

# 7. RDP Firewall Rules

## 7.1 CMD

```cmd
netsh advfirewall firewall add rule name="RDP" dir=in action=allow protocol=TCP localport=3389
```

## 7.2 PowerShell

```powershell
New-NetFirewallRule -DisplayName "RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow
```

---

# 8. Change RDP Port

Default port: **3389**

### CMD

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 3390 /f
```

### PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name PortNumber -Value 3390
```

Restart service:

```powershell
Restart-Service TermService -Force
```

---

# 9. RDP Certificates

RDP uses:

- Self‑signed certificates (default)  
- Domain CA certificates (recommended)  

### PowerShell (View certificate)

```powershell
Get-ChildItem -Path Cert:\LocalMachine\Remote Desktop\
```

---

# 10. Remote Desktop Users Group

Users must be in:

```
Remote Desktop Users
```

### CMD

```cmd
net localgroup "Remote Desktop Users"
```

### PowerShell

```powershell
Get-LocalGroupMember -Group "Remote Desktop Users"
```

---

# 11. Remote Desktop Session Host (Server)

Servers support:

- Multiple RDP sessions  
- Licensing (RDS CALs)  
- Session limits  
- RemoteApp  

### CMD (Enable multiple sessions)

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fSingleSessionPerUser /t REG_DWORD /d 0 /f
```

### PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" -Name fSingleSessionPerUser -Value 0
```

---

# 12. Remote Desktop Gateway (Advanced)

Allows secure RDP over HTTPS (TCP 443).

### PowerShell (Install role)

```powershell
Install-WindowsFeature RD-Gateway -IncludeManagementTools
```

---

# 13. Security Hardening

- Enable NLA  
- Use domain accounts  
- Disable local admin RDP  
- Use strong passwords  
- Restrict RDP by firewall rules  
- Use RDP Gateway  
- Use certificates  
- Disable SMBv1  
- Audit logon events  

### CMD (Restrict RDP to specific IP)

```cmd
netsh advfirewall firewall add rule name="RDP Restricted" dir=in action=allow protocol=TCP localport=3389 remoteip=10.0.0.0/24
```

### PowerShell

```powershell
New-NetFirewallRule -DisplayName "RDP Restricted" -Direction Inbound -Protocol TCP -LocalPort 3389 -RemoteAddress 10.0.0.0/24 -Action Allow
```

---

# 14. Troubleshooting RDP

| Issue | Cause | Fix |
|-------|--------|-----|
| Cannot connect | Firewall blocked | Enable RDP rule |
| Black screen | GPU driver | Update driver |
| NLA error | Wrong credentials | Disable NLA temporarily |
| CredSSP error | Patch mismatch | Update OS |
| RDP disconnects | Network issue | Check latency |
| Wrong certificate | CA issue | Reissue cert |

---

## 14.1 CMD Diagnostics

```cmd
qwinsta
query user
netstat -ano | findstr 3389
```

## 14.2 PowerShell Diagnostics

```powershell
Get-NetFirewallRule -DisplayGroup "Remote Desktop"
Test-NetConnection -Port 3389
```

---

# 15. Complete RDP Reference Table

| Feature | Supported OS | CMD | PowerShell |
|---------|--------------|-----|------------|
| Enable RDP | XP → 11 | ✔ | ✔ |
| NLA | Vista → 11 | ✔ | ✔ |
| Change port | XP → 11 | ✔ | ✔ |
| Firewall rules | XP → 11 | ✔ | ✔ |
| RDP Gateway | 2008 → 2022 | ✘ | ✔ |
| Multi‑session | Server only | ✔ | ✔ |

---

