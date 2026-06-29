
---

# Joining a Windows Client to a Domain  
**Complete Technical Documentation (Windows 2000 → Windows 11)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

Joining a client to a domain allows:

- Centralized authentication  
- Group Policy enforcement  
- Access to domain resources  
- Kerberos security  
- Roaming profiles and folder redirection  
- Enterprise management  

Supported OS:

- Windows 2000 Professional  
- Windows XP  
- Windows Vista  
- Windows 7  
- Windows 8 / 8.1  
- Windows 10  
- Windows 11  

---

# 2. Requirements

## 2.1 Network Requirements

- Client must reach the domain controller  
- DNS must point to the **domain controller’s DNS**  
- Time difference must be < 5 minutes (Kerberos requirement)  
- Firewall must allow domain traffic  

### CMD (Check connectivity)

```cmd
ping dc01.domain.local
nslookup domain.local
ipconfig /all
```

### PowerShell

```powershell
Test-NetConnection -ComputerName dc01.domain.local -Port 389
```

---

## 2.2 DNS Requirement (Critical)

**The client’s primary DNS server MUST be the domain controller.**

Example:

```
Preferred DNS: 10.0.0.10 (DC)
Alternate DNS: <empty or another DC>
```

### CMD

```cmd
netsh interface ip set dns "Ethernet" static 10.0.0.10
```

### PowerShell

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.10
```

---

# 3. GUI Method (All Windows Versions)

1. Open **System Properties**  
2. Click **Change settings**  
3. Click **Change…**  
4. Select **Domain**  
5. Enter domain name:  
   ```
   domain.local
   ```
6. Enter domain credentials  
7. Reboot when prompted  

---

# 4. CMD Method (Windows XP → Windows 11)

### Join domain

```cmd
netdom join %computername% /domain:domain.local /userd:domain\admin /passwordd:*
```

### Leave domain

```cmd
netdom remove %computername% /domain:domain.local
```

### Rename + join domain

```cmd
netdom renamecomputer %computername% /newname:PC01 /userd:domain\admin /passwordd:* /reboot:5
```

---

# 5. PowerShell Method (Windows 7 → Windows 11)

### Join domain

```powershell
Add-Computer -DomainName "domain.local" -Credential (Get-Credential) -Restart
```

### Join domain + specify OU

```powershell
Add-Computer -DomainName "domain.local" `
    -Credential (Get-Credential) `
    -OUPath "OU=Workstations,DC=domain,DC=local" `
    -Restart
```

### Leave domain

```powershell
Remove-Computer -UnjoinDomainCredential (Get-Credential) -Restart
```

---

# 6. Windows 2000 / XP Special Notes

Older systems require:

- NetBIOS name resolution  
- DNS suffix manually configured  
- LMHOSTS sometimes required  

### CMD (Set DNS suffix)

```cmd
ipconfig /setclassid "Local Area Connection" domain.local
```

### LMHOSTS example

```
10.0.0.10   dc01   #PRE #DOM:domain
```

---

# 7. OU Placement

By default, clients join:

```
CN=Computers,DC=domain,DC=local
```

To place clients in a specific OU:

### CMD

```cmd
netdom join PC01 /domain:domain.local /OU:"OU=Workstations,DC=domain,DC=local"
```

### PowerShell

```powershell
Add-Computer -DomainName "domain.local" -OUPath "OU=Workstations,DC=domain,DC=local"
```

---

# 8. Firewall Requirements

Clients must allow outbound:

- TCP/UDP 53 (DNS)  
- TCP 88 (Kerberos)  
- TCP/UDP 389 (LDAP)  
- TCP 445 (SMB)  
- TCP 135 (RPC)  
- Dynamic RPC ports  

### CMD

```cmd
netsh advfirewall firewall add rule name="Allow-Domain" dir=out action=allow protocol=TCP localport=88,389,445
```

### PowerShell

```powershell
New-NetFirewallRule -DisplayName "Allow-Domain" -Direction Outbound -Protocol TCP -LocalPort 88,389,445 -Action Allow
```

---

# 9. Verify Domain Join

### CMD

```cmd
systeminfo | find "Domain"
whoami /fqdn
```

### PowerShell

```powershell
(Get-WmiObject Win32_ComputerSystem).Domain
```

---

# 10. Post‑Join Tasks

- Apply Group Policy  
- Move computer to correct OU  
- Install required software  
- Configure security baselines  

### CMD (Force GP update)

```cmd
gpupdate /force
```

### PowerShell

```powershell
Invoke-GPUpdate
```

---

# 11. Troubleshooting Domain Join

| Issue | Cause | Fix |
|-------|--------|-----|
| Cannot find domain | Wrong DNS | Point DNS to DC |
| Access denied | Wrong credentials | Use domain admin |
| Time sync error | Clock skew | Sync NTP |
| RPC server unavailable | Firewall | Open ports |
| Domain not reachable | Network issue | Check routing |
| Trust relationship failed | Password mismatch | Rejoin domain |

---

## 11.1 CMD Diagnostics

```cmd
nltest /dsgetdc:domain.local
nltest /sc_verify:domain.local
dcdiag /test:dns
```

## 11.2 PowerShell Diagnostics

```powershell
Test-ComputerSecureChannel
Resolve-DnsName domain.local
```

---

# 12. Complete Domain Join Reference Table

| Method | OS Support | CMD | PowerShell |
|--------|------------|-----|------------|
| GUI | 2000 → 11 | ✔ | ✔ |
| netdom | XP → 11 | ✔ | ✘ |
| Add-Computer | 7 → 11 | ✘ | ✔ |
| OU placement | XP → 11 | ✔ | ✔ |
| DNS prep | 2000 → 11 | ✔ | ✔ |

---

