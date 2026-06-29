
---

# Mandatory Background (Forced Desktop Wallpaper)  
**Complete Technical Documentation (Windows 2000 → Windows Server 2022)**  
Includes **CMD + PowerShell** examples in every section.

---

## 1. Overview

A **mandatory background** is a **forced desktop wallpaper** applied through:

- Group Policy  
- Mandatory profiles  
- Registry enforcement  
- File system restrictions  

Users **cannot change** the wallpaper, and attempts to modify it are ignored or reverted.

Mandatory backgrounds are used for:

- Corporate branding  
- Compliance requirements  
- Classroom/kiosk environments  
- Security messaging  
- Standardized user experience  

---

# 2. Requirements

- Windows (2000 → 2022)  
- Active Directory (optional but recommended)  
- Central wallpaper file (BMP or JPG)  
- Read‑only access for users  
- GPO or registry enforcement  

---

# 3. Supported Wallpaper Formats

| OS Version | BMP | JPG | PNG |
|------------|-----|-----|-----|
| Windows 2000 | ✔ | ✘ | ✘ |
| Windows XP | ✔ | ✔ | ✘ |
| Windows Vista → 2022 | ✔ | ✔ | ✔ |

**Recommendation:** Use **JPG** for performance.

---

# 4. Preparing the Wallpaper File

Place the wallpaper in a **read‑only network share**:

Example:

```
\\server\wallpaper\corp.jpg
```

### NTFS Permissions

| Principal | Permission |
|----------|------------|
| Administrators | Full Control |
| SYSTEM | Full Control |
| Users | Read |

### CMD (Create share)

```cmd
net share wallpaper=D:\Wallpaper /grant:Everyone,READ
```

### PowerShell

```powershell
New-SmbShare -Name "wallpaper" -Path "D:\Wallpaper" -ReadAccess "Everyone"
```

---

# 5. Mandatory Background via Group Policy

## 5.1 GPO Path

```
User Configuration
 └── Administrative Templates
     └── Desktop
         └── Desktop
             └── Desktop Wallpaper
```

---

## 5.2 Configure Mandatory Wallpaper (GUI)

1. Open **Group Policy Management**  
2. Edit or create a GPO  
3. Navigate to the path above  
4. Enable **Desktop Wallpaper**  
5. Set wallpaper path:  
   ```
   \\server\wallpaper\corp.jpg
   ```
6. Set wallpaper style:  
   - Center  
   - Stretch  
   - Fill  
   - Fit  
   - Tile  

---

## 5.3 CMD (Force wallpaper via registry)

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /v Wallpaper /t REG_SZ /d "\\server\wallpaper\corp.jpg" /f
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /v WallpaperStyle /t REG_SZ /d 2 /f
```

WallpaperStyle values:

| Value | Style |
|-------|--------|
| 0 | Center |
| 2 | Stretch |
| 6 | Fit |
| 10 | Fill |

---

## 5.4 PowerShell (Force wallpaper via registry)

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name Wallpaper -Value "\\server\wallpaper\corp.jpg"
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name WallpaperStyle -Value "2"
```

---

# 6. Mandatory Background via Mandatory Profiles

Mandatory profiles can include a **preconfigured wallpaper**.

## 6.1 Steps

1. Log in as template user  
2. Set wallpaper manually  
3. Log off  
4. Convert profile to mandatory:  
   ```
   NTUSER.DAT → NTUSER.MAN
   ```
5. Assign mandatory profile to users  

### CMD (Rename)

```cmd
rename \\server\profiles\mandatory\NTUSER.DAT NTUSER.MAN
```

### PowerShell

```powershell
Rename-Item "\\server\profiles\mandatory\NTUSER.DAT" "NTUSER.MAN"
```

---

# 7. Mandatory Background via Local Policy (Non‑Domain)

## 7.1 GPO Path

```
gpedit.msc
User Configuration → Administrative Templates → Desktop → Desktop Wallpaper
```

---

## 7.2 CMD

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /v Wallpaper /t REG_SZ /d "C:\Corp\corp.jpg" /f
```

---

## 7.3 PowerShell

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name Wallpaper -Value "C:\Corp\corp.jpg"
```

---

# 8. Prevent Users from Changing Wallpaper

## 8.1 GPO Path

```
User Configuration
 └── Administrative Templates
     └── Control Panel
         └── Personalization
             └── Prevent changing desktop background
```

Enable this setting.

---

## 8.2 CMD

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\ActiveDesktop" /v NoChangingWallPaper /t REG_DWORD /d 1 /f
```

---

## 8.3 PowerShell

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\ActiveDesktop" -Name NoChangingWallPaper -Value 1
```

---

# 9. Enforcing Mandatory Background with File System Lockdown

To prevent users from replacing the wallpaper file:

### NTFS Permissions

| Principal | Permission |
|----------|------------|
| Users | Read |
| Administrators | Full Control |
| SYSTEM | Full Control |

### CMD

```cmd
icacls D:\Wallpaper\corp.jpg /grant Users:RX
```

### PowerShell

```powershell
icacls D:\Wallpaper\corp.jpg /grant Users:RX
```

---

# 10. Troubleshooting Mandatory Backgrounds

| Issue | Cause | Fix |
|-------|--------|-----|
| Wallpaper not applied | Wrong path | Use UNC path |
| Wallpaper resets | User has write access | Remove write permissions |
| Wallpaper not loading | Network slow | Use local fallback |
| GPO ignored | Wrong OU | Link GPO correctly |
| Mandatory profile ignored | NTUSER.MAN missing | Rename file |

---

## 10.1 CMD Diagnostics

```cmd
gpresult /r
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System"
```

## 10.2 PowerShell Diagnostics

```powershell
Get-GPResultantSetOfPolicy -ReportType Html -Path C:\gp.html
```

---

# 11. Complete Mandatory Background Reference Table

| Method | Domain | Non‑Domain | Supports Mandatory Profiles | Best Use |
|--------|--------|------------|------------------------------|----------|
| GPO Wallpaper | ✔ | ✔ | ✔ | Corporate environments |
| Registry Enforcement | ✔ | ✔ | ✔ | Kiosks, classrooms |
| Mandatory Profile | ✔ | ✔ | ✔ | Locked‑down systems |
| NTFS Lockdown | ✔ | ✔ | ✔ | Prevent tampering |

---

