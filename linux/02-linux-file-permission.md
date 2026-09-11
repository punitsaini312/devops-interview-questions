# Linux File Permissions & Access Control

## Overview

Linux is a multi-user, multi-tasking operating system where security and resource isolation rely heavily on **File Permissions** and **Access Control**. Every file, directory, and process in Linux is associated with specific ownership and permissions that decide who can read, write, or execute it.

As a DevOps engineer, understanding and managing file permissions is essential for:
- Securing application configurations and sensitive secrets
- Setting up web server roots and application storage directories
- Managing SSH key permissions (`.ssh/config`, `authorized_keys`, private keys)
- Configuring CI/CD pipelines and executable automation scripts
- Hardening container environments and host volume mounts
- Troubleshooting "Permission Denied" errors in deployments

---

# Understanding File Permissions Structure

When you execute `ls -l` in a terminal, Linux displays standard file attributes:

```bash
ls -l /var/www/html/index.html
-rw-r--r-- 1 www-data www-data 2450 May 12 10:30 index.html
```

### Breakdown of Output Fields

| Field | Example | Description |
|-------|---------|-------------|
| File Type & Permissions | `-rw-r--r--` | 10-character string representing file type and 3 permission sets |
| Hard Link Count | `1` | Number of hard links pointing to this file |
| Owner (User) | `www-data` | Account that owns the file |
| Group | `www-data` | Group associated with the file |
| File Size | `2450` | Size in bytes |
| Modification Time | `May 12 10:30` | Date and time last modified |
| File Name | `index.html` | Name of file or directory |

---

# Detailed Permission Bits Breakdown

The 10-character string (`-rw-r--r--`) breaks down into four distinct sections:

```text
 -    r w -    r - -    r - -
 |    |---|    |---|    |---|
 |      |        |        |---> Others (world) permissions
 |      |        |------------> Group permissions
 |      |---------------------> User (owner) permissions
 |----------------------------> File Type Indicator
```

### File Types Indicator (`1st Character`)
- `-` : Regular file
- `d` : Directory
- `l` : Symbolic link (symlink)
- `c` : Character device file (e.g., terminal, serial port)
- `b` : Block device file (e.g., hard drive partition)
- `s` : Local socket
- `p` : Named pipe (FIFO)

---

# Permission Types: Files vs Directories

Permissions act differently depending on whether they apply to a **file** or a **directory**.

| Permission | Symbol | Binary / Numeric | Effect on Files | Effect on Directories |
|------------|--------|------------------|-----------------|-----------------------|
| **Read** | `r` | `4` | View contents (`cat`, `less`) | List directory contents (`ls`) |
| **Write** | `w` | `2` | Modify file contents (`vim`, `echo`) | Create, delete, or rename files within directory |
| **Execute** | `x` | `1` | Execute script or binary (`./script.sh`) | Traverse directory (`cd` into it) |

> ⚠️ **Key Note**: To read or write files inside a directory, a user **MUST** have execute (`x`) permission on that directory to traverse into it.

---

# Numeric (Octal) Notation Reference

Permissions are calculated by adding numerical values for User, Group, and Others:

| Representation | Permission | Value |
|----------------|------------|-------|
| `r` | Read | 4 |
| `w` | Write | 2 |
| `x` | Execute | 1 |
| `-` | None | 0 |

### Common Octal permission modes:

- `7` = `4 + 2 + 1` (`rwx`) — Full permissions
- `6` = `4 + 2 + 0` (`rw-`) — Read & Write
- `5` = `4 + 0 + 1` (`r-x`) — Read & Execute
- `4` = `4 + 0 + 0` (`r--`) — Read Only
- `0` = `0 + 0 + 0` (`---`) — No permissions

### Popular Standard Permission Patterns:
- `755` (`rwxr-xr-x`): Standard for executable scripts & web directories.
- `644` (`rw-r--r--`): Standard for regular files (readable by all, editable by owner).
- `700` (`rwx------`): Private directory accessible only by owner (e.g., `~/.ssh`).
- `600` (`rw-------`): Private file accessible only by owner (e.g., `~/.ssh/id_rsa`).
- `777` (`rwxrwxrwx`): Open to everyone (**Security risk! Avoid in production**).

