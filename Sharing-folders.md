

---

# Windows Server Folder Sharing & Permission Management  
Complete Technical Documentation (Windows 2000 → Windows Server 2022)

---

## 1. Overview

Windows Server uses two permission layers:

- **Share Permissions** — applied when accessing a folder via SMB (`\\server\share`).  
- **NTFS Permissions** — applied at the file system level, both locally and remotely.

**Effective access = most restrictive combination of Share + NTFS permissions.**

This document covers:

- Creating SMB shares  
- Share permissions  
- NTFS permissions (standard + advanced)  
- Inheritance, ownership, effective permissions  
- CMD + PowerShell commands for all operations  

---

# 2. Creating SMB Shares

## 2.1 GUI Method (All Versions)

### Windows 2000 / XP / Server 2003

1. Right‑click folder → **Sharing**  
2. Select **Share this folder**  
3. Assign share name  
4. Configure permissions  
5. Apply

### Server 2008 → 2022

1. Right‑click folder → **Properties**  
2. **Sharing** tab → **Advanced Sharing**  
3. Enable **Share this folder**  
4. Assign share name  
5. Configure permissions  
6. Apply

---

# 3. Creating SMB Shares (CMD)

### Windows 2000 / XP / Server 2003

```cmd
net share MyShare=C:\Data /remark:"Shared Folder" /grant:Everyone,READ
```

### Server 2008 → 2022

```cmd
net share MyShare=C:\Data /grant:Everyone,READ
```

---

# 4. Creating SMB Shares (PowerShell)

> PowerShell share creation is supported on Server 2012 → 2022.

```powershell
New-SmbShare -Name "MyShare" -Path "C:\Data" -FullAccess "Everyone"
```

---

# 5. Share Permissions

Share permissions apply only to SMB access.

### 5.1 Permission Levels

| Permission | Capabilities |
|-----------|--------------|
| **Read** | View folder contents, open files |
| **Change** | Read, create, modify, delete files/folders |
| **Full Control** | All Change capabilities + modify share permissions |

---

# 6. NTFS Permissions (Standard)

NTFS permissions apply to both local and remote access.

### 6.1 Standard NTFS Permission Levels

| Permission | Includes | Description |
|-----------|----------|-------------|
| **Full Control** | All permissions | Modify, delete, change permissions, take ownership |
| **Modify** | Read, Write, Execute, Delete | Standard read/write/delete access |
| **Read & Execute** | Read, Execute | View and run files |
| **List Folder Contents** | List | View folder structure |
| **Read** | Read | View contents and attributes |
| **Write** | Write | Create files/folders, write data |

---

# 7. NTFS Advanced (Special) Permissions

This section lists **every advanced NTFS permission**, grouped by category.

---

## 7.1 Folder‑Level Special Permissions

| Permission | Description |
|-----------|-------------|
| Traverse Folder / Execute File | Enter folders or run executables |
| List Folder / Read Data | View folder contents or read file data |
| Read Attributes | View basic attributes |
| Read Extended Attributes | View extended metadata |
| Create Files / Write Data | Create files or write data |
| Create Folders / Append Data | Create subfolders or append data |
| Write Attributes | Modify basic attributes |
| Write Extended Attributes | Modify extended metadata |
| Delete Subfolders and Files | Delete items inside folder |
| Delete | Delete the folder |
| Read Permissions | View ACL |
| Change Permissions | Modify ACL |
| Take Ownership | Take ownership |

---

## 7.2 File‑Level Special Permissions

| Permission | Description |
|-----------|-------------|
| Read Data | Read file contents |
| Write Data | Modify file contents |
| Append Data | Add data to end of file |
| Read Attributes | View basic attributes |
| Read Extended Attributes | View extended metadata |
| Write Attributes | Modify basic attributes |
| Write Extended Attributes | Modify extended metadata |
| Execute File | Run executable files |
| Delete | Delete the file |
| Read Permissions | View ACL |
| Change Permissions | Modify ACL |
| Take Ownership | Take ownership |

---

# 8. Permission Bundles (How Windows Groups Special Permissions)

### 8.1 Full Control

Includes:

- All special permissions  
- Change permissions  
- Take ownership  

### 8.2 Modify

Includes:

- Read & Execute  
- Write  
- Delete  

### 8.3 Read & Execute

Includes:

- Read  
- Execute File / Traverse Folder  

### 8.4 Read

Includes:

- Read Data  
- Read Attributes  
- Read Extended Attributes  
- Read Permissions  

### 8.5 Write

Includes:

- Write Data  
- Append Data  
- Write Attributes  
- Write Extended Attributes  

---

# 9. Inheritance

### 9.1 How It Works

- Parent folder permissions propagate to child objects  
- Inheritance can be enabled or disabled  
- Breaking inheritance allows custom ACLs

### 9.2 Inheritance Flags

| Flag | Meaning |
|------|---------|
| This folder only | Applies only to the folder |
| This folder, subfolders, and files | Applies to all children |
| Subfolders and files only | Not applied to the folder |
| Subfolders only | Applies only to subfolders |
| Files only | Applies only to files |

---

# 10. Ownership

### 10.1 Rules

- Owners can always modify permissions  
- Ownership can be transferred by users with **Take Ownership** permission  
- Typical owners: **Administrators**, **SYSTEM**

---

# 11. Effective Permissions

Effective access is determined by:

1. All NTFS Allow entries  
2. All NTFS Deny entries (override Allow)  
3. Share permissions  
4. Inherited permissions  
5. Group membership  

Use **Advanced → Effective Access** to evaluate.

---

# 12. Permission Interaction Summary

| Share Permission | NTFS Permission | Effective Result |
|------------------|-----------------|------------------|
| Full Control | Read | **Read** |
| Read | Modify | **Read** |
| Change | Full Control | **Change‑like** (share caps access) |

---

# 13. Managing NTFS Permissions (CMD)

### Windows 2000 → Server 2003

```cmd
cacls C:\Data /E /G User:W
```

### Server 2008 → 2022

```cmd
icacls C:\Data /grant User:(OI)(CI)M
```

---

# 14. Managing NTFS Permissions (PowerShell)

> PowerShell ACL management works on Server 2008 → 2022.

```powershell
$acl = Get-Acl "C:\Data"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("User","Modify","ContainerInherit,ObjectInherit","None","Allow")
$acl.AddAccessRule($rule)
Set-Acl "C:\Data" $acl
```

---

# 15. Creating Hidden Shares

### CMD

```cmd
net share Finance$=D:\Finance /grant:FinanceGroup,FULL
```

### PowerShell

```powershell
New-SmbShare -Name "Finance$" -Path "D:\Finance" -FullAccess "FinanceGroup"
```

---

# 16. Example Folder Structure & Permissions

```
D:\Data
 ├── HR
 ├── Finance
 ├── IT
 └── Public
```

| Folder | NTFS Permissions | Share Permissions |
|--------|------------------|------------------|
| HR | HR_Modify, HR_Read | Domain Users – Full Control |
| Finance | Finance_Modify, Finance_Read | Domain Users – Full Control |
| IT | IT_Admins – Full Control | Domain Users – Full Control |
| Public | Everyone – Read | Domain Users – Full Control |

---

