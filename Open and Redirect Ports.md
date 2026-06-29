

---

# Windows Server Port Opening & Port Redirection  
Complete Technical Documentation (Windows 2000 → Windows Server 2022)

---

## 1. Overview

Windows Server handles port access and redirection using:

- **Windows Firewall / ICF / WFAS** (depending on OS version)  
- **RRAS (Routing and Remote Access Service)**  
- **NAT (Network Address Translation)**  
- **Port Forwarding / Port Redirection Rules**

This document covers:

- Opening ports (all OS versions)  
- Redirecting ports (RRAS/NAT)  
- CMD + PowerShell commands for every operation  

---

# 2. Opening Ports (Firewall)

Windows Server versions differ:

| OS Version | Firewall System |
|-----------|------------------|
| Windows 2000 | *No built‑in firewall* (requires external firewall) |
| Windows XP / Server 2003 | ICF / Windows Firewall (netsh firewall) |
| Server 2008 → 2022 | Windows Firewall with Advanced Security (netsh advfirewall / PowerShell) |

---

# 3. Opening Ports (GUI)

### 3.1 Windows XP / Server 2003 (ICF)

1. Control Panel → Windows Firewall  
2. **Exceptions** tab  
3. **Add Port**  
4. Enter name, port, TCP/UDP  
5. Save

### 3.2 Server 2008 → 2022 (WFAS)

1. Open **Windows Firewall with Advanced Security**  
2. **Inbound Rules** → **New Rule**  
3. Select **Port**  
4. Choose **TCP** or **UDP**  
5. Enter port numbers  
6. **Allow the connection**  
7. Apply to profiles  
8. Name the rule

---

# 4. Opening Ports (CMD)

### 4.1 Windows XP / Server 2003

```cmd
netsh firewall add portopening protocol=TCP port=80 name="HTTP"
```

### 4.2 Server 2008 → 2022

```cmd
netsh advfirewall firewall add rule name="HTTP-Inbound" dir=in action=allow protocol=TCP localport=80
```

---

# 5. Opening Ports (PowerShell)

> PowerShell firewall cmdlets exist only on Server 2008 R2 → 2022.

```powershell
New-NetFirewallRule -DisplayName "HTTP-Inbound" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
```

---

# 6. Outbound Port Rules

### CMD (Server 2008 → 2022)

```cmd
netsh advfirewall firewall add rule name="Block-SMTP-Out" dir=out action=block protocol=TCP localport=25
```

### PowerShell

```powershell
New-NetFirewallRule -DisplayName "Block-SMTP-Out" -Direction Outbound -Protocol TCP -LocalPort 25 -Action Block
```

---

# 7. Port Redirection / Port Forwarding (RRAS)

Port redirection requires:

- **RRAS installed**
- **NAT configured**
- **At least one public and one private interface**

RRAS is available on:

- Windows 2000 Server  
- Windows Server 2003  
- Windows Server 2008 → 2022  

---

## 7.1 Enable RRAS for NAT (GUI)

1. Server Manager → Add **Remote Access** role  
2. Enable **Routing**  
3. Open **Routing and Remote Access**  
4. Right‑click server → **Configure and Enable Routing and Remote Access**  
5. Choose **NAT**  
6. Select **public interface**  
7. Enable NAT

---

# 8. Creating Port Forwarding Rules (RRAS)

### Example  
Forward external port **8080** → internal server **192.168.1.50:80**

---

## 8.1 GUI Method (All RRAS Versions)

1. Open **Routing and Remote Access**  
2. Expand:  
   ```
   IPv4
     └── NAT
   ```
3. Right‑click **public interface** → **Properties**  
4. Open **Services and Ports** tab  
5. Click **Add…**  
6. Enter:  
   - Description: `Web Redirect`  
   - Incoming Port: `8080`  
   - Private Address: `192.168.1.50`  
   - Private Port: `80`  
7. Save

---

## 8.2 CMD Method (Windows Server 2003 → 2022)

```cmd
netsh routing ip nat add portmapping name="PublicInterface" protocol=tcp externalport=8080 internalport=80 internaladdress=192.168.1.50
```

> Replace `"PublicInterface"` with the actual interface name from:  
> `netsh routing ip nat show interface`

---

## 8.3 PowerShell Method (Server 2012 → 2022)

PowerShell does **not** include RRAS NAT cmdlets.  
Use **netsh** inside PowerShell:

```powershell
netsh routing ip nat add portmapping name="PublicInterface" protocol=tcp externalport=8080 internalport=80 internaladdress=192.168.1.50
```

---

# 9. Common Port Forwarding Examples

### HTTP

```
External: 80 → Internal: 192.168.1.10:80
```

### HTTPS

```
External: 443 → Internal: 192.168.1.10:443
```

### RDP

```
External: 3389 → Internal: 192.168.1.20:3389
```

### SSH

```
External: 22 → Internal: 192.168.1.30:22
```

---

# 10. Testing Ports

### CMD

```cmd
telnet server-ip 8080
```

### PowerShell

```powershell
Test-NetConnection -ComputerName server-ip -Port 8080
```

### Local port check

```cmd
netstat -ano | findstr :8080
```

```powershell
Get-NetTCPConnection -LocalPort 8080
```

---

# 11. Troubleshooting

| Issue | Cause | Solution |
|-------|--------|----------|
| Port open but unreachable | Firewall blocking | Check inbound rules |
| Port forwarded but not working | RRAS misconfigured | Verify NAT interface |
| Wrong interface | Public interface not selected | Reconfigure NAT |
| Service not listening | App not bound to port | Check `netstat -ano` |
| Duplicate rules | Conflicting firewall entries | Remove duplicates |

---