---

# Changing Ownership & Permissions Commands

## `chmod` (Change Mode)

Used to modify file or directory access permissions.

### 1. Symbolic Mode Syntax

```bash
chmod [who][operation][permissions] file
```

- **Who**: `u` (user/owner), `g` (group), `o` (others), `a` (all)
- **Operation**: `+` (add), `-` (remove), `=` (set exact)
- **Permissions**: `r`, `w`, `x`

Examples:

```bash
# Grant execution rights to script owner
chmod u+x script.sh

# Remove write access from group and others
chmod go-w application.conf

# Set exact permissions: Owner full access, Group read/exec, Others none
chmod u=rwx,g=rx,o= config.env

# Grant execute permission to everyone
chmod +x deploy.sh
```

### 2. Numeric / Octal Mode Syntax

```bash
chmod NNN file
```

Examples:

```bash
# Standard secure permission for private SSH key
chmod 600 ~/.ssh/id_rsa

# Standard permission for executable script
chmod 755 /usr/local/bin/deploy.sh

# Recursive change across entire directory tree
chmod -R 755 /var/www/html
```

---

## `chown` (Change Owner & Group)

Modifies user ownership and primary group assignment.

```bash
# Change owner only
sudo chown devops /var/log/app.log

# Change owner and group simultaneously
sudo chown devops:developers /var/www/html

# Change group only (using colon)
sudo chown :developers /var/www/html

# Recursive ownership modification for nested directories
sudo chown -R www-data:www-data /var/www/app
```

---

## `chgrp` (Change Group)

Used specifically to change group ownership.

```bash
# Change group ownership of a file
chgrp developers project.tar.gz

# Change group ownership recursively
chgrp -R developers /opt/shared_data
```

---

# Special Permissions (SUID, SGID, Sticky Bit)

Beyond basic `rwx`, Linux provides 3 special bits for elevated binary execution and group collaboration.

| Special Bit | Octal Value | Symbol | Target | Effect |
|-------------|-------------|--------|--------|--------|
| **SUID** (Set User ID) | `4000` | `u+s` (`rws------`) | Executable Files | File runs with the privileges of the file **owner**, not the user running it. |
| **SGID** (Set Group ID) | `2000` | `g+s` (`rwxr-s---`) | Directories & Files | Files created inside directory inherit directory's group owner, not creator's primary group. |
| **Sticky Bit** | `1000` | `o+t` (`rwxrwxrwt`) | Directories | Users can only delete or rename their own files, even if directory is writable by all (e.g., `/tmp`). |

### SUID Examples & Commands
The `passwd` binary uses SUID so standard users can temporarily elevate to modify `/etc/shadow`.

```bash
# Set SUID on binary
chmod u+s /usr/local/bin/custom-tool
chmod 4755 /usr/local/bin/custom-tool

# Remove SUID
chmod u-s /usr/local/bin/custom-tool
```

### SGID Examples & Commands
Crucial for shared group workspace directories.

```bash
# Set SGID on shared group folder
chmod g+s /var/shared/devops
chmod 2775 /var/shared/devops

# Remove SGID
chmod g-s /var/shared/devops
```

### Sticky Bit Examples & Commands
Used on `/tmp` to prevent users from deleting each other's temporary files.

```bash
# Set Sticky Bit on shared directory
chmod +t /tmp/shared_uploads
chmod 1777 /tmp/shared_uploads

# Remove Sticky Bit
chmod -t /tmp/shared_uploads
```

---

# Default Permissions & `umask`

When a new file or directory is created, default permissions are determined by subtracting the **`umask`** (user mask) value from base creation limits.

- **Base default file permission**: `666` (`rw-rw-rw-`) — Files are not created executable by default.
- **Base default directory permission**: `777` (`rwxrwxrwx`).

### Formula:
```text
Default File Permission      = 666 - umask
Default Directory Permission = 777 - umask
```

### Standard `umask` Examples:

