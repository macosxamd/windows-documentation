

---

# Limiting Logon in Windows Domains  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

# 1. Overview

“Limiting logon” refers to restricting **where**, **when**, and **how** users can authenticate in a Windows environment.

Windows supports multiple logon restriction mechanisms:

- Logon hours  
- Logon workstation restrictions  
- Logon to specific computers  
- Smart card enforcement  
- Interactive logon policies  
- Network logon restrictions  
- RDP logon restrictions  
- Local logon vs remote logon rights  
- Account lockout policies  
- Credential restrictions  
- Conditional access (newer versions)

This document covers **all** of them.

---

# 2. Logon Hours Restrictions

Restrict when a user can log on.

## 2.1 GUI (All Domain Versions)

1. Open **Active Directory Users and Computers**  
2. Right‑click user → **Properties**  
3. **Account** tab  
4. **Logon Hours**  
5. Configure allowed times

## 2.2 CMD (Windows 2000 → Server 2022)

```cmd
net user username /times:M-F,08:00-17:00
```

## 2.3 PowerShell (Server 2008 → 2022)

PowerShell cannot directly set logon hours, but you can modify ADSI attributes:

```powershell
$user = [ADSI]"LDAP://CN=User,CN=Users,DC=domain,DC=local"
$user.logonHours = $hoursByteArray
$user.SetInfo()
```

---

# 3. Logon Workstation Restrictions

Restrict which computers a user can log on to.

## 3.1 GUI

1. ADUC → User → **Account** tab  
2. Click **Log On To…**  
3. Add allowed workstation names

## 3.2 CMD

```cmd
net user username /workstations:PC1,PC2,PC3
```

## 3.3 PowerShell

```powershell
Set-ADUser username -LogonWorkstations "PC1","PC2","PC3"
```

---

# 4. Restrict Local Logon (User Rights Assignment)

Controls who can log on **locally** at a machine.

## 4.1 Policy Path

```
Computer Configuration
 └── Windows Settings
     └── Security Settings
         └── Local Policies
             └── User Rights Assignment
```

## 4.2 Relevant Rights

| Right | Description |
|-------|-------------|
| Allow log on locally | Who can log on at console |
| Deny log on locally | Explicit deny |
| Allow log on through Remote Desktop Services | RDP access |
| Deny log on through Remote Desktop Services | Explicit deny |

## 4.3 CMD (Local Security Policy)

```cmd
secedit /export /cfg sec.cfg
```

## 4.4 PowerShell (Server 2012 → 2022)

```powershell
Import-Module SecurityPolicyDsc
```

*(PowerShell cannot directly modify these rights without DSC or external modules.)*

---

# 5. Restrict Remote Logon (RDP)

## 5.1 GUI

Computer Properties → Remote Settings → Select Users

## 5.2 CMD

```cmd
net localgroup "Remote Desktop Users" username /add
```

## 5.3 PowerShell

```powershell
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "username"
```

---

# 6. Smart Card Logon Enforcement

Requires users to authenticate using smart cards.

## 6.1 GUI

User → **Account** tab → Check **Smart card is required for interactive logon**

## 6.2 CMD

```cmd
dsmod user "CN=User,CN=Users,DC=domain,DC=local" -smartcard required
```

## 6.3 PowerShell

```powershell
Set-ADUser username -SmartcardLogonRequired $true
```

---

# 7. Account Lockout Policies

Restrict logon attempts.

## 7.1 Policy Path

```
Computer Configuration
 └── Windows Settings
     └── Security Settings
         └── Account Policies
             └── Account Lockout Policy
```

## 7.2 CMD

```cmd
net accounts /lockoutthreshold:5 /lockoutduration:30 /lockoutwindow:30
```

## 7.3 PowerShell

```powershell
net accounts /lockoutthreshold:5
```

*(PowerShell does not have native cmdlets for lockout policy.)*

---

# 8. Kerberos Logon Restrictions

## 8.1 Enforce Pre‑Authentication

```cmd
dsmod user "CN=User,CN=Users,DC=domain,DC=local" -disabled no
```

## 8.2 PowerShell

```powershell
Set-ADUser username -KerberosEncryptionType AES128,AES256
```

---

# 9. Credential Restrictions

## 9.1 Deny Cached Credentials

Policy:

```
Computer Configuration → Windows Settings → Security Settings → Local Policies → Security Options
```

Setting:

- **Interactive logon: Number of previous logons to cache**

## 9.2 CMD

```cmd
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v CachedLogonsCount /t REG_SZ /d 0 /f
```

## 9.3 PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name CachedLogonsCount -Value 0
```

---

# 10. Limit Logon to Specific Domain Controllers

Use **site/subnet configuration**.

## 10.1 CMD

```cmd
nltest /dsgetsite
```

## 10.2 PowerShell

```powershell
Get-ADReplicationSite -Filter *
```

---

# 11. Limit Logon by Network Location

Use **Windows Firewall** or **GPO** to restrict authentication traffic.

## 11.1 CMD

```cmd
netsh advfirewall firewall add rule name="Block Logon" dir=in action=block protocol=TCP localport=389
```

## 11.2 PowerShell

```powershell
New-NetFirewallRule -DisplayName "Block Logon" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Block
```

---

# 12. Limit Logon by Authentication Type

## 12.1 Disable NTLM

Policy:

```
Security Settings → Local Policies → Security Options → Network security: Restrict NTLM
```

## 12.2 CMD

```cmd
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RestrictSendingNTLMTraffic /t REG_DWORD /d 2 /f
```

## 12.3 PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name RestrictSendingNTLMTraffic -Value 2
```

---

# 13. Limit Logon by Time + Workstation + Rights (Combined)

Example: User can log on only:

- Monday–Friday  
- 08:00–17:00  
- Only on PC1 and PC2  
- Only via RDP  
- Only with smart card  

### CMD

```cmd
net user username /times:M-F,08:00-17:00 /workstations:PC1,PC2
net localgroup "Remote Desktop Users" username /add
dsmod user "CN=User,CN=Users,DC=domain,DC=local" -smartcard required
```

### PowerShell

```powershell
Set-ADUser username -LogonWorkstations "PC1","PC2" -SmartcardLogonRequired $true
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "username"
```

---

# 14. Troubleshooting Logon Restrictions

| Issue | Cause | Fix |
|-------|--------|-----|
| Logon denied | Logon hours | Expand allowed hours |
| Logon denied | Wrong workstation | Add workstation |
| Logon denied | Smart card required | Disable requirement |
| Logon denied | NTLM blocked | Allow NTLM or use Kerberos |
| Logon denied | Cached logons disabled | Enable caching |
| Logon denied | Wrong site/subnet | Fix AD Sites |
| Logon denied | UAC elevation | Run as admin |

### CMD

```cmd
whoami /groups
net user username
nltest /dsgetdc:domain.local
```

### PowerShell

```powershell
Get-ADUser username -Properties *
```

---


- A **GitHub‑ready “Complete Domain Security Hardening Guide”**  

Just tell me what you want next.
