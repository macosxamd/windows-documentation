

---

# Creating a Child Domain  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.





---

## 1. Overview

A **child domain** is a domain that exists under a parent domain within the same Active Directory forest.

Example:

```
Parent domain: corp.local
Child domain:  eu.corp.local
```

Child domains provide:

- Delegated administration  
- Geographic separation  
- Organizational separation  
- Independent domain policies  
- Distributed authentication  

Child domains share:

- Forest schema  
- Global Catalog  
- Trust relationships (automatic two‑way transitive)  

---

## 2. Requirements

### 2.1 Technical Requirements

- Windows Server (2000 → 2022)  
- Static IP address  
- DNS resolution of parent domain  
- Network connectivity to parent DC  
- Correct time synchronization (NTP)  
- Enterprise Admin credentials  
- Schema Master reachable  

### 2.2 Functional Requirements

- Forest functional level supports child domains  
- AD DS installed  
- DNS configured  
- No replication issues in parent domain  

---

## 3. DNS Requirements

Child domain creation **depends heavily on DNS**.

### Required DNS Records

| Record | Purpose |
|--------|---------|
| A | DC hostname → IP |
| SRV | LDAP, Kerberos, GC service discovery |
| CNAME | GUID-based DC alias |

### CMD (Check DNS)

```cmd
nslookup _ldap._tcp.corp.local
nslookup _ldap._tcp.dc._msdcs.corp.local
```

### PowerShell

```powershell
Resolve-DnsName -Name "_ldap._tcp.dc._msdcs.corp.local" -Type SRV
```

---

## 4. Preparing the Forest (Older OS)

### Windows 2000 / 2003 / 2008

Run ADPREP on the **Schema Master**:

### CMD

```cmd
adprep /forestprep
adprep /domainprep
```

### PowerShell

```powershell
Start-Process "adprep.exe" -ArgumentList "/forestprep"
Start-Process "adprep.exe" -ArgumentList "/domainprep"
```

---

# 5. Creating a Child Domain

## 5.1 GUI Method (Windows 2000 → Server 2012)

1. Install **Active Directory Domain Services**  
2. Run **dcpromo.exe**  
3. Select:  
   **Create a new domain in an existing forest**  
4. Choose:  
   **Child Domain**  
5. Enter parent domain name (e.g., `corp.local`)  
6. Enter child domain prefix (e.g., `eu`)  
7. Provide Enterprise Admin credentials  
8. Choose site  
9. Install DNS (optional)  
10. Complete promotion  
11. Reboot  

---

## 5.2 GUI Method (Server 2012 → 2022)

1. Server Manager → Add Roles → AD DS  
2. Click **Promote this server to a domain controller**  
3. Select:  
   **Add a new domain to an existing forest**  
4. Choose:  
   **Child Domain**  
5. Enter parent domain  
6. Enter child domain name  
7. Provide credentials  
8. Configure DNS  
9. Complete promotion  

---

# 6. Creating Child Domain (CMD)

### Windows 2000 → Server 2012

```cmd
dcpromo /unattend /ReplicaOrNewDomain:ChildDomain /NewDomain:Child /ParentDomainDNSName:corp.local /NewDomainDNSName:eu.corp.local /UserName:corp\administrator /Password:*
```

---

# 7. Creating Child Domain (PowerShell)

### Windows Server 2012 → 2022

```powershell
Install-WindowsFeature AD-Domain-Services
Import-Module ADDSDeployment

Install-ADDSDomain `
    -NewDomainName "eu" `
    -ParentDomainName "corp.local" `
    -DomainMode "Win2016" `
    -InstallDns `
    -Credential (Get-Credential)
```

---

# 8. Replication Behavior

Child domains replicate:

- **Schema** (forest-wide)  
- **Configuration partition** (forest-wide)  
- **Domain partition** (child domain only)  
- **DNS application partitions**  
- **SYSVOL**  

### CMD (Check replication)

```cmd
repadmin /replsummary
repadmin /showrepl
```

### PowerShell

```powershell
Get-ADReplicationPartnerMetadata -Target "DC1"
```

---

# 9. Global Catalog Considerations

Child domains **should have at least one GC**.

### GUI

Active Directory Sites and Services → NTDS Settings → Check **Global Catalog**

### CMD

```cmd
nltest /server:DC1 /dsgetdc:corp.local
```

### PowerShell

```powershell
Get-ADDomainController -Filter * | Select Name,IsGlobalCatalog
```

---

# 10. DNS Delegation

When creating a child domain, the parent DNS zone must delegate the child zone.

Example:

```
corp.local delegates eu.corp.local → child DC
```

### CMD

```cmd
dnscmd corp-dc /ZoneAdd eu.corp.local /DsDelegation 10.0.0.20
```

### PowerShell

```powershell
Add-DnsServerZoneDelegation -Name "eu" -ParentZoneName "corp.local" -ChildServer 10.0.0.20
```

---

# 11. Trust Relationships

Child domains automatically create:

- **Two‑way transitive trust** with parent domain  
- **Forest-wide trust**  

### CMD

```cmd
nltest /trusted_domains
```

### PowerShell

```powershell
Get-ADTrust -Filter *
```

---

# 12. Sites and Services

Child domains should be placed in correct AD sites.

### CMD

```cmd
nltest /dsgetsite
```

### PowerShell

```powershell
Get-ADReplicationSite -Filter *
```

---

# 13. Security Considerations

## 13.1 Harden Child Domain Controllers

- Disable SMBv1  
- Enable LDAP signing  
- Enable LDAPS  
- Restrict DC logon rights  
- Use DC isolation firewall rules  
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

# 14. Demoting a Child Domain Controller

## 14.1 GUI

Run **dcpromo.exe** (2000 → 2012)  
Server Manager → **Demote** (2012 → 2022)

## 14.2 CMD

```cmd
dcpromo /unattend /demote
```

## 14.3 PowerShell

```powershell
Uninstall-ADDSDomainController -DemoteOperationMasterRole -ForceRemoval
```

---

# 15. Troubleshooting Child Domain Creation

| Issue | Cause | Fix |
|-------|--------|-----|
| Cannot contact parent domain | DNS misconfigured | Fix SRV records |
| Schema mismatch | ADPREP not run | Run forestprep/domainprep |
| Replication failure | Site link issues | Fix AD Sites |
| Kerberos errors | Time skew | Fix NTP |
| DNS delegation missing | Parent DNS not configured | Add delegation |
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

# 16. Complete Child Domain Reference Table

| Feature | Supported OS | CMD | PowerShell |
|---------|--------------|-----|------------|
| Child Domain Creation | 2000 → 2022 | ✔ | ✔ (2012+) |
| DNS Delegation | 2000 → 2022 | ✔ | ✔ |
| Replication Diagnostics | 2000 → 2022 | ✔ | ✔ |
| Trust Verification | 2000 → 2022 | ✔ | ✔ |
| GC Configuration | 2000 → 2022 | ✔ | ✔ |

---

