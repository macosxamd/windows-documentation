
---

# Roaming & Mandatory Profiles  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

Windows supports multiple profile types:

- **Roaming Profiles** — user profile stored on a server and downloaded at logon.  
- **Mandatory Profiles** — read‑only roaming profiles; changes are discarded at logoff.  
- **Local Profiles** — stored on the workstation.  
- **Temporary Profiles** — used when roaming profile fails.

Roaming and mandatory profiles are used to:

- Provide consistent user environments across devices  
- Enforce standardized configurations  
- Reduce support overhead  
- Improve user mobility  

---

# 2. Roaming Profiles

## 2.1 How Roaming Profiles Work

1. User logs on  
2. Workstation downloads profile from server  
3. User works normally  
4. At logoff, profile changes are uploaded back to server  
5. Profile is available on any domain‑joined workstation

Profile path example:

```
\\fileserver\profiles\%username%
```

---

# 3. Mandatory Profiles

Mandatory profiles are **roaming profiles made read‑only**.

- User can log on normally  
- Changes are **not saved**  
- Profile resets at every logon  
- Useful for kiosks, classrooms, shared computers  

Mandatory profile file:

```
NTUSER.MAN
```

Instead of:

```
NTUSER.DAT
```

---

# 4. Profile Storage Structure

Typical roaming profile folder:

```
\\server\profiles\jdoe\
    AppData\
    Desktop\
    Documents\
    NTUSER.DAT
```

Mandatory profile folder:

```
\\server\profiles\mandatory\
    AppData\
    Desktop\
    Documents\
    NTUSER.MAN
```

---

# 5. Requirements

- Windows Server (2000 → 2022)  
- SMB share for profiles  
- Correct NTFS permissions  
- Correct share permissions  
- DFS‑R or FRS replication (optional)  
- Domain membership  

---

# 6. Creating the Profile Share

## 6.1 Share Permissions (Recommended)

- **Everyone: Full Control**  
- Enforce security via NTFS

## 6.2 NTFS Permissions (Recommended)

| Principal | Permission |
|----------|------------|
| Administrators | Full Control |
| SYSTEM | Full Control |
| User (SELF) | Full Control on own folder |

---

## 6.3 CMD (Create share)

```cmd
net share profiles=D:\Profiles /grant:Everyone,FULL
```

## 6.4 PowerShell (Create share)

```powershell
New-SmbShare -Name "profiles" -Path "D:\Profiles" -FullAccess "Everyone"
```

---

# 7. Configure Roaming Profiles

## 7.1 GUI (ADUC)

1. Open **Active Directory Users and Computers**  
2. User → **Properties**  
3. **Profile** tab  
4. Set **Profile Path**:  
   ```
   \\server\profiles\%username%
   ```

---

## 7.2 CMD

```cmd
dsmod user "CN=John Doe,CN=Users,DC=domain,DC=local" -profile "\\server\profiles\jdoe"
```

---

## 7.3 PowerShell

```powershell
Set-ADUser jdoe -ProfilePath "\\server\profiles\jdoe"
```

---

# 8. Configure Mandatory Profiles

## 8.1 Steps

1. Create a roaming profile normally  
2. Log in as the template user  
3. Configure desktop, settings, etc.  
4. Log off  
5. On the server, rename:

```
NTUSER.DAT → NTUSER.MAN
```

6. Assign the profile path to users:

```
\\server\profiles\mandatory
```

---

## 8.2 CMD (Rename)

```cmd
rename \\server\profiles\mandatory\NTUSER.DAT NTUSER.MAN
```

## 8.3 PowerShell (Rename)

```powershell
Rename-Item "\\server\profiles\mandatory\NTUSER.DAT" "NTUSER.MAN"
```

---

# 9. GPO Settings for Roaming Profiles

Path:

```
Computer Configuration
 └── Administrative Templates
     └── System
         └── User Profiles
```

Key settings:

- **Set roaming profile path for all users**  
- **Only allow local user profiles**  
- **Prevent roaming profile changes from propagating**  
- **Delete cached copies of roaming profiles**  
- **Exclude directories in roaming profile**  
- **Wait for network at computer startup**  

---

## 9.1 CMD (Local GPO export)

```cmd
secedit /export /cfg sec.cfg
```

## 9.2 PowerShell (View GPO settings)

```powershell
Get-GPRegistryValue -Name "User Profile Policy"
```

---

# 10. Folder Redirection (Optional but Recommended)

Redirect folders to reduce profile size:

- Documents  
- Desktop  
- AppData  
- Pictures  
- Videos  

This improves logon/logoff speed.

---

## 10.1 CMD (Create redirection share)

```cmd
net share redirect=D:\Redirect /grant:Everyone,FULL
```

## 10.2 PowerShell

```powershell
New-SmbShare -Name "redirect" -Path "D:\Redirect" -FullAccess "Everyone"
```

---

# 11. Profile Version Compatibility

| Windows Version | Profile Version |
|-----------------|-----------------|
| XP / 2003 | V1 |
| Vista / 2008 | V2 |
| Win7 / 2008 R2 | V2 |
| Win8 / 2012 | V3 |
| Win8.1 / 2012 R2 | V4 |
| Win10 / 2016 | V5 |
| Win10 1709+ | V6 |
| Win11 / 2022 | V7 |

Profiles are **not backward compatible**.

---

# 12. Replication (DFS‑R / FRS)

Roaming profiles can be replicated using:

- **FRS** (2000 → 2008)  
- **DFS‑R** (2008 → 2022)

### CMD

```cmd
dfsrdiag replicationstate
```

### PowerShell

```powershell
Get-DfsrBacklog -GroupName "Profiles"
```

---

# 13. Troubleshooting

| Issue | Cause | Fix |
|-------|--------|-----|
| Temporary profile | NTUSER.DAT corrupt | Delete profile folder |
| Slow logons | Large profile | Use folder redirection |
| Profile not loading | Permissions wrong | Fix NTFS ACL |
| Mandatory profile not applied | NTUSER.MAN missing | Rename file |
| Profile version mismatch | OS version conflict | Use correct OS |
| Profile not saving | Network issue | Check SMB connectivity |

---

## 13.1 CMD Diagnostics

```cmd
whoami /user
echo %USERPROFILE%
net share
```

## 13.2 PowerShell Diagnostics

```powershell
Get-SmbSession
Get-ADUser jdoe -Properties ProfilePath
```

---

# 14. Complete Roaming & Mandatory Profile Reference Table

| Feature | Roaming | Mandatory |
|---------|---------|-----------|
| Saves changes | ✔ | ✘ |
| NTUSER file | NTUSER.DAT | NTUSER.MAN |
| Multi‑PC support | ✔ | ✔ |
| Customizable | ✔ | Limited |
| Best for | Normal users | Kiosks, classrooms |

---