| umask Value | Calculated File Permission | Calculated Directory Permission | Purpose |
|-------------|----------------------------|---------------------------------|---------|
| `0022` (default) | `644` (`rw-r--r--`) | `755` (`rwxr-xr-x`) | Standard user/system default |
| `0027` | `640` (`rw-r-----`) | `750` (`rwxr-x---`) | Restrictive (no public access) |
| `0077` | `600` (`rw-------`) | `700` (`rwx------`) | Highly secure private access |

### Checking & Setting `umask`:

```bash
# Display current umask in octal
umask

# Display current umask in symbolic form
umask -S

# Set temporary umask in current shell session
umask 0027
```

> **Permanent Configuration**: Set `umask 0027` in `/etc/profile`, `~/.bashrc`, or `/etc/bashrc`.

---

# Advanced Access Control Lists (ACL)

Standard POSIX permissions only allow specifying rules for **one owner**, **one group**, and **others**.
**ACLs** allow granting specific permissions to multiple specific users or groups.

### Checking ACLs
```bash
getfacl filename
```

Output:
```text
# file: filename
# owner: devops
# group: developers
user::rw-
user:alice:r--
group::r--
mask::r--
other::r--
```

### Setting ACLs (`setfacl`)

```bash
# Grant user 'alice' read-write access to a specific file
setfacl -m u:alice:rw application.conf

# Grant group 'auditors' read access to log directory
setfacl -m g:auditors:rx /var/log/audit

# Grant default permissions for future new files in directory (Default ACL)
setfacl -d -m g:developers:rwx /var/www/shared

# Remove specific user ACL permission
setfacl -x u:alice application.conf

# Remove all extended ACL rules from file
setfacl -b application.conf
```

---

# Useful Permission Commands Quick Reference

```bash
# View detailed permissions and hidden files
ls -la

# Find all files with 777 permissions (Security Audit)
find /var/www -type f -perm 0777

# Find all files with SUID bit set
find / -perm -4000 -type f 2>/dev/null

# Find all files owned by specific user
find /data -user devops

# Recursively grant directory traversal (x) without making files executable
chmod -R a+X /var/www/html

# Check ACL attributes
getfacl /etc/shadow
```

---

# Production Best Practices

- **Principle of Least Privilege**: Never grant `777` permissions to fix issues. Diagnose whether `user`, `group`, or `path traversal (x)` is lacking.
- **Secure SSH Keys**: Keep private key (`id_rsa`, `*.pem`) permissions strictly at `600` or `400`. SSH will reject keys with broad permissions.
- **Shared Team Directories**: Combine `SGID` (`2770`) with a shared group so team members can automatically access files created by peers.
- **Restrict Web Shell / File Uploads**: Web server upload directories (e.g., `/var/www/html/uploads`) should never have execution (`x`) rights.
- **Use ACLs for Multi-Tenant Projects**: When standard owner/group structures are insufficient, use ACLs rather than granting global access.

---

# Interview Questions

### 1. What do the numbers `7`, `5`, and `5` mean in `chmod 755`?
**Answer**:
- `7` (`4+2+1`): Read, write, execute permissions for the **Owner**.
- `5` (`4+0+1`): Read and execute permissions for the **Group**.
- `5` (`4+0+1`): Read and execute permissions for **Others**.

---

### 2. Why does SSH fail with `WARNING: UNPROTECTED PRIVATE KEY FILE!`?
**Answer**:
SSH enforces strict permissions on private keys. If a private key file is readable/writable by group or others (e.g., `644`), SSH blocks connections for security. Fix it using:
```bash
chmod 600 ~/.ssh/id_rsa
```

---

### 3. What is the difference between `chmod -R 755` and `chmod -R a+X`?
**Answer**:
`chmod -R 755` applies execution permissions to **both files and directories**. `chmod -R a+X` (capital `X`) applies execution permissions **only to directories** (and files that already have execution rights set), avoiding accidentally making regular data files executable.

---

### 4. What is the Sticky Bit and where is it commonly used?
**Answer**:
The Sticky Bit (octal `1000` / `o+t`) restricts file deletion in writeable directories. Users can only delete or rename files they own. It is commonly used on world-writable directories like `/tmp` or `/var/tmp`.

---

### 5. How does SGID work on a directory?
**Answer**:
When SGID (`g+s` / octal `2000`) is enabled on a directory, any new file or sub-directory created inside it automatically inherits the group ownership of the parent directory rather than the primary group of the user who created it.

