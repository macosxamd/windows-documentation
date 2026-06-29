

---

# Additional Domain Controllers  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

# 1. Overview

An **Additional Domain Controller (ADC)** is a server that hosts a writable copy of the Active Directory database for an existing domain.

Adding ADCs provides:

- Redundancy  
- Load balancing  
- Faster authentication  
- Fault tolerance  
- Distributed replication  
- Local logon capability in remote sites  

---

# 2. Requirements

## 2.1 System Requirements

- Windows Server (2000 → 2022)  
- Static IP address  
- Correct DNS configuration  
- Time synchronization (NTP)  
- Domain membership (optional before promotion)  

## 2.2 Functional Requirements

- Existing domain controller reachable  
- Correct domain credentials  
- DNS SRV records resolvable  
- SYSVOL replication functioning  

---

# 3. DNS Requirements

Active Directory **requires DNS**.

### Required DNS Records

| Record | Purpose |
|--------|---------|
| A | DC hostname → IP |
| SRV | LDAP, Kerberos, GC service discovery |
| CNAME | GUID-based DC alias |

### CMD (Check DNS)

```cmd
nslookup _ldap._tcp.dc._msdcs.domain.local
```

### PowerShell

```powershell
Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.domain.local" -Type SRV
```

---

# 4. Preparing the Forest/Domain (Older OS)

### Windows 2000 / 2003 / 2008

Before adding newer DCs, run:

### CMD

```cmd
adprep /forestprep
adprep /domainprep
```

### PowerShell (Server 2012 → 2022)

```powershell
Start-Process "adprep.exe" -ArgumentList "/forestprep"
Start-Process "adprep.exe" -ArgumentList "/domainprep"
```

---

# 5. Adding an Additional Domain Controller

## 5.1 GUI Method (All Versions)

1. Install **Active Directory Domain Services**  
2. Run **dcpromo.exe** (Windows 2000 → Server 2012)  
3. Select **Additional Domain Controller for an existing domain**  
4. Provide domain credentials  
5. Choose site  
6. Install DNS (optional)  
7. Complete promotion  
8. Reboot  

### Windows Server 2012 → 2022

Use **Server Manager → Add Roles → AD DS → Promote**.

---

# 6. Adding ADC (CMD)

### Windows 2000 → Server 2012

```cmd
dcpromo /unattend /ReplicaOrNewDomain:Replica /ReplicaDomainDNSName:domain.local /UserName:domain\admin /Password:*
```

### Windows Server 2016 → 2022

```cmd
dcpromo.exe
```

*(Still supported for backward compatibility)*

---

# 7. Adding ADC (PowerShell)

### Windows Server 2012 → 2022

```powershell
Install-WindowsFeature AD-Domain-Services
Import-Module ADDSDeployment
Install-ADDSDomainController -DomainName "domain.local" -Credential (Get-Credential) -InstallDns
```

---

# 8. Replication

Active Directory uses **multi‑master replication**.

## 8.1 Replication Components

- **NTDS.DIT** database  
- **SYSVOL**  
- **DNS application partitions**  
- **Site links**  
- **KCC (Knowledge Consistency Checker)**  

## 8.2 Replication Protocols

- **RPC over TCP** (default)  
- **SMTP** (schema only)  

---

# 9. Replication Diagnostics

### CMD

```cmd
repadmin /replsummary
repadmin /showrepl
repadmin /syncall /AeD
dcdiag /test:replications
```

### PowerShell

```powershell
Get-ADReplicationPartnerMetadata -Target "DC1"
Get-ADReplicationFailure -Target "DC1"
```

---

# 10. SYSVOL Replication

## 10.1 Windows 2000 → 2003

- **FRS (File Replication Service)**

## 10.2 Windows Server 2008 → 2022

- **DFSR (Distributed File System Replication)**

### CMD (Check SYSVOL)

```cmd
net share
```

### PowerShell

```powershell
Get-SmbShare | Where-Object Name -eq "SYSVOL"
```

---

# 11. FSMO Roles and ADCs

FSMO roles are **not** automatically transferred when adding ADCs.

## 11.1 FSMO Roles

| Role | Scope |
|------|--------|
| Schema Master | Forest |
| Domain Naming Master | Forest |
| PDC Emulator | Domain |
| RID Master | Domain |
| Infrastructure Master | Domain |

### CMD (Check FSMO)

```cmd
netdom query fsmo
```

### PowerShell

```powershell
Get-ADForest | Select SchemaMaster,DomainNamingMaster
Get-ADDomain | Select PDCEmulator,RIDMaster,InfrastructureMaster
```

---

# 12. Global Catalog

A **Global Catalog (GC)** stores partial attributes for all objects.

### Enable GC (GUI)

Active Directory Sites and Services → NTDS Settings → Check **Global Catalog**

### CMD

```cmd
nltest /server:DC1 /dsgetdc:domain.local
```

### PowerShell

```powershell
Get-ADDomainController -Filter * | Select Name,IsGlobalCatalog
```

---

# 13. Sites and Services

Sites optimize replication and authentication.

## 13.1 Components

- Sites  
- Subnets  
- Site links  
- Bridgehead servers  

### CMD

```cmd
nltest /dsgetsite
```

### PowerShell

```powershell
Get-ADReplicationSite -Filter *
```

---

# 14. Security Considerations

## 14.1 Harden ADCs

- Disable SMBv1  
- Enable LDAP signing  
- Enable LDAPS  
- Restrict DC logon rights  
- Use DC isolation (firewall rules)  
- Monitor replication failures  

### CMD (Firewall)

```cmd
netsh advfirewall firewall add rule name="Allow LDAP" dir=in action=allow protocol=TCP localport=389
```

### PowerShell

```powershell
New-NetFirewallRule -DisplayName "Allow LDAP" -Direction Inbound -Protocol TCP -LocalPort 389 -Action Allow
```

---

# 15. Backup & Recovery

## 15.1 System State Backup

### CMD

```cmd
wbadmin start systemstatebackup -backupTarget:D:
```

### PowerShell

```powershell
wbadmin start systemstatebackup -backupTarget:D:
```

---

# 16. Demoting a Domain Controller

## 16.1 GUI

Run **dcpromo.exe** (2000 → 2012)  
Server Manager → **Demote** (2012 → 2022)

## 16.2 CMD

```cmd
dcpromo /unattend /demote
```

## 16.3 PowerShell

```powershell
Uninstall-ADDSDomainController -DemoteOperationMasterRole -ForceRemoval
```

---

# 17. Troubleshooting

| Issue | Cause | Fix |
|-------|--------|-----|
| Replication failure | DNS broken | Fix SRV records |
| SYSVOL not replicating | FRS/DFSR error | Check event logs |
| Cannot promote | Time skew | Fix NTP |
| Kerberos errors | SPN issues | Reset SPNs |
| Slow logons | Wrong site/subnet | Fix AD Sites |

### CMD

```cmd
dcdiag /v
repadmin /showrepl
nltest /dsgetdc:domain.local
```

### PowerShell

```powershell
Get-EventLog -LogName "Directory Service" -Newest 20
```

---

