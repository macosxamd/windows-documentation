
---

# HASH Rules (Executable File Rules)  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

**HASH Rules** are a type of executable control used in:

- **Software Restriction Policies (SRP)** — Windows XP → Windows Server 2022  
- **AppLocker** — Windows 7 → Windows Server 2022  

HASH rules identify an executable **by its cryptographic hash**, not by:

- Path  
- Publisher  
- File name  
- File location  

This makes HASH rules extremely **precise** but also **high‑maintenance**, because **any change to the file modifies the hash**.

---

# 2. When to Use HASH Rules

HASH rules are ideal when:

- You must block or allow **one specific executable**  
- The executable **does not have a reliable publisher signature**  
- The executable **may move between folders**  
- You need **strong enforcement** that cannot be bypassed by renaming or relocating files  

HASH rules are **not ideal** when:

- The executable updates frequently  
- You need broad rules (use path or publisher rules instead)  
- You manage large environments  

---

# 3. How HASH Rules Work

Windows calculates a **SHA‑256 hash** (older systems used SHA‑1) of the executable.

Example hash:

```
A3B4C5D6E7F8A9B0C1D2E3F4A5B6C7D8E9F0A1B2C3D4E5F6A7B8C9D0E1F2A3B4
```

This hash is stored in:

- SRP policy  
- AppLocker policy  

When a user attempts to run the executable:

1. Windows calculates the hash of the file  
2. Compares it to the stored hash  
3. Allows or blocks execution  

---

# 4. Software Restriction Policies (SRP) HASH Rules

Supported on:

- Windows XP  
- Windows Vista  
- Windows 7  
- Windows 8  
- Windows 10  
- Windows 11  
- Windows Server 2003 → 2022  

---

## 4.1 GPO Path

```
Computer Configuration
 └── Windows Settings
     └── Security Settings
         └── Software Restriction Policies
             └── Additional Rules
```

---

## 4.2 Creating HASH Rules (GUI)

1. Open **gpedit.msc** or Group Policy Management  
2. Navigate to SRP → Additional Rules  
3. Right‑click → **New Hash Rule**  
4. Browse to executable  
5. Choose **Disallowed** or **Unrestricted**  
6. Apply  

---

## 4.3 CMD (Generate Hash)

Windows does not include a built‑in hash generator until Windows 7.

### Windows XP / Server 2003

Use `fciv.exe` (File Checksum Integrity Verifier):

```cmd
fciv.exe C:\Path\Program.exe -sha1
```

### Windows 7 → Server 2022

```cmd
certutil -hashfile C:\Path\Program.exe SHA256
```

---

## 4.4 PowerShell (Generate Hash)

```powershell
Get-FileHash "C:\Path\Program.exe" -Algorithm SHA256
```

---

## 4.5 CMD (Create SRP HASH Rule via Registry)

```cmd
reg add "HKLM\Software\Policies\Microsoft\Windows\Safer\CodeIdentifiers\0\Hashes" /v "Program.exe" /t REG_BINARY /d <HASH> /f
```

---

## 4.6 PowerShell (Create SRP HASH Rule)

```powershell
$hash = (Get-FileHash "C:\Path\Program.exe" -Algorithm SHA256).Hash
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\Safer\CodeIdentifiers\0\Hashes\Program.exe" -Force
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\Safer\CodeIdentifiers\0\Hashes\Program.exe" -Name "ItemData" -Value $hash
```

---

# 5. AppLocker HASH Rules

Supported on:

- Windows 7 Enterprise / Ultimate  
- Windows 8 / 8.1 Enterprise  
- Windows 10 Enterprise  
- Windows 11 Enterprise  
- Windows Server 2008 R2 → 2022  

AppLocker supports HASH rules for:

- EXE  
- DLL  
- MSI  
- Scripts  
- Packaged apps  

---

## 5.1 GPO Path

```
Computer Configuration
 └── Windows Settings
     └── Security Settings
         └── Application Control Policies
             └── AppLocker
                 ├── Executable Rules
                 ├── Windows Installer Rules
                 ├── Script Rules
                 └── Packaged App Rules
```

---

## 5.2 Creating HASH Rules (GUI)

1. Open GPMC  
2. Navigate to AppLocker → Executable Rules  
3. Right‑click → **Create New Rule**  
4. Choose **Hash**  
5. Browse to executable  
6. Choose **Allow** or **Deny**  
7. Apply  

---

## 5.3 CMD (Generate Hash)

```cmd
certutil -hashfile C:\Path\Program.exe SHA256
```

---

## 5.4 PowerShell (Generate Hash)

```powershell
Get-FileHash "C:\Path\Program.exe" -Algorithm SHA256
```

---

## 5.5 PowerShell (Create AppLocker HASH Rule)

```powershell
$hash = (Get-FileHash "C:\Path\Program.exe" -Algorithm SHA256).Hash
Add-AppLockerFileRule -RuleType Hash -FilePath "C:\Path\Program.exe" -User "Everyone" -Action Deny
```

---

# 6. HASH Rule Behavior

| Behavior | Description |
|----------|-------------|
| File changes break rule | Any modification changes hash |
| File relocation does NOT break rule | Hash stays same |
| File rename does NOT break rule | Hash stays same |
| Updates require new rule | Updated file = new hash |
| Very strong enforcement | Cannot bypass by moving/renaming |

---

# 7. HASH Rule Limitations

- High maintenance  
- Must update rules after every patch  
- Not ideal for large environments  
- Not ideal for frequently updated apps  
- Requires hash generation tools  

---

# 8. Best Practices

- Use HASH rules only for **static executables**  
- Prefer **Publisher rules** for signed software  
- Prefer **Path rules** for internal apps  
- Combine HASH rules with AppLocker  
- Use GPO to deploy rules centrally  
- Document all hashes  

---

# 9. Troubleshooting HASH Rules

| Issue | Cause | Fix |
|-------|--------|-----|
| App blocked unexpectedly | Hash changed | Recalculate hash |
| Rule not applied | Wrong GPO scope | Link GPO correctly |
| Rule ignored | AppLocker service disabled | Enable service |
| SRP not working | Enforcement disabled | Enable SRP |
| Hash mismatch | Wrong algorithm | Use SHA256 |

---

## 9.1 CMD Diagnostics

```cmd
gpresult /r
certutil -hashfile Program.exe SHA256
```

## 9.2 PowerShell Diagnostics

```powershell
Get-AppLockerPolicy -Effective -Xml
Get-FileHash Program.exe -Algorithm SHA256
```

---

# 10. Complete HASH Rule Reference Table

| Feature | SRP | AppLocker |
|---------|-----|-----------|
| EXE | ✔ | ✔ |
| DLL | ✘ | ✔ |
| MSI | ✔ | ✔ |
| Scripts | ✘ | ✔ |
| Packaged apps | ✘ | ✔ |
| SHA‑256 | ✔ | ✔ |
| SHA‑1 | ✔ (legacy) | ✔ (legacy) |
| GPO support | ✔ | ✔ |
| Enterprise‑grade | Moderate | High |

---

