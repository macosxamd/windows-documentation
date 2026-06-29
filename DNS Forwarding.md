

---

# DNS Forwarding  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

DNS Forwarding allows a DNS server to send queries it cannot resolve locally to another DNS server.  
Forwarding improves:

- Query resolution speed  
- Control over external DNS traffic  
- Security and isolation  
- WAN bandwidth optimization  
- Centralized DNS management  

Windows supports:

- **Standard Forwarders**  
- **Conditional Forwarders**  
- **Recursive Forwarding**  
- **Forwarding Timeouts**  
- **Root Hints fallback**  

---

## 2. Types of DNS Forwarding

### 2.1 Standard Forwarders

Used for **all queries** the server cannot resolve locally.

Example: Forward all external queries to ISP DNS.

### 2.2 Conditional Forwarders

Used for **specific domains**.

Example: Forward `corp.remote.local` queries to `10.10.10.10`.

### 2.3 Stub Zones (Forwarding Alternative)

Stub zones replicate only NS records and are used for cross‑forest resolution.

### 2.4 Forwarding vs Root Hints

| Feature | Description |
|--------|-------------|
| Forwarders | Preferred external DNS servers |
| Root Hints | Used when forwarders fail or are disabled |

---

# 3. DNS Forwarding Requirements

- DNS Server role installed  
- Network connectivity to forwarder  
- Correct firewall rules  
- No conflicting conditional forwarders  
- Root hints present (optional fallback)  

---

# 4. Configure Standard Forwarders

## 4.1 GUI (Windows 2000 → Server 2022)

1. Open **DNS Manager**  
2. Right‑click server → **Properties**  
3. **Forwarders** tab  
4. Add IP addresses of forwarder DNS servers  
5. Apply

---

## 4.2 CMD (Windows 2000 → Server 2022)

```cmd
dnscmd /ResetForwarders 8.8.8.8 1
```

- `8.8.8.8` = forwarder  
- `1` = timeout (seconds)

Add multiple forwarders:

```cmd
dnscmd /ResetForwarders 8.8.8.8 1 1.1.1.1 1
```

---

## 4.3 PowerShell (Server 2012 → 2022)

```powershell
Set-DnsServerForwarder -IPAddress 8.8.8.8,1.1.1.1
```

View forwarders:

```powershell
Get-DnsServerForwarder
```

Remove forwarders:

```powershell
Remove-DnsServerForwarder -IPAddress 8.8.8.8
```

---

# 5. Configure Conditional Forwarders

## 5.1 GUI

1. DNS Manager  
2. Right‑click **Conditional Forwarders**  
3. **New Conditional Forwarder**  
4. Enter domain name  
5. Add forwarder IPs  
6. Save

---

## 5.2 CMD

```cmd
dnscmd /ZoneAdd corp.remote.local /Forwarder 10.10.10.10
```

Add multiple:

```cmd
dnscmd /ZoneAdd corp.remote.local /Forwarder 10.10.10.10 10.10.10.11
```

---

## 5.3 PowerShell

```powershell
Add-DnsServerConditionalForwarderZone -Name "corp.remote.local" -MasterServers 10.10.10.10,10.10.10.11
```

View:

```powershell
Get-DnsServerConditionalForwarderZone
```

Remove:

```powershell
Remove-DnsServerConditionalForwarderZone -Name "corp.remote.local"
```

---

# 6. Forwarding Timeouts & Behavior

Windows DNS uses:

- **Forwarder timeout** (default: 3 seconds)  
- **Fallback to next forwarder**  
- **Fallback to root hints** (if enabled)  

## 6.1 CMD

```cmd
dnscmd /ResetForwarders 8.8.8.8 5
```

(Timeout = 5 seconds)

## 6.2 PowerShell

```powershell
Set-DnsServerForwarder -IPAddress 8.8.8.8 -Timeout 5
```

---

# 7. Disable Root Hints (Force Forwarding Only)

## 7.1 GUI

DNS Manager → Server → Properties → **Advanced** → Disable recursion

## 7.2 CMD

```cmd
dnscmd /Config /NoRecursion 1
```

## 7.3 PowerShell

```powershell
Set-DnsServerRecursion -Enable $false
```

---

# 8. DNS Forwarding in Multi‑Site Environments

Use conditional forwarders for:

- Cross‑forest trusts  
- Branch office DNS  
- Hybrid cloud DNS  
- Split‑brain DNS  

Example:

```
HQ DNS forwards "branch.local" → 192.168.50.10
Branch DNS forwards "hq.local" → 10.0.0.10
```

---

# 9. DNS Forwarding Security

## 9.1 Best Practices

- Use internal DNS forwarders, not public ones  
- Disable recursion on public‑facing DNS  
- Use DNSSEC where possible  
- Restrict zone transfers  
- Monitor DNS logs  

## 9.2 CMD (Disable zone transfers)

```cmd
dnscmd /ZoneResetSecondaries domain.local /SecureSecondaries 0
```

## 9.3 PowerShell

```powershell
Set-DnsServerPrimaryZone -Name "domain.local" -SecureSecondaries TransferToSecureServers
```

---

# 10. Testing DNS Forwarding

## 10.1 CMD

```cmd
nslookup www.google.com 127.0.0.1
```

Force specific forwarder:

```cmd
nslookup www.google.com 8.8.8.8
```

## 10.2 PowerShell

```powershell
Resolve-DnsName www.google.com -Server 127.0.0.1
```

---

# 11. Troubleshooting DNS Forwarding

| Issue | Cause | Fix |
|-------|--------|-----|
| Forwarding fails | Wrong forwarder IP | Correct IP |
| Slow resolution | Timeout too high | Lower timeout |
| No fallback | Root hints disabled | Enable hints |
| Conditional forwarder ignored | Typo in domain | Correct domain |
| Recursion disabled | Forwarding requires recursion | Enable recursion |
| Firewall blocking | UDP/TCP 53 blocked | Open firewall |

## 11.1 CMD Diagnostics

```cmd
dnscmd /info
dnscmd /enumzones
nslookup
```

## 11.2 PowerShell Diagnostics

```powershell
Get-DnsServerForwarder
Get-DnsServerConditionalForwarderZone
Get-DnsServerDiagnostics
```

---

# 12. Complete DNS Forwarding Reference Table

| Feature | Supported OS | CMD | PowerShell |
|---------|--------------|-----|------------|
| Standard Forwarders | 2000 → 2022 | ✔ | ✔ (2012+) |
| Conditional Forwarders | 2003 → 2022 | ✔ | ✔ |
| Timeout Control | 2000 → 2022 | ✔ | ✔ |
| Disable Recursion | 2000 → 2022 | ✔ | ✔ |
| Root Hints | 2000 → 2022 | ✔ | ✔ |
| Stub Zones | 2003 → 2022 | ✔ | ✔ |

---

