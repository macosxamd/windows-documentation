
---

# Windows Server NTFS & Share Permissions  
Complete Technical Reference (Including All Advanced Permissions)

---

## 1. Permission Systems Overview

Windows Server uses two permission layers:

- **Share Permissions** — applied only when accessing via SMB (`\\server\share`).  
- **NTFS Permissions** — applied at the file system level, both locally and remotely.

**Effective access = most restrictive combination of Share + NTFS permissions.**

---

# 2. Share Permissions (SMB)

Share permissions are simple and apply only to network access.

### 2.1 Share Permission Levels

| Permission | Description |
|-----------|-------------|
| **Read** | View folder contents, open files. |
| **Change** | Read, create, modify, delete files/folders. |
| **Full Control** | All Change capabilities + modify share permissions. |

### 2.2 Notes

- Share permissions do **not** apply to local access.  
- Share permissions are coarse; NTFS provides fine‑grained control.  
- Hidden shares use `$` suffix (e.g., `Finance$`).

---

# 3. NTFS Permissions (Standard)

NTFS permissions apply to both local and remote access.

### 3.1 Standard NTFS Permission Levels

| Permission | Includes | Description |
|-----------|----------|-------------|
| **Full Control** | All permissions | Modify, delete, change permissions, take ownership. |
| **Modify** | Read, Write, Execute, Delete | Standard read/write/delete access. |
| **Read & Execute** | Read, Execute | View and run files. |
| **List Folder Contents** | List | View folder structure (folder only). |
| **Read** | Read | View contents and attributes. |
| **Write** | Write | Create files/folders, write data. |

---

# 4. NTFS Advanced (Special) Permissions  
**This section lists *every* advanced NTFS permission**, grouped by functional category.

---

## 4.1 Folder‑Level Special Permissions

These apply to folders only.

| Permission | Description |
|-----------|-------------|
| **Traverse Folder / Execute File** | Allows entering folders even without List permission; allows executing files. |
| **List Folder / Read Data** | View folder contents; read file data. |
| **Read Attributes** | View basic attributes (read‑only, hidden, etc.). |
| **Read Extended Attributes** | View extended metadata (application‑defined). |
| **Create Files / Write Data** | Create new files in the folder; write data to files. |
| **Create Folders / Append Data** | Create subfolders; append data to existing files. |
| **Write Attributes** | Modify basic attributes. |
| **Write Extended Attributes** | Modify extended metadata. |
| **Delete Subfolders and Files** | Delete items inside the folder, even without Delete permission on the item. |
| **Read Permissions** | View the ACL (Access Control List). |
| **Change Permissions** | Modify the ACL. |
| **Take Ownership** | Take ownership of the folder or files. |

---

## 4.2 File‑Level Special Permissions

These apply to files only.

| Permission | Description |
|-----------|-------------|
| **Read Data** | Read file contents. |
| **Write Data** | Modify file contents. |
| **Append Data** | Add data to the end of the file. |
| **Read Attributes** | View basic attributes. |
| **Read Extended Attributes** | View extended metadata. |
| **Write Attributes** | Modify basic attributes. |
| **Write Extended Attributes** | Modify extended metadata. |
| **Execute File** | Run executable files. |
| **Delete** | Delete the file. |
| **Read Permissions** | View ACL. |
| **Change Permissions** | Modify ACL. |
| **Take Ownership** | Take ownership of the file. |

---

# 5. Permission Groups (How Windows Bundles Special Permissions)

Windows combines special permissions into the standard permission levels.

### 5.1 Full Control

Includes:

- All special permissions  
- Change permissions  
- Take ownership  

### 5.2 Modify

Includes:

- Read & Execute  
- Write  
- Delete  

### 5.3 Read & Execute

Includes:

- Read  
- Execute File / Traverse Folder  

### 5.4 Read

Includes:

- Read Data  
- Read Attributes  
- Read Extended Attributes  
- Read Permissions  

### 5.5 Write

Includes:

- Write Data  
- Append Data  
- Write Attributes  
- Write Extended Attributes  

---

# 6. Inheritance

### 6.1 How It Works

- Permissions on a parent folder propagate to child objects.  
- Inheritance can be **enabled** or **disabled** per object.  
- Breaking inheritance allows custom ACLs.

### 6.2 Inheritance Flags

| Flag | Meaning |
|------|---------|
| **This folder only** | Applies only to the folder. |
| **This folder, subfolders, and files** | Applies to all children. |
| **Subfolders and files only** | Does not apply to the folder itself. |
| **Subfolders only** | Applies only to subfolders. |
| **Files only** | Applies only to files. |

---

# 7. Ownership

### 7.1 Rules

- Owners can always modify permissions, even if denied.  
- Ownership can be transferred by users with **Take Ownership** permission.  
- Typical owners: **Administrators**, **SYSTEM**.

---

# 8. Effective Permissions

Effective access is determined by:

1. **All NTFS Allow entries**  
2. **All NTFS Deny entries** (override Allow)  
3. **Share permissions**  
4. **Inherited permissions**  
5. **Group membership**  

Use **Advanced → Effective Access** to evaluate.

---

# 9. Permission Interaction Summary

| Share Permission | NTFS Permission | Effective Result |
|------------------|-----------------|------------------|
| Full Control | Read | **Read** |
| Read | Modify | **Read** |
| Change | Full Control | **Change‑like** (share caps access) |

---

# 10. Complete Special Permission Reference Table

This table includes **every special permission** for both files and folders.

| Special Permission | Folder | File | Description |
|--------------------|--------|------|-------------|
| Traverse Folder / Execute File | ✔ | ✔ | Enter folders or run executables. |
| List Folder / Read Data | ✔ | ✔ | View folder contents or read file data. |
| Read Attributes | ✔ | ✔ | View basic attributes. |
| Read Extended Attributes | ✔ | ✔ | View extended metadata. |
| Create Files / Write Data | ✔ | ✔ | Create files or write data. |
| Create Folders / Append Data | ✔ | ✔ | Create folders or append data. |
| Write Attributes | ✔ | ✔ | Modify basic attributes. |
| Write Extended Attributes | ✔ | ✔ | Modify extended metadata. |
| Delete Subfolders and Files | ✔ | — | Delete items inside folder. |
| Delete | ✔ | ✔ | Delete the object. |
| Read Permissions | ✔ | ✔ | View ACL. |
| Change Permissions | ✔ | ✔ | Modify ACL. |
| Take Ownership | ✔ | ✔ | Take ownership. |

---

