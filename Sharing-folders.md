
---

# Windows Server File Sharing & Permission Management  
Comprehensive Technical Documentation

---

## 1. Overview

Windows Server uses two permission systems to control access to shared data:

- **Share Permissions** — applied when accessing a folder over SMB (`\\server\share`).  
- **NTFS Permissions** — applied at the file system level, both locally and remotely.

**Effective access** is determined by the most restrictive combination of both.

---

## 2. Creating and Managing SMB Shares

### 2.1 Creating a Share (File Explorer)

1. Right‑click folder → **Properties**  
2. Open **Sharing** tab  
3. Select **Advanced Sharing**  
4. Enable **Share this folder**  
5. Assign share name  
6. Configure **Permissions**

### 2.2 Creating a Share (Server Manager)

1. Open **File and Storage Services** → **Shares**  
2. Select **Tasks** → **New Share**  
3. Choose share profile (SMB Quick, SMB Advanced)  
4. Configure share settings, permissions, caching, ABE

---

## 3. Share Permissions

Share permissions apply only to SMB access.

### 3.1 Permission Levels

| Permission | Capabilities |
|-----------|--------------|
| **Read** | View folder contents, open files |
| **Change** | Read, create, modify, delete files/folders |
| **Full Control** | Change permissions, take ownership, all Change capabilities |

### 3.2 Best Practices

- Use broad share permissions (e.g., **Everyone: Full Control**) and enforce security via NTFS  
- Use AD groups instead of individual users  
- Use hidden shares (`ShareName$`) when needed

---

## 4. NTFS Permissions

NTFS permissions apply to both local and remote access.

### 4.1 Standard NTFS Permission Levels

| Permission | Capabilities |
|-----------|--------------|
| **Full Control** | Modify, delete, change permissions, take ownership |
| **Modify** | Read, write, execute, delete |
| **Read & Execute** | View and run files |
| **List Folder Contents** | View folder structure |
| **Read** | View contents and attributes |
| **Write** | Create files/folders, write data |

---

## 5. NTFS Advanced (Special) Permissions

Special permissions provide granular control.

### 5.1 Special Permission Types

- Traverse folder / execute file  
- List folder / read data  
- Read attributes  
- Read extended attributes  
- Create files / write data  
- Create folders / append data  
- Write attributes  
- Write extended attributes  
- Delete  
- Delete subfolders and files  
- Read permissions  
- Change permissions  
- Take ownership

### 5.2 Permission Inheritance

- Parent folder permissions propagate to child objects  
- Inheritance can be disabled per folder/file  
- Use inheritance for consistent structure; break only when necessary

### 5.3 Ownership

- Owners can always modify permissions  
- Typically assigned to **Administrators** or **SYSTEM**  
- Change via: *Security → Advanced → Owner*

---

## 6. Effective Permissions

Effective access = **minimum** of:

- Share permissions  
- NTFS permissions  
- Deny entries override Allow

Use **Security → Advanced → Effective Access** to evaluate user/group access.

---

## 7. Additional SMB Share Capabilities

### 7.1 Access‑Based Enumeration (ABE)

- Hides folders/files users cannot access  
- Enabled via SMB Advanced Share settings

### 7.2 Offline Files / Caching

- Controls client‑side caching behavior  
- Options: None, Manual, Automatic

### 7.3 Administrative Shares

- Default hidden shares: `C$`, `D$`, `ADMIN$`  
- Used for remote administration

### 7.4 DFS Namespace Integration

- Allows publishing shares into a unified namespace  
- Supports load balancing and fault tolerance

---

## 8. Recommended Permission Design Model

### 8.1 Group‑Based Access Control

Create AD groups such as:

- `Dept_X_Read`  
- `Dept_X_Modify`  
- `Dept_X_FullControl`

Assign NTFS permissions to groups, not users.

### 8.2 Share Permission Strategy

Two common models:

#### Model A: Share Wide Open, NTFS Controls Access

- Share: **Everyone – Full Control**  
- NTFS: granular per folder

#### Model B: Share Restrictive

- Share: **Everyone – Read**  
- NTFS: grant write only to specific groups

### 8.3 Avoid Deny Unless Necessary

- Deny overrides all Allow entries  
- Use only for explicit exclusion cases

---

## 9. Permission Reference Table

| Permission Type | Applies To | Typical Use |
|-----------------|------------|-------------|
| Share: Read | SMB only | Basic access to shared data |
| Share: Change | SMB only | Allow modification over network |
| Share: Full Control | SMB only | Admin-level share control |
| NTFS: Read | Local + SMB | View-only access |
| NTFS: Modify | Local + SMB | Standard user write access |
| NTFS: Full Control | Local + SMB | Administrative access |

---

## 10. Example Folder Structure & Permissions

### 10.1 Example Structure

```
D:\Data
 ├── HR
 ├── Finance
 ├── IT
 └── Public
```

### 10.2 Example Permissions

| Folder | NTFS Permissions | Share Permissions |
|--------|------------------|------------------|
| HR | HR_Modify, HR_Read | Domain Users – Full Control |
| Finance | Finance_Modify, Finance_Read | Domain Users – Full Control |
| IT | IT_Admins – Full Control | Domain Users – Full Control |
| Public | Everyone – Read | Domain Users – Full Control |

---

