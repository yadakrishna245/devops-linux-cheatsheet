<div align="center">

# 🐧 Linux L2/L3 Interview Questions & Answers

**200 Real-World Interview Questions for Senior Linux Administrators**

*Prepared by Krishna Yada | Senior System Administrator | 8+ Years | 4000+ Servers*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Krishna_Yada-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/krishna-yada-8a8441239)
[![GitHub](https://img.shields.io/badge/GitHub-yadakrishna245-black?style=flat-square&logo=github)](https://github.com/yadakrishna245)

</div>

---

## 📋 Table of Contents

| # | Section | Questions |
|---|---------|-----------|
| 1 | [User & Group Administration](#1-user--group-administration) | 1–25 |
| 2 | [File System & Permissions](#2-file-system--permissions) | 26–50 |
| 3 | [Storage & LVM](#3-storage--lvm) | 51–75 |
| 4 | [Networking](#4-networking) | 76–100 |
| 5 | [Services & Boot Process](#5-services--boot-process) | 101–125 |
| 6 | [Package Management](#6-package-management) | 126–145 |
| 7 | [Security & Hardening](#7-security--hardening) | 146–170 |
| 8 | [Performance & Troubleshooting](#8-performance--troubleshooting) | 171–195 |
| 9 | [Scenario-Based Questions](#9-scenario-based-questions) | 196–200 |

---

## 1. User & Group Administration

**Q1. What are the different types of users in Linux?**

There are 5 types:
| Type | UID Range (RHEL 7+) | Example |
|------|---------------------|---------|
| Super User (root) | 0 | root |
| Normal User | 1000–60000 | krishna, admin |
| Static System User | 1–200 | bin, daemon |
| Dynamic System User | 201–999 | apache, nginx |
| Network User (LDAP/AD) | Same as normal | LDAP users |

---

**Q2. What are the important files related to user management?**

| File | Purpose |
|------|---------|
| `/etc/passwd` | User info (name, UID, GID, home, shell) |
| `/etc/shadow` | Encrypted passwords & aging policies |
| `/etc/group` | Group info |
| `/etc/gshadow` | Group passwords |
| `/etc/login.defs` | Default login settings |
| `/etc/skel` | Template files copied to new user's home |
| `/etc/default/useradd` | Default values for useradd |

---

**Q3. Explain the fields in /etc/passwd file.**

```
username:x:UID:GID:comment:home_directory:login_shell
```
- `x` — indicates password is stored in `/etc/shadow`
- UID 0 = root, 1-999 = system, 1000+ = normal users

---

**Q4. Explain the fields in /etc/shadow file.**

```
username:password:last_changed:min:max:warn:inactive:expire:reserved
```
- `password` — encrypted hash (`$6$` = SHA-512, `$5$` = SHA-256)
- `!!` or `!` = locked account, `*` = no login

---

**Q5. How do you create a user with specific UID, GID, shell, and home directory?**

```bash
useradd -u 1500 -g devops -G wheel,docker -c "Krishna Yada" -d /home/krishna -s /bin/bash krishna
```

---

**Q6. How to lock and unlock a user account?**

```bash
# Lock
usermod -L username
passwd -l username

# Unlock
usermod -U username
passwd -u username

# Verify
passwd -S username    # Shows "L" for locked
```

---

**Q7. What is the difference between `usermod -L` and `passwd -l`?**

Both lock the account by adding `!` before the password hash in `/etc/shadow`. However:
- `passwd -l` only locks the password (SSH key login still works)
- `usermod -L` locks the password
- To fully disable: `usermod -L -e 1 username` (lock + expire)

---

**Q8. How do you set password aging policies?**

```bash
# Using chage
chage -M 90 -m 7 -W 14 -I 5 username
# -M = max days, -m = min days, -W = warning, -I = inactive

# View policies
chage -l username

# Force password change on next login
chage -d 0 username

# System-wide defaults in /etc/login.defs
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
```

---

**Q9. How to configure sudo access for a user?**

```bash
# Method 1: Add to wheel group
usermod -aG wheel username

# Method 2: /etc/sudoers.d/ (preferred)
echo "krishna ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/krishna
chmod 440 /etc/sudoers.d/krishna

# Method 3: Command alias in sudoers
visudo
Cmnd_Alias SERVICES = /usr/bin/systemctl restart *, /usr/bin/systemctl status *
krishna ALL=(ALL) SERVICES
```

---

**Q10. Where are sudo command logs stored?**

```bash
/var/log/secure       # RHEL/CentOS
/var/log/auth.log     # Ubuntu/Debian
journalctl _COMM=sudo # Systemd-based systems
```

---

**Q11. How to check integrity of password and group files?**

```bash
pwck                  # Check /etc/passwd and /etc/shadow
grpck                 # Check /etc/group and /etc/gshadow
pwck -r               # Read-only mode
```

---

**Q12. What is the difference between `useradd` and `adduser`?**

| Feature | `useradd` | `adduser` |
|---------|-----------|-----------|
| RHEL/CentOS | Low-level binary | Same as useradd (symlink) |
| Debian/Ubuntu | Low-level binary | Interactive Perl script |
| Creates home | Only with `-m` | Automatically |
| Sets password | No | Prompts for it |

---

**Q13. How to create multiple users at once?**

```bash
# Using newusers (reads from file)
cat users.txt
user1:password1:2001:2001:User One:/home/user1:/bin/bash
user2:password2:2002:2002:User Two:/home/user2:/bin/bash

newusers users.txt

# Using a loop
for user in krishna raju gopal; do
  useradd -m -s /bin/bash $user
  echo "$user:TempPass@123" | chpasswd
  chage -d 0 $user
done
```

---

**Q14. How to find all users with UID 0 (root-equivalent)?**

```bash
awk -F: '$3==0 {print $1}' /etc/passwd
```

---

**Q15. How to disable direct root login?**

```bash
# SSH
sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd

# Console (TTY)
echo > /etc/securetty    # Empty the file

# PAM
# Add to /etc/pam.d/login:
auth required pam_securetty.so
```

---

**Q16. What is the difference between primary group and secondary group?**

- **Primary group** — defined in `/etc/passwd` (4th field), applied to new files by default
- **Secondary groups** — defined in `/etc/group`, provide additional permissions
- A user can have **1 primary** group and up to **65536 secondary** groups

---

**Q17. How to change a user's primary group?**

```bash
usermod -g newgroup username
# Verify
id username
```

---

**Q18. Explain the `/etc/skel` directory.**

- Template directory for new users
- Files here (`.bashrc`, `.bash_profile`, `.bash_logout`) are copied to each new user's home directory
- Customize to set default environment for all new users

---

**Q19. How to check who is currently logged in and their activity?**

```bash
who                   # Currently logged in users
w                     # Detailed: who + what they're doing
last                  # Login history
lastb                 # Failed login attempts
lastlog               # Last login for all users
```

---

**Q20. What happens when you delete a user with `userdel` vs `userdel -r`?**

- `userdel username` — removes user from `/etc/passwd`, `/etc/shadow`, `/etc/group` but keeps home directory and mail spool
- `userdel -r username` — removes user AND deletes home directory + mail spool
- Files owned by deleted user in other locations remain (find with `find / -nouser`)

---

**Q21. How to find files owned by a specific user or group?**

```bash
find / -user krishna -type f
find / -group devops -type f
find / -nouser                # Orphaned files (user deleted)
find / -nogroup               # Orphaned files (group deleted)
```

---

**Q22. How to restrict a user from logging into the system?**

```bash
# Method 1: Change shell
usermod -s /sbin/nologin username

# Method 2: Lock account
usermod -L username

# Method 3: Expire account
usermod -e 2024-01-01 username

# Method 4: PAM restriction (/etc/security/access.conf)
- : username : ALL
```

---

**Q23. What is the `umask` and how does it work?**

- Default file permission: 666 - umask
- Default directory permission: 777 - umask
- Default umask: `0022` → files get 644, dirs get 755
- Set in `/etc/profile`, `~/.bashrc`, or `/etc/login.defs`

```bash
umask           # Show current
umask 0027      # Set: files=640, dirs=750
```

---

**Q24. How to delegate user management to non-root users without full sudo?**

```bash
# In /etc/sudoers.d/usermgmt
Cmnd_Alias USERMGMT = /usr/sbin/useradd, /usr/sbin/usermod, /usr/bin/passwd, /usr/sbin/userdel
krishna ALL=(root) USERMGMT, !/usr/bin/passwd root
```
The `!/usr/bin/passwd root` prevents them from changing root's password.

---

**Q25. Explain PAM (Pluggable Authentication Modules).**

PAM provides a flexible mechanism for authentication. Config files in `/etc/pam.d/`:
- **auth** — verifies identity (password, key, biometric)
- **account** — checks account restrictions (expiry, time-of-day)
- **password** — manages password changes (complexity, history)
- **session** — sets up environment (limits, logging)

Control flags: `required`, `requisite`, `sufficient`, `optional`

---

## 2. File System & Permissions

**Q26. Explain Linux file system hierarchy.**

| Directory | Purpose |
|-----------|---------|
| `/` | Root filesystem |
| `/boot` | Boot loader, kernel (vmlinuz, initramfs) |
| `/etc` | Configuration files |
| `/home` | User home directories |
| `/var` | Variable data (logs, spool, cache) |
| `/tmp` | Temporary files (cleared on reboot) |
| `/usr` | User programs and libraries |
| `/opt` | Third-party software |
| `/proc` | Virtual filesystem (kernel/process info) |
| `/sys` | Virtual filesystem (hardware/driver info) |
| `/dev` | Device files |

---

**Q27. What are the different file types in Linux?**

| Symbol | Type |
|--------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` | Block device |
| `c` | Character device |
| `p` | Named pipe (FIFO) |
| `s` | Socket |

---

**Q28. Explain SUID, SGID, and Sticky Bit.**

| Permission | On Files | On Directories |
|------------|----------|----------------|
| **SUID (4)** | Executes as file owner | No effect |
| **SGID (2)** | Executes as group owner | New files inherit group |
| **Sticky (1)** | No effect | Only owner can delete files |

```bash
chmod 4755 /usr/bin/passwd    # SUID
chmod 2770 /shared            # SGID
chmod 1777 /tmp               # Sticky bit

# Find SUID files (security audit)
find / -perm -4000 -type f 2>/dev/null
```

---

**Q29. What is the difference between hard link and soft link?**

| Feature | Hard Link | Soft Link |
|---------|-----------|-----------|
| Command | `ln file link` | `ln -s file link` |
| Inode | Same inode | Different inode |
| Cross filesystem | No | Yes |
| Link to directory | No | Yes |
| Original deleted | Data still accessible | Broken link |
| File size | Same as original | Path length |

---

**Q30. Explain ACL (Access Control Lists).**

```bash
# Set ACL
setfacl -m u:krishna:rwx /shared/project
setfacl -m g:devops:rx /shared/project
setfacl -m d:u:krishna:rwx /shared/project   # Default (inherited)

# View ACL
getfacl /shared/project

# Remove ACL
setfacl -x u:krishna /shared/project
setfacl -b /shared/project                   # Remove all ACLs
```

---

**Q31. How to find and remove files older than 30 days?**

```bash
find /var/log -type f -name "*.log" -mtime +30 -delete
find /tmp -type f -atime +7 -exec rm -f {} \;

# Safer: list before deleting
find /var/log -type f -mtime +30 -ls
```

---

**Q32. Explain the difference between `mtime`, `atime`, and `ctime`.**

- **mtime** — Modification time (content changed)
- **atime** — Access time (file read/accessed)
- **ctime** — Change time (metadata changed: permissions, ownership)

```bash
stat filename                 # Show all timestamps
find / -mtime -1              # Modified in last 24h
find / -newer /tmp/marker     # Newer than marker file
```

---

**Q33. What are inodes? How to check inode usage?**

- Inode stores file metadata (permissions, owner, timestamps, data block pointers)
- Every file has a unique inode number within a filesystem
- Running out of inodes = can't create files even if disk has space

```bash
df -i                         # Inode usage per filesystem
ls -i filename                # Show inode number
find / -xdev -printf '%h\n' | sort | uniq -c | sort -rn | head   # Dirs with most files
```

---

**Q34. Filesystem is full but no large files found. What could be the issue?**

Possible causes:
1. **Deleted files held open** — `lsof +D /var | grep deleted`
2. **Inode exhaustion** — `df -i` (lots of small files)
3. **Hidden files in mount point** — unmount and check underlying directory
4. **Reserved blocks** — `tune2fs -l /dev/sda1 | grep Reserved` (ext4 reserves 5%)

```bash
# Fix reserved blocks
tune2fs -m 1 /dev/sda1       # Reduce to 1%
```

---

**Q35. How to recover a deleted file that's still open by a process?**

```bash
# Find the file descriptor
lsof | grep deleted
# Example output: nginx 1234 root 5w REG 8,1 1048576 /var/log/nginx/access.log (deleted)

# Copy from /proc
cp /proc/1234/fd/5 /var/log/nginx/access.log.recovered
```

---

**Q36. Explain `/proc` filesystem. Name important files.**

| File | Information |
|------|-------------|
| `/proc/cpuinfo` | CPU details |
| `/proc/meminfo` | Memory stats |
| `/proc/loadavg` | System load |
| `/proc/mounts` | Mounted filesystems |
| `/proc/net/dev` | Network interface stats |
| `/proc/sys/` | Kernel tunable parameters |
| `/proc/<PID>/` | Per-process info |

---

**Q37. How to make kernel parameter changes persistent?**

```bash
# Temporary
sysctl -w net.ipv4.ip_forward=1

# Persistent
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.d/99-custom.conf
sysctl --system               # Apply all
```

---

**Q38. How to extend a filesystem without unmounting (online resize)?**

```bash
# XFS (grow only, cannot shrink)
xfs_growfs /mount/point

# ext4
resize2fs /dev/mapper/vg-lv

# Verify
df -hT /mount/point
```

---

**Q39. What are the different filesystem types and when to use them?**

| FS | Max File Size | Max Volume | Use Case |
|----|---------------|------------|----------|
| ext4 | 16 TB | 1 EB | General purpose, default |
| XFS | 8 EB | 8 EB | Large files, high I/O (RHEL default) |
| Btrfs | 16 EB | 16 EB | Snapshots, compression |
| tmpfs | RAM-based | RAM-based | Temporary, high-speed |

---

**Q40. How to check and repair a filesystem?**

```bash
# ext4 (must unmount first)
umount /dev/sda2
fsck.ext4 -y /dev/sda2

# XFS
xfs_repair /dev/sda2
xfs_repair -L /dev/sda2      # Force log zeroing (data loss risk)

# Check without repair
fsck -n /dev/sda2
```

---

**Q41. Explain `fstab` entries and each field.**

```
/dev/sda1  /boot  xfs  defaults  0  1
```
| Field | Meaning |
|-------|---------|
| 1 | Device/UUID |
| 2 | Mount point |
| 3 | Filesystem type |
| 4 | Mount options (defaults, noexec, nosuid, ro) |
| 5 | Dump (0=no backup, 1=backup) |
| 6 | fsck order (0=skip, 1=root, 2=others) |

**Best practice:** Use UUID instead of device name:
```bash
UUID=abc123-def456  /data  xfs  defaults,noatime  0  2
```

---

**Q42. What is `noatime` mount option and why use it?**

- Disables updating access time on every file read
- **Reduces disk I/O significantly** (especially on busy servers)
- Safe for most workloads; recommended for databases and log servers

---

**Q43. How to find the 10 largest files/directories?**

```bash
# Largest files
find / -type f -exec du -h {} + | sort -rh | head -10

# Largest directories
du -h --max-depth=1 / | sort -rh | head -10

# Fast alternative
ncdu /
```

---

**Q44. What is the difference between `df` and `du`?**

- `df` — reports filesystem-level usage (includes deleted but open files)
- `du` — reports actual file usage by walking the directory tree
- **Mismatch** between df and du usually means deleted files still open

---

**Q45. How to mount an ISO image?**

```bash
mount -o loop /path/to/image.iso /mnt/iso

# In fstab
/path/to/image.iso  /mnt/iso  iso9660  loop,ro  0  0
```

---

**Q46. Explain symbolic links vs bind mounts.**

```bash
# Symlink
ln -s /data/logs /var/log/app

# Bind mount
mount --bind /data/logs /var/log/app

# Bind mount in fstab
/data/logs  /var/log/app  none  bind  0  0
```
Bind mounts work across chroot jails; symlinks don't.

---

**Q47. How to encrypt a filesystem partition?**

```bash
# LUKS encryption
cryptsetup luksFormat /dev/sdb1
cryptsetup luksOpen /dev/sdb1 secure_data
mkfs.xfs /dev/mapper/secure_data
mount /dev/mapper/secure_data /secure

# Auto-mount at boot (/etc/crypttab)
secure_data  /dev/sdb1  /root/keyfile  luks
```

---

**Q48. What is a swap and how to create it?**

```bash
# Create swap file (2GB)
dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# Persistent (fstab)
echo "/swapfile  swap  swap  defaults  0  0" >> /etc/fstab

# Check
swapon --show
free -h

# Adjust swappiness
sysctl vm.swappiness=10
echo "vm.swappiness=10" >> /etc/sysctl.d/99-swap.conf
```

---

**Q49. How to check which process is using a mount point?**

```bash
fuser -vm /mnt/data
lsof +D /mnt/data
# Force unmount
umount -l /mnt/data           # Lazy unmount
fuser -km /mnt/data           # Kill processes then unmount
```

---

**Q50. What is `quota` and how to configure it?**

```bash
# Enable quota on filesystem (fstab)
/dev/sda3  /home  xfs  defaults,usrquota,grpquota  0  2

# Remount
mount -o remount /home

# Set quota for user (ext4)
quotacheck -cugm /home
quotaon /home
edquota -u krishna            # Edit limits

# XFS quota
xfs_quota -x -c 'limit bsoft=5g bhard=6g krishna' /home
xfs_quota -x -c 'report -h' /home
```

---


## 3. Storage & LVM

**Q51. What is LVM and its components?**

| Component | Purpose |
|-----------|---------|
| **PV (Physical Volume)** | Physical disk/partition added to LVM |
| **VG (Volume Group)** | Pool of PVs combined together |
| **LV (Logical Volume)** | Virtual partition created from VG |
| **PE (Physical Extent)** | Smallest unit of allocation (default 4MB) |

---

**Q52. How to create LVM from scratch?**

```bash
# 1. Create Physical Volume
pvcreate /dev/sdb /dev/sdc

# 2. Create Volume Group
vgcreate vg_data /dev/sdb /dev/sdc

# 3. Create Logical Volume
lvcreate -L 100G -n lv_app vg_data
# Or use all free space:
lvcreate -l 100%FREE -n lv_app vg_data

# 4. Create filesystem and mount
mkfs.xfs /dev/vg_data/lv_app
mkdir /app
mount /dev/vg_data/lv_app /app

# 5. Persist in fstab
echo "/dev/vg_data/lv_app /app xfs defaults 0 2" >> /etc/fstab
```

---

**Q53. How to extend a logical volume online?**

```bash
# Extend LV by 20G
lvextend -L +20G /dev/vg_data/lv_app

# Resize filesystem
xfs_growfs /app                   # XFS
resize2fs /dev/vg_data/lv_app     # ext4

# Or do both in one command
lvextend -L +20G -r /dev/vg_data/lv_app
```

---

**Q54. VG is full. How to add more space?**

```bash
# Add new disk to existing VG
pvcreate /dev/sdd
vgextend vg_data /dev/sdd
pvdisplay                         # Verify new PE count
lvextend -l +100%FREE /dev/vg_data/lv_app -r
```

---

**Q55. How to reduce a logical volume? (ext4 only)**

```bash
# XFS CANNOT be shrunk! Only ext4.
umount /app
e2fsck -f /dev/vg_data/lv_app
resize2fs /dev/vg_data/lv_app 50G
lvreduce -L 50G /dev/vg_data/lv_app
mount /app
```

---

**Q56. How to migrate data from one PV to another?**

```bash
# Move all extents from sdb to sdd
pvmove /dev/sdb /dev/sdd

# Remove old PV from VG
vgreduce vg_data /dev/sdb
pvremove /dev/sdb
```

---

**Q57. How to take an LVM snapshot?**

```bash
# Create snapshot (needs free space in VG)
lvcreate -L 5G -s -n lv_app_snap /dev/vg_data/lv_app

# Mount snapshot (read-only)
mount -o ro /dev/vg_data/lv_app_snap /mnt/snap

# Restore from snapshot
lvconvert --merge /dev/vg_data/lv_app_snap
# Reboot required if original is mounted
```

---

**Q58. How to check LVM status and troubleshoot?**

```bash
pvs                               # Physical volumes summary
vgs                               # Volume groups summary
lvs                               # Logical volumes summary
pvdisplay                         # Detailed PV info
vgdisplay                         # Detailed VG info
lvdisplay                         # Detailed LV info
lvscan                            # Scan for LVs
vgscan                            # Scan for VGs
```

---

**Q59. What is the difference between RAID and LVM?**

| Feature | RAID | LVM |
|---------|------|-----|
| Purpose | Redundancy + Performance | Flexible volume management |
| Data protection | Yes (RAID 1,5,6,10) | No (unless combined) |
| Resize online | No | Yes |
| Snapshots | No | Yes |
| Best practice | Use RAID under LVM for both |

---

**Q60. Explain RAID levels.**

| RAID | Min Disks | Redundancy | Performance | Usable Space |
|------|-----------|-----------|-------------|--------------|
| 0 | 2 | None | Best read/write | 100% |
| 1 | 2 | Mirror | Good read | 50% |
| 5 | 3 | Single parity | Good read | (n-1)/n |
| 6 | 4 | Double parity | Good read | (n-2)/n |
| 10 | 4 | Mirror + stripe | Best | 50% |

---

**Q61. How to create a software RAID?**

```bash
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mkfs.xfs /dev/md0
mdadm --detail /dev/md0
cat /proc/mdstat

# Save config
mdadm --detail --scan >> /etc/mdadm.conf
```

---

**Q62. What is iSCSI? How to configure a client?**

```bash
# Install
yum install iscsi-initiator-utils -y

# Discover targets
iscsiadm -m discovery -t st -p 192.168.1.100

# Login
iscsiadm -m node -T iqn.2024-01.com.example:storage -p 192.168.1.100 -l

# Auto-login at boot
iscsiadm -m node -T iqn.2024-01.com.example:storage -p 192.168.1.100 --op update -n node.startup -v automatic
```

---

**Q63. How to identify which disk a partition belongs to?**

```bash
lsblk -f                          # Full tree with filesystem info
blkid                             # UUID and filesystem type
fdisk -l                          # Partition table
pvs -o+devices                    # PV to device mapping
```

---

**Q64. What are the partition table types?**

| Feature | MBR | GPT |
|---------|-----|-----|
| Max disk size | 2 TB | 9.4 ZB |
| Max partitions | 4 primary (or 3+1 extended) | 128 |
| Boot mode | BIOS | UEFI |
| Tool | `fdisk` | `gdisk` or `parted` |

---

**Q65. How to add a new disk without rebooting?**

```bash
# Scan for new disks
echo "- - -" > /sys/class/scsi_host/host0/scan
echo "- - -" > /sys/class/scsi_host/host1/scan
echo "- - -" > /sys/class/scsi_host/host2/scan

# Verify
lsblk
fdisk -l
dmesg | tail
```

---

**Q66. How to check disk health?**

```bash
smartctl -a /dev/sda              # SMART data
smartctl -t short /dev/sda        # Run short test
badblocks -sv /dev/sda            # Check bad sectors (slow)
```

---

**Q67. What is multipath and why is it used?**

- Provides redundant paths between server and SAN storage
- Failover: if one path fails, traffic uses another
- Load balancing across paths

```bash
multipath -ll                     # Show multipath topology
multipath -f <mpath>             # Flush a multipath device
systemctl restart multipathd
```

---

**Q68. How to resize a partition on a running system?**

```bash
# Using growpart (for cloud VMs)
growpart /dev/sda 2
xfs_growfs /

# Using parted
parted /dev/sda resizepart 2 100%
```

---

**Q69. Disk I/O is slow. How to troubleshoot?**

```bash
iostat -xz 1 5                    # Check await, %util
iotop -oP                         # Top I/O consuming processes
dmesg | grep -i error             # Hardware errors
cat /sys/block/sda/queue/scheduler  # Check I/O scheduler
smartctl -H /dev/sda              # Disk health
```

---

**Q70. How to configure stratis storage (RHEL 8+)?**

```bash
yum install stratisd stratis-cli -y
systemctl enable --now stratisd

stratis pool create mypool /dev/sdb
stratis fs create mypool myfs
mount /dev/stratis/mypool/myfs /data
```

---

**Q71. What is VDO (Virtual Data Optimizer)?**

- Provides deduplication and compression for block storage
- Reduces actual disk usage

```bash
yum install vdo kmod-kvdo -y
vdo create --name=vdo1 --device=/dev/sdb --vdoLogicalSize=100G
mkfs.xfs -K /dev/mapper/vdo1
vdostats --human-readable
```

---

**Q72. How to troubleshoot "no space left on device" when `df` shows free space?**

```bash
# Check inodes
df -i

# Check deleted open files
lsof +L1

# Check for files hidden under mount point
umount /data
ls /data/    # Check for files here

# Check reserved blocks (ext4)
tune2fs -l /dev/sda1 | grep -i reserved
```

---

**Q73. What is the difference between `fdisk`, `gdisk`, and `parted`?**

| Tool | Table | Interactive | Scripting |
|------|-------|-------------|-----------|
| fdisk | MBR | Yes | Limited |
| gdisk | GPT | Yes | Limited |
| parted | Both | Yes | Yes (`parted -s`) |

---

**Q74. How to check SAN disk connectivity?**

```bash
multipath -ll
lsscsi
cat /proc/scsi/scsi
sg_inq /dev/sda                   # SCSI inquiry
systool -c fc_host -v             # FC HBA info
```

---

**Q75. How to configure automount (autofs)?**

```bash
yum install autofs -y

# /etc/auto.master
/mnt/nfs  /etc/auto.nfs  --timeout=300

# /etc/auto.nfs
data  -rw,sync  server:/export/data

systemctl enable --now autofs
# Access /mnt/nfs/data — mounts automatically
```

---

## 4. Networking

**Q76. How to check all network interfaces and their IPs?**

```bash
ip addr show                      # Modern (preferred)
ip -4 addr                        # IPv4 only
ifconfig                          # Legacy
nmcli device status               # NetworkManager
```

---

**Q77. How to configure a static IP (RHEL 7+)?**

```bash
# Using nmcli
nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
nmcli con mod eth0 ipv4.gateway 192.168.1.1
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.method manual
nmcli con up eth0

# Verify
nmcli con show eth0
```

---

**Q78. How to troubleshoot network connectivity?**

```bash
# Layer by layer approach:
ping -c 3 gateway_ip             # 1. Local network
ping -c 3 8.8.8.8               # 2. Internet (IP)
nslookup google.com             # 3. DNS resolution
traceroute google.com           # 4. Path/routing
curl -I http://google.com       # 5. Application layer
ss -tulnp                       # 6. Local listening ports
```

---

**Q79. Explain the difference between `ss` and `netstat`.**

- `ss` — newer, faster (reads from kernel directly via netlink)
- `netstat` — deprecated, reads from `/proc/net`

```bash
ss -tulnp                        # TCP/UDP listening with process
ss -s                            # Socket statistics summary
ss -t state established          # Established connections
ss dst 192.168.1.100             # Connections to specific IP
```

---

**Q80. How to check which process is using a specific port?**

```bash
ss -tulnp | grep :8080
lsof -i :8080
fuser 8080/tcp
netstat -tulnp | grep 8080       # Legacy
```

---

**Q81. Explain NIC bonding/teaming modes.**

| Mode | Name | Description |
|------|------|-------------|
| 0 | balance-rr | Round-robin (load balance) |
| 1 | active-backup | Failover (most common) |
| 2 | balance-xor | XOR hash-based |
| 3 | broadcast | Send on all slaves |
| 4 | 802.3ad | LACP (requires switch support) |
| 5 | balance-tlb | Adaptive transmit |
| 6 | balance-alb | Adaptive load balance |

---

**Q82. How to configure NIC bonding (RHEL 7+)?**

```bash
nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup,miimon=100"
nmcli con add type ethernet con-name bond0-slave1 ifname eth0 master bond0
nmcli con add type ethernet con-name bond0-slave2 ifname eth1 master bond0
nmcli con mod bond0 ipv4.addresses 192.168.1.100/24 ipv4.method manual
nmcli con up bond0

# Verify
cat /proc/net/bonding/bond0
```

---

**Q83. How to configure firewalld?**

```bash
# Status
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --get-active-zones

# Add rules (permanent)
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.0.0.0/8 port port=22 protocol=tcp accept'
firewall-cmd --reload

# Zones
firewall-cmd --permanent --zone=internal --add-source=192.168.1.0/24
```

---

**Q84. What's the difference between `firewalld` and `iptables`?**

| Feature | firewalld | iptables |
|---------|-----------|----------|
| RHEL version | 7+ (default) | 6 (default) |
| Zones | Yes | No |
| Runtime + Permanent | Yes | Only persistent after save |
| Reload without disconnect | Yes | No |
| Backend | nftables (RHEL 8+) | netfilter |

---

**Q85. How to check and flush routing table?**

```bash
ip route show                     # Show routes
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0  # Add route
ip route del 10.0.0.0/8          # Delete route

# Persistent route
nmcli con mod eth0 +ipv4.routes "10.0.0.0/8 192.168.1.1"
nmcli con up eth0
```

---

**Q86. How to capture network traffic?**

```bash
tcpdump -i eth0 port 80 -w capture.pcap
tcpdump -i any host 192.168.1.100 and port 443
tcpdump -i eth0 -nn -c 100      # No DNS resolution, 100 packets
```

---

**Q87. Explain DNS resolution order in Linux.**

1. `/etc/hosts` (local)
2. `/etc/resolv.conf` (DNS servers)
3. Order defined in `/etc/nsswitch.conf` (`hosts: files dns`)

```bash
dig google.com +short            # Query DNS
nslookup google.com
host google.com
getent hosts google.com          # Uses nsswitch order
```

---

**Q88. How to configure DNS client?**

```bash
# Modern (NetworkManager)
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.dns-search "example.com"
nmcli con up eth0

# Check
cat /etc/resolv.conf
resolvectl status                # systemd-resolved
```

---

**Q89. How to set hostname permanently?**

```bash
hostnamectl set-hostname server01.example.com
# Verify
hostname
hostnamectl
cat /etc/hostname
```

---

**Q90. What is the difference between TCP and UDP?**

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | Best-effort |
| Ordering | Ordered | Unordered |
| Speed | Slower | Faster |
| Use case | HTTP, SSH, DB | DNS, DHCP, streaming |

---

**Q91. How to check network bandwidth and throughput?**

```bash
# Install iperf3 on both ends
iperf3 -s                        # Server
iperf3 -c server_ip              # Client

# Quick check
nload                            # Real-time bandwidth
iftop                            # Per-connection bandwidth
vnstat                           # Historical traffic
```

---

**Q92. What is VLAN and how to configure it?**

```bash
# Create VLAN interface
nmcli con add type vlan con-name vlan100 dev eth0 id 100
nmcli con mod vlan100 ipv4.addresses 10.10.100.1/24 ipv4.method manual
nmcli con up vlan100
```

---

**Q93. How to troubleshoot DNS resolution failure?**

```bash
# Check DNS servers
cat /etc/resolv.conf
resolvectl status

# Test resolution
dig @8.8.8.8 example.com        # Direct to Google DNS
nslookup example.com 8.8.8.8

# Check if it's local or DNS server issue
ping 8.8.8.8                    # If works, network is fine
getent hosts example.com        # Check nsswitch order
```

---

**Q94. What is SSH tunneling and how to set it up?**

```bash
# Local port forwarding (access remote:3306 via localhost:9906)
ssh -L 9906:localhost:3306 user@remote-server

# Remote port forwarding (expose local:8080 on remote:9090)
ssh -R 9090:localhost:8080 user@remote-server

# Dynamic SOCKS proxy
ssh -D 1080 user@remote-server
```

---

**Q95. How to configure SSH key-based authentication?**

```bash
# Generate key pair
ssh-keygen -t ed25519 -C "krishna@server"

# Copy to remote
ssh-copy-id user@remote-host

# Or manually
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

**Q96. How to restrict SSH access to specific users/IPs?**

```bash
# /etc/ssh/sshd_config
AllowUsers krishna admin
AllowGroups sshusers
DenyUsers baduser

# IP restriction via firewalld
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.0.0.0/24 service name=ssh accept'
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --reload
```

---

**Q97. What is the difference between proxy and reverse proxy?**

| Feature | Forward Proxy | Reverse Proxy |
|---------|--------------|---------------|
| Sits in front of | Clients | Servers |
| Purpose | Privacy, filtering | Load balancing, SSL |
| Example | Squid | Nginx, HAProxy |
| Client knows? | Yes (configured) | No (transparent) |

---

**Q98. How to check ARP table and troubleshoot ARP issues?**

```bash
ip neigh show                    # Show ARP table
arp -a                           # Legacy
ip neigh flush all               # Clear ARP cache
arping -I eth0 192.168.1.1       # Check IP conflict
```

---

**Q99. How to configure network bridge?**

```bash
nmcli con add type bridge con-name br0 ifname br0
nmcli con add type ethernet con-name br0-port1 ifname eth0 master br0
nmcli con mod br0 ipv4.addresses 192.168.1.100/24 ipv4.method manual
nmcli con up br0
```

---

**Q100. How to check network interface errors and drops?**

```bash
ip -s link show eth0             # Statistics
ethtool -S eth0                  # Detailed NIC stats
cat /proc/net/dev                # All interface stats
netstat -i                       # Interface table
dmesg | grep -i eth0             # Driver messages
```

---


## 5. Services & Boot Process

**Q101. Explain the Linux boot process.**

1. **BIOS/UEFI** → POST (Power-On Self Test)
2. **Bootloader** → GRUB2 loads kernel
3. **Kernel** → Initializes hardware, mounts initramfs
4. **initramfs** → Loads drivers, finds root filesystem
5. **systemd (PID 1)** → Starts services based on target
6. **Default target** → multi-user.target (CLI) or graphical.target (GUI)

---

**Q102. How to change the default boot target?**

```bash
# Check current
systemctl get-default

# Change
systemctl set-default multi-user.target      # CLI
systemctl set-default graphical.target       # GUI

# Boot once into different target
systemctl isolate rescue.target
```

---

**Q103. How to reset root password (RHEL 7+)?**

1. Reboot → interrupt GRUB → press `e`
2. Find line starting with `linux` → append `rd.break`
3. Press `Ctrl+X` to boot
4. At emergency shell:
```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel                # Required if SELinux enabled
exit
reboot
```

---

**Q104. How to manage services with systemd?**

```bash
systemctl start|stop|restart|reload <service>
systemctl enable|disable <service>
systemctl status <service>
systemctl is-active <service>
systemctl is-enabled <service>
systemctl mask <service>          # Prevent starting completely
systemctl unmask <service>

# List services
systemctl list-units --type=service
systemctl list-units --type=service --state=failed
```

---

**Q105. What is the difference between `systemctl restart` and `systemctl reload`?**

- **restart** — stops and starts the service (brief downtime)
- **reload** — reloads config without stopping (no downtime, graceful)
- Not all services support reload (check with `systemctl cat <service>`)

---

**Q106. How to create a custom systemd service?**

```bash
cat > /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Application Service
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start.sh
ExecStop=/opt/myapp/bin/stop.sh
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now myapp
```

---

**Q107. What are systemd targets?**

| Target | Equivalent (SysVinit) | Purpose |
|--------|----------------------|---------|
| poweroff.target | runlevel 0 | Shutdown |
| rescue.target | runlevel 1 | Single-user |
| multi-user.target | runlevel 3 | Multi-user CLI |
| graphical.target | runlevel 5 | GUI |
| reboot.target | runlevel 6 | Reboot |
| emergency.target | — | Minimal shell |

---

**Q108. How to troubleshoot a service that fails to start?**

```bash
systemctl status <service> -l     # Full status + last logs
journalctl -u <service> -xe       # Detailed logs with explanation
journalctl -u <service> --since "5 min ago"
systemctl cat <service>           # View unit file
systemd-analyze blame             # Boot time per service
```

---

**Q109. How to schedule tasks with cron and systemd timers?**

```bash
# Cron
crontab -e
# MIN HOUR DOM MON DOW COMMAND
0 2 * * * /opt/scripts/backup.sh
*/5 * * * * /opt/scripts/health_check.sh

# List cron jobs
crontab -l
ls /etc/cron.d/

# Systemd timer (modern alternative)
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

---

**Q110. What is the difference between `cron` and `anacron`?**

| Feature | cron | anacron |
|---------|------|---------|
| Runs when system is off | No (missed) | Catches up after boot |
| Precision | Minutes | Days |
| Config | crontab | /etc/anacrontab |
| Use case | Servers (always on) | Laptops/desktops |

---

**Q111. How to troubleshoot a system that won't boot?**

1. Check if it's stuck at GRUB → reinstall grub
2. Kernel panic → boot older kernel from GRUB menu
3. Filesystem error → boot into rescue mode, run `fsck`
4. Service failing → boot with `systemd.unit=emergency.target`
5. SELinux relabel needed → add `autorelabel` to kernel command

---

**Q112. How to view and manage journal logs?**

```bash
journalctl                        # All logs
journalctl -b                     # Current boot
journalctl -b -1                  # Previous boot
journalctl -u sshd -f             # Follow specific service
journalctl --since "1 hour ago"
journalctl -p err                 # Only errors and above
journalctl --disk-usage           # Log storage size
journalctl --vacuum-size=500M     # Cleanup
```

---

**Q113. How to make journal logs persistent across reboots?**

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald

# Or edit /etc/systemd/journald.conf
Storage=persistent
```

---

**Q114. How to schedule a one-time task?**

```bash
# Using at
at 2:00 AM tomorrow
> /opt/scripts/migrate.sh
> <Ctrl+D>

# List pending jobs
atq

# Remove job
atrm <job_id>
```

---

**Q115. What is GRUB2 and how to recover it?**

```bash
# Reinstall GRUB
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg

# Edit default boot options
vim /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg

# List kernel entries
grub2-editenv list
awk -F\' '/menuentry / {print $2}' /boot/grub2/grub.cfg
```

---

**Q116. How to check system boot time and slow services?**

```bash
systemd-analyze                   # Total boot time
systemd-analyze blame             # Time per service (sorted)
systemd-analyze critical-chain    # Critical path
systemd-analyze plot > boot.svg   # Visual boot chart
```

---

**Q117. What is `initramfs` and how to rebuild it?**

- Initial RAM filesystem loaded by bootloader
- Contains drivers needed to mount root filesystem
- Rebuilt after kernel update or driver changes

```bash
# Rebuild
dracut -f                         # RHEL/CentOS
update-initramfs -u               # Ubuntu

# Rebuild for specific kernel
dracut -f /boot/initramfs-$(uname -r).img $(uname -r)
```

---

**Q118. How to prevent a specific service from starting at boot?**

```bash
# Disable
systemctl disable <service>

# Mask (cannot be started manually either)
systemctl mask <service>

# Verify
systemctl is-enabled <service>
```

---

**Q119. Explain socket activation in systemd.**

- Service starts only when connection is received
- Saves resources (service not running until needed)
- Example: `sshd.socket` starts `sshd.service` on connection

```bash
systemctl list-sockets
systemctl status sshd.socket
```

---

**Q120. What happens when you run `systemctl daemon-reload`?**

- Reloads systemd manager configuration
- Picks up changes to unit files in `/etc/systemd/system/`
- Does NOT restart any services
- Required after creating/modifying unit files

---

**Q121. How to check which services depend on a specific service?**

```bash
systemctl list-dependencies <service>         # What it depends on
systemctl list-dependencies --reverse <service>  # What depends on it
```

---

**Q122. What is `nohup` and how is it different from `screen`/`tmux`?**

| Feature | nohup | screen/tmux |
|---------|-------|-------------|
| Survives logout | Yes | Yes |
| Reattach | No | Yes |
| Multiple windows | No | Yes |
| Use case | Simple background tasks | Long interactive sessions |

```bash
nohup /opt/scripts/long_task.sh &
disown -h %1                      # Detach from terminal
```

---

**Q123. How to limit resources for a service using systemd?**

```bash
# In service file [Service] section
MemoryMax=2G
CPUQuota=200%                     # 2 CPU cores max
TasksMax=512
LimitNOFILE=65536

# Or via drop-in
systemctl edit myapp
# [Service]
# MemoryMax=2G
```

---

**Q124. How to configure logrotate?**

```bash
cat > /etc/logrotate.d/myapp << 'EOF'
/var/log/myapp/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser appgroup
    postrotate
        systemctl reload myapp
    endscript
}
EOF

# Test
logrotate -d /etc/logrotate.d/myapp    # Dry run
logrotate -f /etc/logrotate.d/myapp    # Force run
```

---

**Q125. How to check what's consuming the most disk space in /var/log?**

```bash
du -sh /var/log/* | sort -rh | head -10
journalctl --disk-usage
find /var/log -type f -name "*.log" -size +100M
```

---

## 6. Package Management

**Q126. What is the difference between `rpm` and `yum`/`dnf`?**

| Feature | rpm | yum/dnf |
|---------|-----|---------|
| Dependency resolution | No | Yes (automatic) |
| Repository support | No | Yes |
| Update system | No | Yes (`yum update`) |
| Use case | Single package ops | Full package management |

---

**Q127. How to check what package provides a specific file?**

```bash
yum provides /etc/httpd/conf/httpd.conf
rpm -qf /usr/bin/ls               # Which package owns this file
yum whatprovides "*/ifconfig"
```

---

**Q128. How to list all installed packages and their versions?**

```bash
rpm -qa                           # All packages
rpm -qa --last | head -20         # Recently installed
yum list installed
dnf list installed | wc -l       # Count
```

---

**Q129. How to downgrade a package?**

```bash
yum downgrade <package-name>
yum history list                  # Find transaction
yum history undo <id>             # Undo specific update
```

---

**Q130. How to create a local yum repository?**

```bash
# Install createrepo
yum install createrepo -y

# Create repo directory
mkdir -p /opt/localrepo
cp *.rpm /opt/localrepo/
createrepo /opt/localrepo

# Create repo file
cat > /etc/yum.repos.d/local.repo << 'EOF'
[localrepo]
name=Local Repository
baseurl=file:///opt/localrepo
enabled=1
gpgcheck=0
EOF

yum repolist
```

---

**Q131. How to check package dependencies?**

```bash
yum deplist <package>
rpm -qR <package>                 # Required dependencies
rpm -q --whatrequires <package>   # What requires this package
```

---

**Q132. How to install a specific version of a package?**

```bash
yum install httpd-2.4.6-97.el7
yum --showduplicates list httpd   # Show available versions
dnf install package-version.arch
```

---

**Q133. What is `yum versionlock`?**

Prevents a package from being updated:
```bash
yum install yum-plugin-versionlock
yum versionlock add httpd
yum versionlock list
yum versionlock delete httpd
```

---

**Q134. How to clean yum/dnf cache?**

```bash
yum clean all
yum clean metadata
yum makecache                     # Rebuild cache
rm -rf /var/cache/yum/*
```

---

**Q135. How to list enabled/disabled repositories?**

```bash
yum repolist all                  # All repos
yum repolist enabled              # Only enabled
yum-config-manager --enable <repo>
yum-config-manager --disable <repo>
```

---

**Q136. How to verify package integrity?**

```bash
rpm -V <package>                  # Verify installed package
rpm -K <package.rpm>              # Verify RPM signature
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

Output codes: `S`=size, `M`=mode, `5`=MD5, `T`=mtime, `U`=user, `G`=group

---

**Q137. How to install packages on Ubuntu/Debian?**

```bash
apt update
apt install <package>
apt remove <package>
apt purge <package>               # Remove + config files
apt autoremove                    # Remove unused dependencies
apt list --installed
dpkg -l | grep <package>
dpkg -L <package>                 # List files in package
```

---

**Q138. How to hold a package from updates (Debian/Ubuntu)?**

```bash
apt-mark hold <package>
apt-mark unhold <package>
apt-mark showhold
```

---

**Q139. What is the difference between `apt remove` and `apt purge`?**

- `remove` — removes package binaries but keeps config files
- `purge` — removes package + all configuration files

---

**Q140. How to extract files from an RPM without installing?**

```bash
rpm2cpio package.rpm | cpio -idmv
# Extract specific file
rpm2cpio package.rpm | cpio -iv --to-stdout ./etc/httpd/conf/httpd.conf > httpd.conf
```

---

**Q141. How to find which repository a package comes from?**

```bash
yum info <package>                # Shows "From repo" field
dnf repoquery --info <package>
```

---

**Q142. How to rebuild RPM database?**

```bash
rm -f /var/lib/rpm/__db*
rpm --rebuilddb
yum clean all
```

---

**Q143. What is `dnf` and how does it differ from `yum`?**

- `dnf` = next-generation `yum` (default in RHEL 8+)
- Faster dependency resolution (libsolv)
- Better memory usage
- Modular content support
- Compatible commands (yum is aliased to dnf in RHEL 8+)

---

**Q144. How to install a group of packages?**

```bash
yum grouplist
yum groupinstall "Development Tools"
dnf group install "Server with GUI"
```

---

**Q145. How to check if a package has known security vulnerabilities?**

```bash
yum updateinfo list security
yum updateinfo list cves
yum update --security             # Apply only security updates
dnf updateinfo list --sec-severity=Critical
```

---

## 7. Security & Hardening

**Q146. How to configure SELinux?**

```bash
# Check status
getenforce                        # Enforcing/Permissive/Disabled
sestatus

# Change mode (runtime)
setenforce 0                      # Permissive
setenforce 1                      # Enforcing

# Permanent (/etc/selinux/config)
SELINUX=enforcing

# Troubleshoot
ausearch -m avc --start recent
sealert -a /var/log/audit/audit.log
```

---

**Q147. How to change SELinux context for a file/directory?**

```bash
# Temporary
chcon -t httpd_sys_content_t /web/html

# Permanent (survives relabel)
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web

# Check context
ls -Z /web
```

---

**Q148. How to allow a service to use a non-standard port in SELinux?**

```bash
# Check current allowed ports
semanage port -l | grep http

# Add new port
semanage port -a -t http_port_t -p tcp 8443

# Verify
semanage port -l | grep 8443
```

---

**Q149. Explain SELinux booleans.**

```bash
# List all booleans
getsebool -a
getsebool -a | grep httpd

# Enable boolean
setsebool -P httpd_can_network_connect on     # -P = permanent

# Example: Allow Apache to connect to database
setsebool -P httpd_can_network_connect_db on
```

---

**Q150. How to harden SSH configuration?**

```bash
# /etc/ssh/sshd_config
Port 2222                         # Change default port
PermitRootLogin no
PasswordAuthentication no         # Key-only
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers krishna admin
Protocol 2
X11Forwarding no
LoginGraceTime 60

systemctl restart sshd
```

---


**Q151. How to configure `fail2ban` for SSH protection?**

```bash
yum install fail2ban -y

cat > /etc/fail2ban/jail.local << 'EOF'
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/secure
maxretry = 3
bantime = 3600
findtime = 600
EOF

systemctl enable --now fail2ban
fail2ban-client status sshd
```

---

**Q152. How to audit user activity on a Linux system?**

```bash
# Enable auditing
systemctl enable --now auditd

# Audit rules
auditctl -w /etc/passwd -p wa -k identity
auditctl -w /etc/shadow -p wa -k identity
auditctl -w /etc/sudoers -p wa -k sudoers_changes
auditctl -a always,exit -F arch=b64 -S execve -k commands

# Search audit logs
ausearch -k identity --start today
aureport --auth                   # Authentication report
aureport --login                  # Login report
```

---

**Q153. How to implement TCP wrappers?**

```bash
# /etc/hosts.allow (checked first)
sshd: 192.168.1.0/255.255.255.0
ALL: 10.0.0.5

# /etc/hosts.deny (checked second)
sshd: ALL
ALL: ALL

# Order: allow → deny → default (allow)
```

---

**Q154. How to check for rootkits?**

```bash
# Install rkhunter
yum install rkhunter -y
rkhunter --update
rkhunter --check

# Install chkrootkit
chkrootkit

# Check for suspicious files
find / -name ".*" -type f         # Hidden files
find / -perm -4000               # SUID files
```

---

**Q155. How to configure PAM for password complexity?**

```bash
# /etc/security/pwquality.conf (RHEL 7+)
minlen = 12
dcredit = -1                      # At least 1 digit
ucredit = -1                      # At least 1 uppercase
lcredit = -1                      # At least 1 lowercase
ocredit = -1                      # At least 1 special char
maxrepeat = 3                     # No more than 3 consecutive same chars
```

---

**Q156. How to set up `auditd` to monitor file access?**

```bash
# Monitor a specific file
auditctl -w /etc/shadow -p rwa -k shadow_access

# Monitor directory
auditctl -w /opt/app/config/ -p wa -k config_changes

# Make persistent
echo "-w /etc/shadow -p rwa -k shadow_access" >> /etc/audit/rules.d/custom.rules
systemctl restart auditd
```

---

**Q157. How to disable USB storage (security hardening)?**

```bash
# Blacklist USB storage module
echo "blacklist usb-storage" > /etc/modprobe.d/usb-storage.conf
echo "install usb-storage /bin/true" >> /etc/modprobe.d/usb-storage.conf

# Unload if already loaded
rmmod usb-storage
```

---

**Q158. How to check for open ports and close unnecessary ones?**

```bash
# Check open ports
ss -tulnp
nmap -sT localhost

# Close via firewalld
firewall-cmd --permanent --remove-port=8080/tcp
firewall-cmd --reload

# Disable the service using that port
systemctl stop <service>
systemctl disable <service>
```

---

**Q159. How to encrypt data in transit and at rest?**

```bash
# In transit: SSH, TLS/SSL
# At rest: LUKS
cryptsetup luksFormat /dev/sdb1
cryptsetup luksOpen /dev/sdb1 encrypted_vol
mkfs.xfs /dev/mapper/encrypted_vol

# GPG file encryption
gpg -c sensitive_file.txt         # Encrypt
gpg -d sensitive_file.txt.gpg     # Decrypt
```

---

**Q160. How to check and fix file permissions for security compliance?**

```bash
# Find world-writable files
find / -type f -perm -0002 -ls
find / -type d -perm -0002 -ls

# Find files without owner
find / -nouser -o -nogroup

# Fix common issues
chmod 600 /etc/shadow
chmod 644 /etc/passwd
chmod 640 /etc/gshadow
chmod 750 /home/*
```

---

**Q161. How to implement IP-based access control?**

```bash
# Firewalld rich rules
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.0.0.0/8 service name=ssh accept'
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=0.0.0.0/0 service name=ssh drop'
firewall-cmd --reload

# PAM (/etc/security/access.conf)
+ : krishna : 192.168.1.0/24
- : ALL : ALL
```

---

**Q162. How to set file immutable attribute?**

```bash
# Make file unchangeable (even by root)
chattr +i /etc/resolv.conf

# Remove immutable
chattr -i /etc/resolv.conf

# Check attributes
lsattr /etc/resolv.conf

# Append only
chattr +a /var/log/secure
```

---

**Q163. What is `AIDE` and how to use it?**

- Advanced Intrusion Detection Environment
- Monitors file integrity (like tripwire)

```bash
yum install aide -y
aide --init                       # Create baseline
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
aide --check                      # Compare against baseline
aide --update                     # Update baseline
```

---

**Q164. How to implement two-factor authentication for SSH?**

```bash
yum install google-authenticator -y
google-authenticator              # Run as user

# /etc/pam.d/sshd - add:
auth required pam_google_authenticator.so

# /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive

systemctl restart sshd
```

---

**Q165. How to restrict `su` to specific users?**

```bash
# Only wheel group members can su to root
# /etc/pam.d/su
auth required pam_wheel.so use_uid

# Add user to wheel
usermod -aG wheel krishna
```

---

**Q166. How to secure shared memory?**

```bash
# /etc/fstab - add:
tmpfs  /dev/shm  tmpfs  defaults,noexec,nosuid,nodev  0  0

mount -o remount /dev/shm
```

---

**Q167. How to implement central logging (rsyslog forwarding)?**

```bash
# Client: /etc/rsyslog.conf
*.* @@logserver.example.com:514     # TCP
*.* @logserver.example.com:514      # UDP

# Server: /etc/rsyslog.conf
module(load="imtcp")
input(type="imtcp" port="514")

template(name="RemoteLogs" type="string" string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")
*.* ?RemoteLogs

systemctl restart rsyslog
```

---

**Q168. How to check for unauthorized SUID/SGID binaries?**

```bash
# Find all SUID binaries
find / -perm -4000 -type f 2>/dev/null > /tmp/suid_current.txt

# Compare with baseline
diff /tmp/suid_baseline.txt /tmp/suid_current.txt

# Known legitimate SUID files: /usr/bin/passwd, /usr/bin/sudo, /usr/bin/su
```

---

**Q169. How to configure kernel parameters for security?**

```bash
cat >> /etc/sysctl.d/99-security.conf << 'EOF'
# Disable IP forwarding
net.ipv4.ip_forward = 0

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# Enable SYN cookies (DDoS protection)
net.ipv4.tcp_syncookies = 1

# Disable source routing
net.ipv4.conf.all.accept_source_route = 0

# Enable reverse path filtering
net.ipv4.conf.all.rp_filter = 1

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1

# Disable IPv6 if not needed
net.ipv6.conf.all.disable_ipv6 = 1
EOF

sysctl --system
```

---

**Q170. How to check last security patches applied?**

```bash
yum history list                  # All yum transactions
yum updateinfo list security --installed
rpm -qa --last | head -20
cat /var/log/yum.log | grep -i security
```

---

## 8. Performance & Troubleshooting

**Q171. Server is slow. How do you troubleshoot?**

```bash
# 1. Check load average
uptime
# Load > number of CPUs = overloaded

# 2. Check CPU
top -bn1 | head -20
mpstat -P ALL 1 3

# 3. Check memory
free -h
vmstat 1 5

# 4. Check disk I/O
iostat -xz 1 5
iotop -oP

# 5. Check network
iftop
ss -s

# 6. Check processes
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10
```

---

**Q172. How to identify memory leaks?**

```bash
# Check memory usage over time
while true; do free -h | grep Mem >> /tmp/memlog.txt; sleep 60; done

# Find process with growing memory
ps aux --sort=-%mem | head
watch -n 5 'ps aux --sort=-%mem | head -5'

# Detailed per-process
pmap -x <PID>
cat /proc/<PID>/status | grep -i vm

# Check for OOM events
dmesg | grep -i "out of memory"
journalctl -k | grep -i oom
```

---

**Q173. What is load average and how to interpret it?**

- Shows system load over 1, 5, 15 minutes
- **1.0 on single CPU** = fully utilized
- **Rule:** load average should be < number of CPU cores
- Example: 4-core server → load 4.0 = 100% utilized

```bash
uptime
cat /proc/loadavg
nproc                             # Number of CPU cores
```

---

**Q174. How to check which process is consuming the most CPU?**

```bash
top -bn1 -o %CPU | head -15
ps aux --sort=-%cpu | head -10
pidstat 1 5                       # Per-process CPU stats
mpstat -P ALL 1 3                 # Per-CPU stats
```

---

**Q175. How to check memory usage in detail?**

```bash
free -h
cat /proc/meminfo
vmstat -s
slabtop                           # Kernel slab cache

# Per-process memory
smem -tk                          # Shows PSS (proportional)
pmap -x <PID>
```

---

**Q176. What is the difference between buffers and cache in `free` output?**

- **Buffers** — kernel buffer cache for block device I/O (metadata)
- **Cache** — page cache for file content
- Both are reclaimable — kernel frees them when apps need memory
- **Available** column = free + reclaimable (actual free memory)

---

**Q177. How to clear system cache (buffers/cache)?**

```bash
sync                              # Flush writes to disk first
echo 1 > /proc/sys/vm/drop_caches    # Page cache
echo 2 > /proc/sys/vm/drop_caches    # Dentries + inodes
echo 3 > /proc/sys/vm/drop_caches    # All

# NOT recommended in production (performance impact)
```

---

**Q178. How to monitor disk I/O performance?**

```bash
iostat -xz 1 5
# Key metrics:
# %util > 80% = disk saturated
# await > 10ms = high latency
# r/s, w/s = IOPS

iotop -oP                         # Real-time per-process I/O
dstat -d                          # Disk stats
```

---

**Q179. How to check and tune I/O scheduler?**

```bash
# Check current scheduler
cat /sys/block/sda/queue/scheduler

# Change (runtime)
echo "deadline" > /sys/block/sda/queue/scheduler

# Common schedulers:
# noop/none — SSD, VM guests
# deadline/mq-deadline — databases
# cfq/bfq — desktops, general
```

---

**Q180. How to trace system calls of a process?**

```bash
strace -p <PID>                   # Attach to running process
strace -f -e trace=network <command>  # Network calls
strace -c <command>               # Summary statistics
strace -e open,read,write ls      # Specific syscalls
ltrace <command>                  # Library calls
```

---

**Q181. How to find zombie processes and fix them?**

```bash
# Find zombies
ps aux | grep Z
ps -eo pid,ppid,stat,comm | grep Z

# Kill parent to clean zombies
kill -SIGCHLD <parent_PID>        # Ask parent to reap
kill -9 <parent_PID>              # Force (last resort)
```

---

**Q182. How to check network latency and packet loss?**

```bash
ping -c 100 target_host           # Check loss percentage
mtr target_host                   # Continuous traceroute
traceroute -T target_host         # TCP traceroute
ss -ti                            # TCP socket info (retransmits)
```

---

**Q183. How to check and increase file descriptor limits?**

```bash
# Check current limits
ulimit -n                         # Soft limit
ulimit -Hn                        # Hard limit
cat /proc/<PID>/limits

# Increase globally (/etc/security/limits.conf)
* soft nofile 65536
* hard nofile 65536

# For specific service (systemd)
# [Service]
# LimitNOFILE=65536

# System-wide max
sysctl fs.file-max
echo "fs.file-max = 2097152" >> /etc/sysctl.d/99-files.conf
sysctl --system
```

---

**Q184. How to monitor real-time system activity?**

```bash
top                               # Process viewer
htop                              # Enhanced process viewer
atop                              # Advanced system monitor
dstat                             # Versatile resource stats
glances                           # System overview
sar -u 1 10                       # CPU
sar -r 1 10                       # Memory
sar -d 1 10                       # Disk
sar -n DEV 1 10                   # Network
```

---

**Q185. How to use `perf` for performance analysis?**

```bash
# CPU profiling
perf top                          # Live function hotspots
perf record -g -p <PID> -- sleep 30
perf report                       # Analyze recording

# System-wide stats
perf stat -a sleep 5
```

---

**Q186. How to troubleshoot high CPU usage by `kswapd`?**

- `kswapd` = kernel swap daemon (reclaiming memory)
- High usage means system is under memory pressure

```bash
# Check
free -h
vmstat 1 5                        # Check si/so (swap in/out)
cat /proc/meminfo | grep -i swap

# Solutions:
# 1. Add more RAM
# 2. Reduce swappiness
sysctl vm.swappiness=10
# 3. Identify memory-hungry process
ps aux --sort=-%mem | head
# 4. Kill or restart leaking process
```

---

**Q187. How to check TCP connection states?**

```bash
ss -tan | awk '{print $1}' | sort | uniq -c | sort -rn
# Common states:
# ESTABLISHED — active connections
# TIME_WAIT — connection closed, waiting to expire
# CLOSE_WAIT — remote closed, local hasn't (possible leak)
# SYN_RECV — under SYN flood attack

# Too many TIME_WAIT? Tune:
echo "net.ipv4.tcp_tw_reuse = 1" >> /etc/sysctl.d/99-tcp.conf
sysctl --system
```

---

**Q188. How to check if a port is reachable from this server?**

```bash
# Using telnet
telnet target_host 443

# Using nc (netcat)
nc -zv target_host 443

# Using curl
curl -v telnet://target_host:443

# Using bash
echo > /dev/tcp/target_host/443 && echo "Open" || echo "Closed"
```

---

**Q189. How to profile a slow application?**

```bash
# Check what it's doing
strace -cp <PID>                  # System call summary
lsof -p <PID>                    # Open files/sockets
/proc/<PID>/status                # Memory status

# Check for locks
cat /proc/locks | grep <PID>

# Thread analysis
ps -Lf -p <PID>                  # List threads
top -H -p <PID>                  # Per-thread CPU
```

---

**Q190. How to identify and handle OOM (Out of Memory) killer?**

```bash
# Check OOM events
dmesg | grep -i "out of memory"
journalctl -k | grep -i oom
grep -i "killed process" /var/log/messages

# Adjust OOM score (protect a process)
echo -1000 > /proc/<PID>/oom_score_adj    # Never kill
echo 1000 > /proc/<PID>/oom_score_adj     # Kill first

# For systemd services
# [Service]
# OOMScoreAdjust=-900
```

---

**Q191. How to check historical performance data?**

```bash
# sar (System Activity Reporter)
sar -u -f /var/log/sa/sa$(date +%d -d yesterday)   # CPU yesterday
sar -r -f /var/log/sa/sa15                          # Memory on 15th
sar -b -s 08:00:00 -e 17:00:00                     # I/O 8am-5pm

# Install sysstat if not available
yum install sysstat -y
systemctl enable --now sysstat
```

---

**Q192. What is a fork bomb and how to prevent it?**

```bash
# Fork bomb: :(){ :|:& };:

# Prevention (/etc/security/limits.conf)
* soft nproc 4096
* hard nproc 8192

# Per-user
krishna hard nproc 2048

# Systemd service
# [Service]
# TasksMax=512
```

---

**Q193. How to check and increase kernel semaphore limits?**

```bash
# Check current
cat /proc/sys/kernel/sem
ipcs -s                           # Show semaphores
ipcs -l                           # Show limits

# Increase
echo "kernel.sem = 250 32000 100 128" >> /etc/sysctl.d/99-sem.conf
sysctl --system
```

---

**Q194. How to troubleshoot "too many open files" error?**

```bash
# Find which process hit the limit
lsof | wc -l                     # Total open files
lsof -p <PID> | wc -l           # Per process
cat /proc/<PID>/limits | grep "Max open files"

# Current system-wide usage
cat /proc/sys/fs/file-nr         # allocated  free  max

# Fix: increase limits (see Q183)
```

---

**Q195. How to check network throughput between two servers?**

```bash
# Server side
iperf3 -s

# Client side
iperf3 -c server_ip -t 30 -P 4   # 30 seconds, 4 parallel streams

# Quick alternative with dd + ssh
dd if=/dev/zero bs=1M count=1024 | ssh user@remote "cat > /dev/null"
```

---

## 9. Scenario-Based Questions

**Q196. A production server suddenly becomes unreachable. Walk through your troubleshooting steps.**

1. **Check from console** (iLO/iDRAC/IPMI/VMware console) — is it up?
2. **Check if kernel panic** — console output
3. **Check network** — `ip addr`, `ip route`, `ping gateway`
4. **Check disk** — `df -h`, `dmesg | grep error`
5. **Check logs** — `journalctl -xe`, `/var/log/messages`
6. **Check OOM** — `dmesg | grep -i "out of memory"`
7. **Check services** — `systemctl list-units --state=failed`
8. **Inform monitoring team** — acknowledge alert
9. **Engage network team** if network issue
10. **Document and RCA** after resolution

---

**Q197. Filesystem is 100% full on a production server. How do you handle it?**

```bash
# 1. Identify largest consumers
du -sh /* | sort -rh | head
du -sh /var/log/* | sort -rh | head

# 2. Quick fixes
journalctl --vacuum-size=200M
find /var/log -name "*.gz" -mtime +7 -delete
find /tmp -type f -mtime +3 -delete

# 3. Check for deleted but open files
lsof +L1 | grep deleted

# 4. Truncate (don't delete) active log files
> /var/log/large-active.log

# 5. If application logs: implement logrotate
# 6. Long-term: extend filesystem (LVM)
lvextend -L +10G -r /dev/vg/lv_var
```

---

**Q198. Server is experiencing intermittent high load. How do you investigate?**

```bash
# 1. Set up monitoring
sar -u 1 > /tmp/cpu_log.txt &
sar -r 1 > /tmp/mem_log.txt &
vmstat 1 > /tmp/vm_log.txt &

# 2. Check during spike
top -bn1 -o %CPU | head -20
iostat -xz 1 3
ss -s                             # Connection counts

# 3. Check for cron jobs at that time
grep -r "30 \* \*" /etc/cron* /var/spool/cron/

# 4. Check for memory pressure
vmstat 1 5                        # si/so columns
dmesg | grep -i oom

# 5. Historical data
sar -u -s 14:00:00 -e 14:30:00   # CPU during incident window
```

---

**Q199. An application team reports their service is slow but server resources look fine. How do you help?**

```bash
# 1. Confirm "fine" — check properly
top, free -h, iostat, ss -s

# 2. Check application-specific
systemctl status <app_service>
journalctl -u <app_service> --since "30 min ago" | grep -i error

# 3. Check network
ss -tnp | grep <app_port> | wc -l    # Connection count
ss -tnp | grep CLOSE_WAIT            # Leaked connections

# 4. Check DNS resolution speed
time nslookup app-dependency.internal

# 5. Check connectivity to dependencies
nc -zv database-server 5432
nc -zv redis-server 6379

# 6. Check for resource limits
cat /proc/<PID>/limits
lsof -p <PID> | wc -l

# 7. Suggest: enable debug logging, check APM (Dynatrace/AppDynamics)
```

---

**Q200. You need to patch 500 servers with zero downtime. Describe your approach.**

1. **Planning:**
   - Group servers by application/role
   - Identify patch window per group
   - Get change approval (CAB)

2. **Pre-checks:**
   - Verify backups are current
   - Check cluster/HA status
   - Snapshot VMs if applicable

3. **Rolling update strategy:**
   - Remove server from load balancer
   - Apply patches: `yum update -y`
   - Reboot if kernel update: `needs-restarting -r`
   - Run health checks (application + OS)
   - Add back to load balancer
   - Move to next server

4. **Automation:**
   ```bash
   # Ansible playbook approach
   ansible-playbook patch.yml --limit batch1 --serial 1
   ```

5. **Post-patch:**
   - Verify all services running
   - Check monitoring for anomalies
   - Document patched servers
   - Update CMDB

6. **Rollback plan:**
   - Restore from snapshot (VMs)
   - `yum history undo <id>` (packages)
   - Restore from backup (worst case)

---

<div align="center">

## 🎯 Quick Tips for Interview

| Topic | Key Command to Remember |
|-------|------------------------|
| Boot issue | `rd.break` → `chroot /sysroot` → `passwd` |
| Disk full | `lsof +L1`, `du -sh /*`, `journalctl --vacuum-size` |
| High CPU | `top`, `ps aux --sort=-%cpu`, `strace -cp PID` |
| High Memory | `free -h`, `ps aux --sort=-%mem`, OOM check |
| Network | `ss -tulnp`, `tcpdump`, `traceroute`, `mtr` |
| LVM extend | `lvextend -L +10G -r /dev/vg/lv` |
| SELinux | `getenforce`, `semanage`, `restorecon` |
| Service fail | `systemctl status`, `journalctl -u -xe` |

---

**⭐ Star this repo if it helped you prepare!**

*Prepared by [Krishna Yada](https://github.com/yadakrishna245) — Senior System Administrator | 8+ Years | 4000+ Production Servers*

</div>
