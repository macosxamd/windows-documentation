

---

# Installing Active Directory Domain Services  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

Active Directory Domain Services (AD DS) provides:

- Centralized authentication  
- Group Policy  
- Domain controllers  
- Kerberos & LDAP services  
- DNS integration  
- Forests, domains, and trusts  

Installing AD DS involves:

1. Preparing the server  
2. Installing the AD DS role  
3. Promoting the server to a domain controller  
4. Creating a **new forest**, **new domain**, or **additional domain controller**  

---

# 2. Prerequisites

## 2.1 System Requirements

- Windows Server (2000 → 2022)  
- Static IP address  
- Correct DNS configuration  
- Time synchronization (NTP)  
- Administrator privileges  

## 2.2 Network Requirements

- DNS server reachable  
- No duplicate IPs  
- Firewall ports open:  
  - TCP/UDP 53 (DNS)  
  - TCP 88 (Kerberos)  
  - TCP/UDP 389 (LDAP)  
  - TCP 445 (SMB)  
  - TCP 135 (RPC)  
  - Dynamic RPC ports  

### CMD (Check network)

```cmd
ipconfig /all
ping domain.local
nslookup domain.local
```

### PowerShell

```powershell
Test-NetConnection -ComputerName domain.local -Port 389
```

---

# 3. Installing AD DS Role

## 3.1 Windows Server 2000 → 2008 (DCPROMO)

AD DS is installed **during promotion** using `dcpromo.exe`.

---

## 3.2 Windows Server 2008 R2 → 2022 (Role-Based)

### GUI

1. Server Manager  
2. Add Roles  
3. Select **Active Directory Domain Services**  
4. Install  

---

## 3.3 CMD

### Windows Server 2008 → 2012

```cmd
servermanagercmd -install ADDS-Domain-Controller
```

### Windows Server 2012 → 2022

```cmd
dism /online /enable-feature /featurename:AD-Domain-Services
```

---

## 3.4 PowerShell

### Windows Server 2012 → 2022

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

---

# 4. Promoting the Server to a Domain Controller

Promotion creates:

- New forest  
- New domain  
- Child domain  
- Additional domain controller  

---

# 5. Creating a New Forest

## 5.1 GUI (Windows 2000 → 2012)

1. Run **dcpromo.exe**  
2. Select **Create a new domain in a new forest**  
3. Enter forest root domain name  
4. Choose functional level  
5. Install DNS (recommended)  
6. Set Directory Services Restore Mode (DSRM) password  
7. Complete promotion  
8. Reboot  

---

## 5.2 GUI (Server 2012 → 2022)

1. Server Manager → AD DS → **Promote this server**  
2. Select **Add a new forest**  
3. Enter root domain name  
4. Configure options  
5. Complete promotion  

---

## 5.3 CMD (Windows 2000 → 2012)

```cmd
dcpromo /unattend /NewDomain:Forest /NewDomainDNSName:corp.local /SafeModeAdminPassword:Password123
```

---

## 5.4 PowerShell (Server 2012 → 2022)

```powershell
Install-ADDSForest `
    -DomainName "corp.local" `
    -DomainMode "Win2016" `
    -ForestMode "Win2016" `
    -InstallDns `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "Password123" -AsPlainText -Force)
```

---

# 6. Creating a New Domain in an Existing Forest

## 6.1 Child Domain

```powershell
Install-ADDSDomain `
    -NewDomainName "eu" `
    -ParentDomainName "corp.local" `
    -InstallDns `
    -Credential (Get-Credential)
```

---

## 6.2 Tree Domain

```powershell
Install-ADDSDomain `
    -NewDomainName "newtree" `
    -ParentDomainName "corp.local" `
    -DomainType TreeDomain
```

---

# 7. Creating an Additional Domain Controller

### CMD

```cmd
dcpromo /unattend /ReplicaOrNewDomain:Replica /ReplicaDomainDNSName:corp.local /UserName:corp\admin /Password:*
```

### PowerShell

```powershell
Install-ADDSDomainController `
    -DomainName "corp.local" `
    -InstallDns `
    -Credential (Get-Credential)
```

---

# 8. DNS Integration

AD DS requires DNS.

### CMD

```cmd
nslookup _ldap._tcp.dc._msdcs.corp.local
```

### PowerShell

```powershell
Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.corp.local" -Type SRV
```

---

# 9. SYSVOL Replication

## Windows 2000 → 2008  
- **FRS (File Replication Service)**

## Windows Server 2008 → 2022  
- **DFSR (Distributed File System Replication)**

### CMD

```cmd
dfsrdiag replicationstate
```

### PowerShell

```powershell
Get-DfsrBacklog -GroupName "Domain System Volume"
```

---

# 10. FSMO Roles

AD DS creates 5 FSMO roles:

| Role | Scope |
|------|--------|
| Schema Master | Forest |
| Domain Naming Master | Forest |
| PDC Emulator | Domain |
| RID Master | Domain |
| Infrastructure Master | Domain |

### CMD

```cmd
netdom query fsmo
```

### PowerShell

```powershell
Get-ADForest | Select SchemaMaster,DomainNamingMaster
Get-ADDomain | Select PDCEmulator,RIDMaster,InfrastructureMaster
```

---

# 11. Post‑Installation Tasks

- Verify DNS  
- Verify replication  
- Configure sites/subnets  
- Create admin groups  
- Harden domain controller  
- Configure backups  
- Enable auditing  

### CMD

```cmd
dcdiag /v
repadmin /replsummary
```

### PowerShell

```powershell
Get-ADDomainController -Filter *
```

---

# 12. Security Hardening

### CMD (Firewall)

```cmd
netsh advfirewall firewall add rule name="Allow LDAP" dir=in action=allow protocol=TCP localport=389
```

### PowerShell

```powershell
New-NetFirewallRule -DisplayName "Allow LDAP" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow
```

---

# 13. Demoting a Domain Controller

### CMD

```cmd
dcpromo /unattend /demote
```

### PowerShell

```powershell
Uninstall-ADDSDomainController -DemoteOperationMasterRole -ForceRemoval
```

---

# 14. Troubleshooting

| Issue | Cause | Fix |
|-------|--------|-----|
| Cannot promote | DNS misconfigured | Fix SRV records |
| Kerberos errors | Time skew | Fix NTP |
| Replication failure | Site link issues | Fix AD Sites |
| SYSVOL not replicating | FRS/DFSR error | Check event logs |
| Access denied | Wrong credentials | Use Enterprise Admin |

### CMD

```cmd
dcdiag /v
repadmin /showrepl
nltest /dsgetdc:corp.local
```

### PowerShell

```powershell
Get-EventLog -LogName "Directory Service" -Newest 20
```

---

# 15. Complete AD DS Installation Reference Table

| Feature | Supported OS | CMD | PowerShell |
|---------|--------------|-----|------------|
| Forest Creation | 2000 → 2022 | ✔ | ✔ (2012+) |
| Domain Creation | 2000 → 2022 | ✔ | ✔ |
| ADC Creation | 2000 → 2022 | ✔ | ✔ |
| DNS Integration | 2000 → 2022 | ✔ | ✔ |
| SYSVOL Replication | 2000 → 2022 | ✔ | ✔ |
| FSMO Management | 2000 → 2022 | ✔ | ✔ |

---