---

### 6. What is `umask` and what will default permissions be for `umask 0022`?
**Answer**:
`umask` defines default permission masking during file creation.
- **Files**: `666 - 022 = 644` (`rw-r--r--`)
- **Directories**: `777 - 022 = 755` (`rwxr-xr-x`)

---

### 7. What permission is required to `cd` into a directory?
**Answer**:
Execute (`x`) permission on the directory is required to enter or traverse into it.

---

### 8. What is the difference between standard Linux permissions and ACLs?
**Answer**:
Standard POSIX permissions permit setting rights for only 1 user (owner), 1 group, and others. ACLs (Access Control Lists) allow assigning specific permissions to multiple distinct users and groups on a single file or directory.

---

### 9. What does `chmod 4755 /usr/bin/tool` do?
**Answer**:
It sets octal `4` in the special bit position, which enables **SUID** (`Set User ID`). The binary will run with the privileges of the file owner (usually root) regardless of who executes it.

---

### 10. How do you grant read access to a file for user `johndoe` without changing ownership or standard group?
**Answer**:
Use Access Control Lists (ACL):
```bash
setfacl -m u:johndoe:r filename
```

---

# Scenario-Based Questions

### Scenario 1
A developer can read directory contents (`ls /data/project`), but receives `Permission Denied` when trying to `cd /data/project`. Why?

**Answer**:
The directory has **Read (`r`)** permission set, but lacks **Execute (`x`)** permission for the user or group. Grant execute permission:
```bash
chmod +x /data/project
```

---

### Scenario 2
Members of the `devs` group create files in `/opt/shared`, but other members cannot edit or clean up those files because they are created with individual user group ownership. How do you fix this permanently?

**Answer**:
1. Set group ownership of the directory to `devs`:
   ```bash
   sudo chown :devs /opt/shared
   ```
2. Enable SGID on the folder so newly created files inherit the `devs` group:
   ```bash
   sudo chmod g+s /opt/shared
   ```
3. Set appropriate directory umask or default ACL:
   ```bash
   sudo setfacl -d -m g:devs:rwx /opt/shared
   ```

---

### Scenario 3
An application service running as user `appuser` cannot read `/var/log/nginx/access.log`, which is owned by `www-data:adm`. You are not allowed to change the owner or primary group. How do you solve it?

**Answer**:
Use ACL to explicitly grant read access to `appuser`:
```bash
sudo setfacl -m u:appuser:r /var/log/nginx/access.log
```

---

### Scenario 4
A script executed by a deployment tool fails with `Permission denied: ./deploy.sh` even though ownership is correct.

**Answer**:
The script lacks execute (`x`) permission bit. Add execute permission:
```bash
chmod +x deploy.sh
```

---

### Scenario 5
How do you find and audit all publicly writable (`777` or `666`) files in `/var/www`?

**Answer**:
Run `find` with perm flag:
```bash
find /var/www -type f \( -perm -0002 -o -perm -0020 \)
# Or exact 777:
find /var/www -perm 0777
```

---

# One-Minute Revision

✓ `r` = 4 (Read), `w` = 2 (Write), `x` = 1 (Execute).

✓ File Permissions Order: **User (u)** → **Group (g)** → **Others (o)**.

✓ Directory Execute (`x`) bit is required to `cd` into a directory.

✓ `chmod` modifies access permissions; `chown` modifies owner/group.

✓ `600` for SSH private keys; `755` for scripts & executable directories; `644` for normal files.

✓ **SUID (`4000`)**: Runs binary as file owner.

✓ **SGID (`2000`)**: New files in directory inherit directory group ownership.

✓ **Sticky Bit (`1000`)**: Prevents users from deleting other users' files in shared directory (e.g. `/tmp`).

✓ **Default Permissions Calculation**: `666 - umask` for files, `777 - umask` for directories.

✓ Use **ACL (`getfacl` / `setfacl`)** when standard owner/group model is too restrictive.



Presets you'll type constantly
Mode	Use case
755	Scripts and executables anyone can run
644	Configs and text files anyone can read
700	Directory only the owner should enter
600	Secrets: SSH keys, API tokens, .env files
