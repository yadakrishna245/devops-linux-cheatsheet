<div align="center">

# 🐧 Linux Administration — Quick Reference Guide

**A Complete Reference for Linux System Administrators**

*Prepared by Krishna Yada | Senior System Administrator | 8+ Years | 4000+ Servers*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Krishna_Yada-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/krishna-yada-8a8441239)
[![GitHub](https://img.shields.io/badge/GitHub-yadakrishna245-black?style=flat-square&logo=github)](https://github.com/yadakrishna245)

</div>

---

## 📋 Table of Contents

| # | Chapter | Topics |
|---|---------|--------|
| 1 | [User & Group Administration](#chapter-1---user--group-administration) | Users, Groups, Sudo, PAM |
| 2 | [File System Management](#chapter-2---file-system-management) | Permissions, ACL, Links, Inodes |
| 3 | [LVM & Storage](#chapter-3---lvm--storage-management) | PV, VG, LV, RAID, NFS |
| 4 | [Network Configuration](#chapter-4---network-configuration) | IP, Bonding, Firewall, DNS |
| 5 | [Boot Process & Services](#chapter-5---boot-process--services) | GRUB2, Systemd, Targets |
| 6 | [Package Management](#chapter-6---package-management) | RPM, YUM/DNF, Repositories |
| 7 | [Apache & Web Services](#chapter-7---web-services) | Apache, Nginx, Virtual Hosts |
| 8 | [Samba & File Sharing](#chapter-8---samba--file-sharing) | SMB, CIFS, NFS |
| 9 | [SSH & Remote Access](#chapter-9---ssh--remote-access) | SSH Config, Keys, Tunneling |
| 10 | [SELinux & Security](#chapter-10---selinux--security) | Contexts, Booleans, Policies |
| 11 | [Performance & Monitoring](#chapter-11---performance--monitoring) | top, sar, vmstat, iostat |
| 12 | [Backup & Recovery](#chapter-12---backup--recovery) | tar, rsync, dd, snapshots |
| 13 | [Real-World Scenarios](#chapter-13---real-world-scenarios) | Production troubleshooting |

---

## Chapter 1 - User & Group Administration

### 1.1 Types of Users

| Type | UID Range (RHEL 7/8/9) | Home Directory | Shell |
|------|------------------------|----------------|-------|
| Super User (root) | 0 | /root | /bin/bash |
| Normal User | 1000–60000 | /home/\<username\> | /bin/bash |
| Static System User | 1–200 | /var/\<service\> | /sbin/nologin |
| Dynamic System User | 201–999 | /var/\<service\> | /sbin/nologin |
| Network User (LDAP/AD) | Same as normal | /home/guests/\<user\> | /bin/bash |
| Sudo User | Same as normal | /home/\<username\> | /bin/bash |

### 1.2 Important User Files

| File | Purpose |
|------|---------|
| `/etc/passwd` | User info: `username:x:UID:GID:comment:home:shell` |
| `/etc/shadow` | Encrypted passwords + aging policies |
| `/etc/group` | Group info: `groupname:x:GID:members` |
| `/etc/gshadow` | Group passwords |
| `/etc/login.defs` | Default login settings (password aging) |
| `/etc/skel` | Template files for new users |
| `/etc/default/useradd` | Default useradd values |

### 1.3 User Management Commands

```bash
# Create user with full options
useradd -u <uid> -g <gid> -G <secondary_groups> -c "comment" -d /home/user -s /bin/bash username

# Create multiple users from file
# File format: username:password:UID:GID:comment:home:shell
newusers /path/to/users_file

# Modify user
usermod -aG wheel,docker username     # Add to groups
usermod -L username                   # Lock account
usermod -U username                   # Unlock account
usermod -s /sbin/nologin username     # Disable login

# Delete user
userdel username                      # Keep home directory
userdel -r username                   # Remove home + mail

# Check user info
id username
finger username
getent passwd username
```

### 1.4 Password Management

```bash
# Set/change password
passwd username
echo "username:newpassword" | chpasswd

# Password aging
chage -l username                     # View policy
chage -M 90 -m 7 -W 14 -I 5 username
chage -d 0 username                   # Force change on next login
chage -E 2025-12-31 username          # Set expiry date

# Check password status
passwd -S username                    # L=locked, P=usable, NP=no password

# System-wide defaults (/etc/login.defs)
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_WARN_AGE   14
PASS_MIN_LEN    8
```

### 1.5 Group Management

```bash
# Create/modify/delete groups
groupadd -g 5000 devops
groupmod -n newname oldname
groupdel groupname

# Group password
gpasswd groupname                     # Set group password
gpasswd -a username groupname         # Add user to group
gpasswd -d username groupname         # Remove user from group
gpasswd -M user1,user2 groupname      # Set member list

# Verify integrity
grpck                                 # Check /etc/group
pwck                                  # Check /etc/passwd
```

### 1.6 Sudo Configuration

```bash
# Edit sudoers (always use visudo)
visudo

# Grant full sudo
username ALL=(ALL) ALL
username ALL=(ALL) NOPASSWD: ALL

# Command aliases
Cmnd_Alias SERVICES = /usr/bin/systemctl restart *, /usr/bin/systemctl status *
Cmnd_Alias NETWORKING = /usr/sbin/route, /usr/sbin/ifconfig
username ALL=(ALL) SERVICES, NETWORKING

# User alias
User_Alias ADMINS = krishna, raju, gopal
ADMINS ALL=(ALL) ALL

# Restrict (allow all except passwd root)
krishna ALL=(ALL) ALL, !/usr/bin/passwd root

# Timestamp (ask password every time)
Defaults timestamp_timeout=0

# Per-user file (preferred)
echo "krishna ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/krishna
chmod 440 /etc/sudoers.d/krishna

# Sudo logs
cat /var/log/secure                   # RHEL
journalctl _COMM=sudo                 # Systemd
```

### 1.7 User Login Control

```bash
# Check who is logged in
who                                   # Current users
w                                     # Detailed activity
last                                  # Login history
lastb                                 # Failed logins
lastlog                               # Last login per user

# Restrict login
usermod -s /sbin/nologin username     # No shell access
usermod -e 2024-01-01 username        # Expire account

# /etc/security/access.conf
+ : krishna : 192.168.1.0/24
- : ALL : ALL
```

---


## Chapter 2 - File System Management

### 2.1 Linux File System Hierarchy

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the filesystem |
| `/boot` | Kernel, initramfs, GRUB config |
| `/etc` | System configuration files |
| `/home` | User home directories |
| `/var` | Variable data: logs, spool, cache |
| `/tmp` | Temporary files (cleared on reboot) |
| `/usr` | User programs, libraries, docs |
| `/opt` | Third-party applications |
| `/proc` | Virtual FS — process/kernel info |
| `/sys` | Virtual FS — hardware/driver info |
| `/dev` | Device files (block, character) |
| `/mnt` | Temporary mount points |
| `/media` | Removable media mount points |
| `/srv` | Service data (web, ftp) |
| `/run` | Runtime data (PIDs, sockets) |

### 2.2 File Types

| Symbol | Type | Example |
|--------|------|---------|
| `-` | Regular file | text, binary, script |
| `d` | Directory | /home, /etc |
| `l` | Symbolic link | /lib → /usr/lib |
| `b` | Block device | /dev/sda |
| `c` | Character device | /dev/tty |
| `p` | Named pipe (FIFO) | mkfifo pipe1 |
| `s` | Socket | /var/run/docker.sock |

### 2.3 Permissions

```bash
# Numeric: r=4, w=2, x=1
chmod 755 file                        # rwxr-xr-x
chmod 640 file                        # rw-r-----
chmod -R 750 /opt/app                 # Recursive

# Symbolic
chmod u+x file                        # Add execute for owner
chmod g-w file                        # Remove write for group
chmod o=r file                        # Set others to read only
chmod a+x script.sh                   # Add execute for all

# Ownership
chown user:group file
chown -R appuser:appgroup /opt/app

# Default permissions (umask)
umask                                 # Show current (0022)
umask 0027                            # Files=640, Dirs=750
```

### 2.4 Special Permissions

| Permission | Numeric | On File | On Directory |
|------------|---------|---------|--------------|
| SUID | 4000 | Runs as file owner | No effect |
| SGID | 2000 | Runs as group owner | New files inherit group |
| Sticky | 1000 | No effect | Only owner can delete |

```bash
chmod 4755 /usr/bin/passwd            # SUID
chmod 2770 /shared/project            # SGID
chmod 1777 /tmp                       # Sticky bit

# Find special permission files
find / -perm -4000 -type f            # SUID files
find / -perm -2000 -type f            # SGID files
find / -perm -1000 -type d            # Sticky directories
```

### 2.5 Access Control Lists (ACL)

```bash
# Set ACL
setfacl -m u:krishna:rwx /project
setfacl -m g:devops:rx /project
setfacl -m d:u:krishna:rwx /project   # Default (for new files)
setfacl -m o::--- /project            # No access for others

# View ACL
getfacl /project

# Remove ACL
setfacl -x u:krishna /project         # Remove specific
setfacl -b /project                   # Remove all

# Backup and restore ACL
getfacl -R /project > acl_backup.txt
setfacl --restore=acl_backup.txt
```

### 2.6 Links

```bash
# Hard link (same inode, same filesystem only)
ln original.txt hardlink.txt
# Cannot cross filesystems, cannot link directories

# Soft/Symbolic link (different inode, can cross FS)
ln -s /opt/app/config.yml /etc/app/config.yml
ln -s /data/logs /var/log/app

# Check inode
ls -li file
stat file
```

### 2.7 File Search and Operations

```bash
# Find files
find / -name "*.log" -mtime +7 -delete
find / -type f -size +100M -exec ls -lh {} \;
find / -user krishna -type f
find / -perm -4000 2>/dev/null
find / -nouser -o -nogroup

# Locate (faster, uses database)
updatedb
locate filename

# Disk usage
du -sh /var/log/*
du -h --max-depth=1 / | sort -rh | head
df -hT                                # Filesystem usage
df -i                                 # Inode usage
```

### 2.8 Archive & Compression

```bash
# tar
tar -czvf archive.tar.gz /path        # Create compressed
tar -xzvf archive.tar.gz -C /dest     # Extract
tar -tzvf archive.tar.gz              # List contents

# Other compression
gzip file                             # .gz
bzip2 file                            # .bz2
xz file                              # .xz (best compression)

# rsync (efficient copy/sync)
rsync -avz --progress source/ dest/
rsync -avz -e ssh /local/ user@remote:/remote/
rsync -avz --delete source/ dest/     # Mirror (delete extra)
```

---


## Chapter 3 - LVM & Storage Management

### 3.1 LVM Components

| Component | Description |
|-----------|-------------|
| **PV** (Physical Volume) | Physical disk/partition added to LVM (type 8e) |
| **PE** (Physical Extent) | Smallest allocation unit (default 4MB) |
| **VG** (Volume Group) | Pool of combined PVs |
| **LV** (Logical Volume) | Virtual partition from VG |
| **LE** (Logical Extent) | Mapped to PE on the physical disk |

### 3.2 LVM Creation Workflow

```bash
# Step 1: Create physical volumes
pvcreate /dev/sdb /dev/sdc

# Step 2: Create volume group
vgcreate vg_data /dev/sdb /dev/sdc
# Or with custom PE size
vgcreate -s 8M vg_data /dev/sdb /dev/sdc

# Step 3: Create logical volume
lvcreate -L 100G -n lv_app vg_data
# Or by PE count
lvcreate -l 100 -n lv_app vg_data
# Or use all free space
lvcreate -l 100%FREE -n lv_app vg_data

# Step 4: Create filesystem
mkfs.xfs /dev/vg_data/lv_app

# Step 5: Mount
mkdir /app
mount /dev/vg_data/lv_app /app

# Step 6: Persist (/etc/fstab)
echo "/dev/vg_data/lv_app /app xfs defaults 0 2" >> /etc/fstab
mount -a
```

### 3.3 LVM Operations

```bash
# Extend LV (online)
lvextend -L +20G /dev/vg_data/lv_app
xfs_growfs /app                       # XFS
resize2fs /dev/vg_data/lv_app         # ext4
# Or both in one step
lvextend -L +20G -r /dev/vg_data/lv_app

# Add disk to VG
pvcreate /dev/sdd
vgextend vg_data /dev/sdd

# Reduce LV (ext4 only — XFS cannot shrink!)
umount /app
e2fsck -f /dev/vg_data/lv_app
resize2fs /dev/vg_data/lv_app 50G
lvreduce -L 50G /dev/vg_data/lv_app
mount /app

# Migrate data between PVs
pvmove /dev/sdb /dev/sdd
vgreduce vg_data /dev/sdb
pvremove /dev/sdb

# Snapshot
lvcreate -L 5G -s -n lv_snap /dev/vg_data/lv_app
mount -o ro /dev/vg_data/lv_snap /mnt/snap
lvconvert --merge /dev/vg_data/lv_snap    # Restore

# Remove LVM
umount /app
lvremove /dev/vg_data/lv_app
vgremove vg_data
pvremove /dev/sdb /dev/sdc
```

### 3.4 LVM Status Commands

```bash
pvs / pvdisplay / pvscan              # Physical volumes
vgs / vgdisplay / vgscan              # Volume groups
lvs / lvdisplay / lvscan              # Logical volumes
lvdisplay -m                          # Show PE mapping
vgs -v                                # Show UUID
cat /etc/lvm/lvm.conf                 # LVM config
```

### 3.5 RAID Levels

| RAID | Min Disks | Redundancy | Performance | Usable Space |
|------|-----------|-----------|-------------|--------------|
| 0 | 2 | None (striping) | High R/W | 100% |
| 1 | 2 | Mirror | Good read | 50% |
| 5 | 3 | Single parity | Good read | (n-1) disks |
| 6 | 4 | Double parity | Good read | (n-2) disks |
| 10 | 4 | Mirror + Stripe | Best | 50% |

```bash
# Create software RAID
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mkfs.xfs /dev/md0
mdadm --detail /dev/md0
cat /proc/mdstat
mdadm --detail --scan >> /etc/mdadm.conf

# Add spare disk
mdadm --manage /dev/md0 --add /dev/sdd

# Remove failed disk
mdadm --manage /dev/md0 --remove /dev/sdb
```

### 3.6 Filesystem Types

| FS | Default In | Max File | Max Volume | Features |
|----|-----------|----------|-----------|----------|
| ext4 | RHEL 6 | 16 TB | 1 EB | Journaling, backward compat |
| XFS | RHEL 7/8/9 | 8 EB | 8 EB | High performance, online grow |
| Btrfs | SUSE | 16 EB | 16 EB | Snapshots, compression |
| vfat | USB drives | 4 GB | 2 TB | Cross-platform |

```bash
# Create filesystem
mkfs.xfs /dev/sdb1
mkfs.ext4 /dev/sdb1

# Check/repair
xfs_repair /dev/sdb1                  # XFS
fsck.ext4 -y /dev/sdb1               # ext4 (unmount first!)

# Filesystem info
xfs_info /dev/sdb1
tune2fs -l /dev/sdb1                  # ext4 details
blkid                                 # Show UUID and type
```

### 3.7 NFS Configuration

**Server Side:**
```bash
yum install nfs-utils -y
mkdir /shared
chmod 777 /shared

# /etc/exports
/shared 192.168.1.0/24(rw,sync,no_root_squash)

exportfs -rav
systemctl enable --now nfs-server

# Firewall
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

**Client Side:**
```bash
yum install nfs-utils -y
showmount -e server_ip
mkdir /mnt/nfs
mount -t nfs server_ip:/shared /mnt/nfs

# Persistent (/etc/fstab)
server_ip:/shared /mnt/nfs nfs defaults 0 0
mount -a
```

### 3.8 Disk Operations

```bash
# Scan for new disks (without reboot)
echo "- - -" > /sys/class/scsi_host/host0/scan
echo "- - -" > /sys/class/scsi_host/host1/scan
echo "- - -" > /sys/class/scsi_host/host2/scan
lsblk

# Partition with fdisk
fdisk /dev/sdb
# n → new, p → primary, t → type (8e for LVM), w → write

# Partition with parted (GPT)
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary xfs 0% 100%

# Mount options in fstab
# device  mount_point  fs_type  options  dump  fsck
/dev/sdb1  /data  xfs  defaults,noatime  0  2
UUID=xxx   /data  xfs  defaults  0  2
```

---


## Chapter 4 - Network Configuration

### 4.1 Network Interface Management

```bash
# View interfaces
ip addr show
ip -4 addr                            # IPv4 only
ip link show                          # Link status
nmcli device status                   # NetworkManager
nmcli connection show                 # All connections

# Configure static IP (nmcli)
nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
nmcli con mod eth0 ipv4.gateway 192.168.1.1
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.method manual
nmcli con up eth0

# Add new connection
nmcli con add type ethernet con-name eth1 ifname eth1 ip4 10.0.0.100/24 gw4 10.0.0.1

# Config file location
/etc/sysconfig/network-scripts/ifcfg-eth0    # RHEL 7
/etc/NetworkManager/system-connections/       # RHEL 8/9
```

### 4.2 NIC Bonding / Teaming

| Mode | Name | Use Case |
|------|------|----------|
| 0 | balance-rr | Round robin load balance |
| 1 | active-backup | Failover (most common) |
| 2 | balance-xor | Hash-based distribution |
| 3 | broadcast | All slaves transmit |
| 4 | 802.3ad | LACP (switch support needed) |
| 5 | balance-tlb | Adaptive transmit balance |
| 6 | balance-alb | Adaptive load balance |

```bash
# Create bond (RHEL 7+)
nmcli con add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup,miimon=100"
nmcli con add type ethernet con-name bond0-slave1 ifname eth0 master bond0
nmcli con add type ethernet con-name bond0-slave2 ifname eth1 master bond0
nmcli con mod bond0 ipv4.addresses 192.168.1.100/24 ipv4.method manual
nmcli con up bond0

# Verify
cat /proc/net/bonding/bond0
watch -n 1 cat /proc/net/bonding/bond0

# Team (RHEL 7+ alternative to bond)
nmcli con add type team con-name team0 ifname team0 config '{"runner":{"name":"activebackup"}}'
nmcli con add type ethernet con-name team0-port1 ifname eth0 master team0
nmcli con add type ethernet con-name team0-port2 ifname eth1 master team0
teamdctl team0 state
```

### 4.3 Firewall Configuration

```bash
# Firewalld (RHEL 7+)
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.0.0.0/8 port port=22 protocol=tcp accept'
firewall-cmd --reload

# IPtables (RHEL 6 / legacy)
iptables -L -n -v                     # List rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -j DROP
iptables-save > /etc/sysconfig/iptables
service iptables restart
```

### 4.4 DNS Configuration

```bash
# Client DNS
cat /etc/resolv.conf
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.dns-search "example.com"

# Resolution order (/etc/nsswitch.conf)
hosts: files dns

# Testing
dig google.com
nslookup google.com
host google.com
getent hosts google.com

# DNS record types
# A      → IPv4 address
# AAAA   → IPv6 address
# CNAME  → Canonical name (alias)
# MX     → Mail exchange
# PTR    → Reverse DNS
# NS     → Name server
# SOA    → Start of authority
```

### 4.5 Network Diagnostics

```bash
# Connectivity
ping -c 3 target
traceroute target
mtr target                            # Continuous traceroute

# Port checking
ss -tulnp                             # Listening ports
nc -zv host port                      # Test port
telnet host port
curl -v telnet://host:port

# Traffic capture
tcpdump -i eth0 port 80 -w out.pcap
tcpdump -i any host 192.168.1.100

# Bandwidth
iperf3 -s                             # Server
iperf3 -c server_ip                   # Client

# ARP
ip neigh show
arp -a
arping -I eth0 192.168.1.1
```

### 4.6 Routing

```bash
ip route show
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0
ip route del 10.0.0.0/8
ip route add default via 192.168.1.1

# Persistent route
nmcli con mod eth0 +ipv4.routes "10.0.0.0/8 192.168.1.1"

# Enable IP forwarding
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.d/99-forward.conf
sysctl --system
```

---

## Chapter 5 - Boot Process & Services

### 5.1 Linux Boot Sequence

```
1. BIOS/UEFI   → POST (hardware check)
2. MBR/GPT     → Bootloader location
3. GRUB2        → Loads kernel + initramfs
4. Kernel       → Initializes hardware, mounts root
5. initramfs    → Loads drivers for root filesystem
6. systemd      → PID 1, starts services based on target
7. Target       → multi-user.target or graphical.target
```

### 5.2 Run Levels / Systemd Targets

| Target | SysVinit | Purpose |
|--------|----------|---------|
| poweroff.target | 0 | Shutdown |
| rescue.target | 1 | Single-user mode |
| multi-user.target | 3 | Multi-user CLI |
| graphical.target | 5 | GUI |
| reboot.target | 6 | Reboot |
| emergency.target | — | Minimal emergency shell |

```bash
# Check/change default
systemctl get-default
systemctl set-default multi-user.target

# Switch target
systemctl isolate rescue.target
```

### 5.3 GRUB2 Management

```bash
# Config files
/etc/default/grub                     # GRUB settings
/boot/grub2/grub.cfg                  # Generated config (don't edit)
/boot/grub2/grubenv                   # Environment

# Rebuild GRUB config
grub2-mkconfig -o /boot/grub2/grub.cfg

# Install GRUB
grub2-install /dev/sda

# List kernels
awk -F\' '/menuentry / {print $2}' /boot/grub2/grub.cfg
grub2-editenv list
```

### 5.4 Reset Root Password

```
1. Reboot → Press 'e' at GRUB menu
2. Find line starting with 'linux' or 'linux16'
3. Append: rd.break
4. Press Ctrl+X to boot
5. At switch_root prompt:
   mount -o remount,rw /sysroot
   chroot /sysroot
   passwd root
   touch /.autorelabel      (if SELinux enabled)
   exit
   reboot
```

### 5.5 Systemd Service Management

```bash
# Service lifecycle
systemctl start|stop|restart|reload <service>
systemctl enable|disable <service>
systemctl enable --now <service>      # Enable + start
systemctl mask|unmask <service>       # Prevent starting
systemctl status <service>

# List services
systemctl list-units --type=service
systemctl list-units --state=failed
systemctl list-unit-files --type=service

# Dependencies
systemctl list-dependencies <service>
systemctl list-dependencies --reverse <service>
```

### 5.6 Create Custom Service

```bash
cat > /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Application
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
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now myapp
```

### 5.7 Cron & Scheduled Tasks

```bash
# Cron format: MIN HOUR DOM MON DOW COMMAND
crontab -e                            # Edit user cron
crontab -l                            # List
crontab -u username -l                # List for specific user

# Examples
0 2 * * * /opt/scripts/backup.sh      # Daily 2 AM
*/5 * * * * /opt/scripts/check.sh     # Every 5 minutes
0 0 * * 0 /opt/scripts/weekly.sh      # Every Sunday midnight
0 9 1 * * /opt/scripts/monthly.sh     # 1st of each month 9 AM

# System cron directories
/etc/cron.d/                          # Drop-in cron files
/etc/cron.daily/                      # Daily scripts
/etc/cron.hourly/                     # Hourly scripts
/etc/cron.weekly/                     # Weekly scripts
/etc/cron.monthly/                    # Monthly scripts

# Restrict cron access
/etc/cron.allow                       # Only listed users can use
/etc/cron.deny                        # Listed users denied

# One-time task
at 2:00 AM tomorrow
> /opt/scripts/migrate.sh
> Ctrl+D
atq                                   # List pending
atrm <job_id>                         # Remove
```

### 5.8 Journal & Log Management

```bash
# Journalctl
journalctl -u <service> -f            # Follow service logs
journalctl -b                         # Current boot
journalctl -b -1                      # Previous boot
journalctl -p err                     # Errors only
journalctl --since "1 hour ago"
journalctl --disk-usage
journalctl --vacuum-size=500M         # Cleanup

# Make persistent
mkdir -p /var/log/journal
systemctl restart systemd-journald

# Syslog
/var/log/messages                     # General system logs
/var/log/secure                       # Auth/security
/var/log/boot.log                     # Boot messages
/var/log/cron                         # Cron logs
/var/log/dmesg                        # Kernel ring buffer
```

---


## Chapter 6 - Package Management

### 6.1 RPM (Red Hat Package Manager)

```bash
# Install/Remove
rpm -ivh package.rpm                  # Install with verbose + hash
rpm -Uvh package.rpm                  # Upgrade
rpm -e package_name                   # Remove
rpm -ivh --force package.rpm          # Force install
rpm -ivh --nodeps package.rpm         # Skip dependencies

# Query
rpm -qa                               # All installed packages
rpm -qa | grep httpd                  # Search
rpm -qa --last | head -20             # Recently installed
rpm -qi package_name                  # Package info
rpm -ql package_name                  # List files in package
rpm -qf /path/to/file                 # Which package owns file
rpm -qd package_name                  # Documentation files
rpm -qc package_name                  # Config files
rpm -qR package_name                  # Dependencies

# Verify
rpm -V package_name                   # Verify integrity
rpm -Va                               # Verify all packages
rpm -K package.rpm                    # Verify signature

# Extract without install
rpm2cpio package.rpm | cpio -idmv
```

### 6.2 YUM / DNF

```bash
# Install/Remove
yum install package -y
yum remove package
yum update                            # Update all
yum update package                    # Update specific
yum downgrade package                 # Downgrade

# Search/Info
yum search keyword
yum info package
yum provides /path/to/file
yum list installed
yum list available
yum deplist package                   # Dependencies
yum history                           # Transaction history
yum history undo <id>                 # Undo transaction

# Repository management
yum repolist all
yum-config-manager --enable repo_name
yum-config-manager --disable repo_name
yum clean all
yum makecache

# Groups
yum grouplist
yum groupinstall "Development Tools"

# Version lock
yum install yum-plugin-versionlock
yum versionlock add httpd
yum versionlock list
yum versionlock delete httpd

# Security updates
yum updateinfo list security
yum update --security

# Local repo creation
mkdir /opt/repo
cp *.rpm /opt/repo/
createrepo /opt/repo
cat > /etc/yum.repos.d/local.repo << 'EOF'
[localrepo]
name=Local Repository
baseurl=file:///opt/repo
enabled=1
gpgcheck=0
EOF
```

### 6.3 DNF (RHEL 8/9)

```bash
# Same syntax as yum (plus)
dnf module list                       # Module streams
dnf module enable php:8.1
dnf module install php:8.1
dnf module reset php

# Automatic updates
dnf install dnf-automatic
systemctl enable --now dnf-automatic-install.timer
```

---

## Chapter 7 - Web Services

### 7.1 Apache HTTP Server

```bash
# Install
yum install httpd -y
systemctl enable --now httpd

# Configuration
/etc/httpd/conf/httpd.conf            # Main config
/etc/httpd/conf.d/                    # Additional configs
/var/www/html/                        # Document root
/var/log/httpd/                       # Logs

# Virtual Host
cat > /etc/httpd/conf.d/mysite.conf << 'EOF'
<VirtualHost *:80>
    ServerName www.example.com
    ServerAlias example.com
    DocumentRoot /var/www/mysite
    ErrorLog /var/log/httpd/mysite-error.log
    CustomLog /var/log/httpd/mysite-access.log combined
    <Directory /var/www/mysite>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

# SSL/TLS
yum install mod_ssl -y
# Configure in /etc/httpd/conf.d/ssl.conf
SSLCertificateFile /etc/pki/tls/certs/server.crt
SSLCertificateKeyFile /etc/pki/tls/private/server.key

# Test config and restart
httpd -t                              # Syntax check
apachectl configtest
systemctl restart httpd

# Firewall
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

### 7.2 Nginx

```bash
# Install
yum install nginx -y
systemctl enable --now nginx

# Configuration
/etc/nginx/nginx.conf                 # Main config
/etc/nginx/conf.d/                    # Server blocks

# Server block
cat > /etc/nginx/conf.d/mysite.conf << 'EOF'
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/mysite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/mysite-access.log;
    error_log /var/log/nginx/mysite-error.log;
}
EOF

nginx -t                              # Test config
systemctl reload nginx
```

---

## Chapter 8 - Samba & File Sharing

### 8.1 Samba Server Configuration

```bash
# Install
yum install samba samba-client samba-common -y

# Important files
/etc/samba/smb.conf                   # Main config
Port: 137 (name), 138 (data), 139 (connection), 445 (auth)

# Create share
mkdir /samba/share
chmod 777 /samba/share
chcon -t samba_share_t /samba/share   # SELinux

# Create Samba user
useradd smbuser
smbpasswd -a smbuser

# Configure /etc/samba/smb.conf
[myshare]
    comment = Shared Directory
    path = /samba/share
    browseable = yes
    writable = yes
    valid users = smbuser
    create mask = 0644
    directory mask = 0755

# Verify and start
testparm                              # Check config syntax
systemctl enable --now smb nmb

# Firewall
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
```

### 8.2 Samba Client

```bash
# Install
yum install samba-client cifs-utils -y

# List shares
smbclient -L //server_ip -U smbuser

# Access share
smbclient //server_ip/myshare -U smbuser

# Mount
mount -t cifs //server_ip/myshare /mnt/samba -o username=smbuser,password=pass

# Persistent mount (/etc/fstab)
//server_ip/myshare /mnt/samba cifs credentials=/root/.smbcreds 0 0

# Credentials file (/root/.smbcreds)
username=smbuser
password=mypassword
chmod 600 /root/.smbcreds
```

---

## Chapter 9 - SSH & Remote Access

### 9.1 SSH Configuration

```bash
# Server config: /etc/ssh/sshd_config
Port 22
PermitRootLogin no
PasswordAuthentication no             # Key-only
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers krishna admin
X11Forwarding no
Banner /etc/ssh/banner

systemctl restart sshd
```

### 9.2 SSH Key Management

```bash
# Generate keys
ssh-keygen -t ed25519 -C "krishna@server"
ssh-keygen -t rsa -b 4096

# Copy to remote
ssh-copy-id user@remote_host

# Manual copy
cat ~/.ssh/id_ed25519.pub >> remote:~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519

# SSH agent
eval $(ssh-agent)
ssh-add ~/.ssh/id_ed25519
ssh-add -l                            # List loaded keys
```

### 9.3 SSH Tunneling

```bash
# Local port forward (access remote:3306 via localhost:9906)
ssh -L 9906:localhost:3306 user@remote

# Remote port forward (expose local:8080 on remote:9090)
ssh -R 9090:localhost:8080 user@remote

# Dynamic SOCKS proxy
ssh -D 1080 user@remote

# Jump host / bastion
ssh -J bastion_user@bastion target_user@target
# Or in ~/.ssh/config:
Host target
    HostName 10.0.0.5
    User admin
    ProxyJump bastion
```

### 9.4 SCP & SFTP

```bash
# SCP
scp file.txt user@remote:/path/
scp -r /local/dir user@remote:/path/
scp user@remote:/path/file.txt /local/

# SFTP
sftp user@remote
sftp> put localfile
sftp> get remotefile
sftp> ls
sftp> exit
```

---


## Chapter 10 - SELinux & Security

### 10.1 SELinux Modes

| Mode | Behavior |
|------|----------|
| Enforcing | Enforces policy, denies and logs |
| Permissive | Logs but does not deny |
| Disabled | SELinux completely off |

```bash
# Check status
getenforce
sestatus

# Change mode (runtime)
setenforce 0                          # Permissive
setenforce 1                          # Enforcing

# Permanent: /etc/selinux/config
SELINUX=enforcing
SELINUXTYPE=targeted
```

### 10.2 SELinux Contexts & Labels

```bash
# View context
ls -Z /var/www/html
ps auxZ | grep httpd

# Change context (temporary)
chcon -t httpd_sys_content_t /web/html

# Change context (permanent)
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
restorecon -Rv /web

# Restore default context
restorecon -Rv /path
```

### 10.3 SELinux Ports & Booleans

```bash
# Ports
semanage port -l | grep http
semanage port -a -t http_port_t -p tcp 8443

# Booleans
getsebool -a | grep httpd
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_connect_db on
```

### 10.4 SELinux Troubleshooting

```bash
# Check audit log for denials
ausearch -m avc --start recent
grep "denied" /var/log/audit/audit.log

# Generate fix suggestions
sealert -a /var/log/audit/audit.log
audit2why < /var/log/audit/audit.log
audit2allow -a                        # Generate policy module

# Install troubleshooting tools
yum install setroubleshoot-server -y
```

### 10.5 Security Hardening Checklist

```bash
# SSH
PermitRootLogin no
PasswordAuthentication no
MaxAuthTries 3

# Firewall
firewall-cmd --permanent --remove-service=cockpit
firewall-cmd --reload

# Disable unused services
systemctl disable --now cups bluetooth avahi-daemon

# Kernel parameters
cat >> /etc/sysctl.d/99-security.conf << 'EOF'
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.tcp_syncookies = 1
kernel.randomize_va_space = 2
EOF
sysctl --system

# File permissions
chmod 600 /etc/shadow
chmod 644 /etc/passwd
chmod 000 /etc/shadow-

# Audit
systemctl enable --now auditd
auditctl -w /etc/passwd -p wa -k identity
auditctl -w /etc/sudoers -p wa -k sudoers
```

---

## Chapter 11 - Performance & Monitoring

### 11.1 CPU Monitoring

```bash
top -bn1 | head -20
mpstat -P ALL 1 5                     # Per-CPU stats
sar -u 1 5                            # CPU history
uptime                                # Load average
nproc                                 # CPU count
lscpu                                 # CPU info
pidstat 1 5                           # Per-process CPU
```

### 11.2 Memory Monitoring

```bash
free -h
vmstat 1 5
sar -r 1 5                            # Memory history
cat /proc/meminfo
slabtop                               # Kernel slab cache
pmap -x <PID>                         # Process memory map

# Key: Available = Free + Reclaimable (buffers/cache)
```

### 11.3 Disk I/O Monitoring

```bash
iostat -xz 1 5
# %util > 80% = saturated
# await > 10ms = high latency
iotop -oP                             # Per-process I/O
sar -d 1 5                            # Disk history

# I/O scheduler
cat /sys/block/sda/queue/scheduler
echo "mq-deadline" > /sys/block/sda/queue/scheduler
```

### 11.4 Network Monitoring

```bash
iftop                                 # Bandwidth per connection
nload                                 # Real-time bandwidth
vnstat                                # Historical traffic
sar -n DEV 1 5                        # Interface stats
ss -s                                 # Socket summary
```

### 11.5 Process Management

```bash
ps aux --sort=-%cpu | head -10        # Top CPU consumers
ps aux --sort=-%mem | head -10        # Top memory consumers
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu
top -H -p <PID>                       # Per-thread view

# Kill
kill <PID>                            # SIGTERM (graceful)
kill -9 <PID>                         # SIGKILL (force)
pkill -f process_name
killall process_name

# Priority
nice -n 10 command                    # Start with lower priority
renice -n 5 -p <PID>                  # Change priority

# Background/foreground
command &                             # Run in background
jobs                                  # List background jobs
fg %1                                 # Bring to foreground
bg %1                                 # Continue in background
nohup command &                       # Survive logout
```

### 11.6 System Activity Reporter (SAR)

```bash
# Install
yum install sysstat -y
systemctl enable --now sysstat

# Usage
sar -u                                # CPU (today)
sar -r                                # Memory (today)
sar -b                                # I/O (today)
sar -n DEV                            # Network (today)
sar -q                                # Load average

# Historical data
sar -u -f /var/log/sa/sa15            # CPU on 15th
sar -r -s 08:00:00 -e 17:00:00       # Memory 8am-5pm
```

### 11.7 Troubleshooting Tools

```bash
strace -p <PID>                       # System calls
ltrace <command>                      # Library calls
lsof -p <PID>                        # Open files
lsof -i :8080                        # What's using port
fuser -vm /mount/point               # Who's using mount

dmesg | tail                          # Kernel messages
dmesg | grep -i error
journalctl -xe                        # Recent errors
```

---

## Chapter 12 - Backup & Recovery

### 12.1 Backup Tools

```bash
# tar
tar -czvf /backup/full_$(date +%Y%m%d).tar.gz /opt/app
tar -xzvf backup.tar.gz -C /restore/

# rsync (incremental)
rsync -avz --delete /source/ /backup/
rsync -avz -e ssh /data/ user@backup-server:/backup/

# dd (disk image)
dd if=/dev/sda of=/backup/disk.img bs=4M status=progress
dd if=/backup/disk.img of=/dev/sda bs=4M

# dump/restore (ext4)
dump -0 -f /backup/root.dump /
restore -rf /backup/root.dump
```

### 12.2 Backup Strategy

```
Full Backup     → Every Sunday (complete copy)
Incremental     → Mon-Sat (only changed since last backup)
Differential    → Mon-Sat (changed since last full)

Retention Policy:
- Daily backups: 7 days
- Weekly backups: 4 weeks
- Monthly backups: 12 months
```

### 12.3 Automated Backup Script

```bash
#!/bin/bash
# Prepared by Krishna Yada

BACKUP_DIR="/backup"
SOURCE="/opt/app"
RETENTION=7
DATE=$(date +%Y%m%d_%H%M%S)
LOG="/var/log/backup.log"

echo "[$(date)] Starting backup..." >> $LOG
tar -czf "${BACKUP_DIR}/app_${DATE}.tar.gz" "$SOURCE" 2>> $LOG

if [ $? -eq 0 ]; then
    echo "[$(date)] Backup successful: app_${DATE}.tar.gz" >> $LOG
else
    echo "[$(date)] ERROR: Backup failed!" >> $LOG
    exit 1
fi

# Cleanup old backups
find "$BACKUP_DIR" -name "app_*.tar.gz" -mtime +$RETENTION -delete
echo "[$(date)] Cleanup done. Retained last $RETENTION days." >> $LOG
```

### 12.4 Recovery Scenarios

```bash
# Recover deleted file (if process still holds it open)
lsof | grep deleted
cp /proc/<PID>/fd/<FD> /path/to/recovered_file

# Recover LVM from snapshot
lvconvert --merge /dev/vg_data/lv_snap

# Recover GRUB
# Boot from rescue media
mount /dev/sda2 /mnt/sysroot
mount /dev/sda1 /mnt/sysroot/boot
chroot /mnt/sysroot
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg

# Recover from VG backup
vgcfgrestore --list vg_data
vgcfgrestore -f /etc/lvm/archive/vg_data_00001.vg vg_data
vgchange -ay vg_data
```

---

## Chapter 13 - Real-World Scenarios

### 13.1 Server Handover Checklist

```
1. Document server details (hostname, IP, OS, role)
2. List running services and their status
3. Document cron jobs and scheduled tasks
4. Note monitoring alerts and thresholds
5. List mount points and storage configuration
6. Document network configuration and bonds
7. List user accounts and sudo access
8. Mail to shift reliever with open tickets
```

### 13.2 Server Decommission Process

```
1. Inform monitoring team to suppress alerts
2. Stop applications and databases
3. Stop cluster services if any
4. Unmount filesystems
5. Wait observation period (1 week)
6. Request network team to release ports
7. Inform datacenter to remove cables
8. Update CMDB and documentation
```

### 13.3 Patching Workflow

```bash
# Pre-checks
uname -r                              # Current kernel
cat /etc/os-release                   # OS version
df -h                                 # Disk space
systemctl list-units --state=failed   # Failed services

# Apply patches
yum update -y                         # Update all
# Or security only:
yum update --security -y

# Post-checks
needs-restarting -r                   # Reboot needed?
needs-restarting -s                   # Services needing restart
rpm -qa --last | head -10             # Verify updates
```

### 13.4 Common Production Issues

| Issue | Quick Fix |
|-------|-----------|
| Disk full | `du -sh /* \| sort -rh`, truncate logs, extend LV |
| High CPU | `top`, identify process, check for runaway cron |
| High memory | `free -h`, check for leaks, OOM killer |
| Service down | `systemctl status`, `journalctl -u`, restart |
| Cannot SSH | Check firewall, sshd status, disk space |
| Boot failure | GRUB rescue, rd.break, fsck |
| Network down | `ip addr`, `nmcli`, check cable/bond |

### 13.5 Incident Response Template

```
1. Acknowledge alert
2. Assess severity (P1/P2/P3/P4)
3. Initial diagnosis (logs, metrics, health)
4. Engage relevant teams if needed
5. Apply fix
6. Verify recovery
7. Update ticket with RCA
8. Post-incident review
```

---

<div align="center">

## 📌 Quick Command Reference Card

| Task | Command |
|------|---------|
| System info | `hostnamectl`, `uname -a`, `cat /etc/os-release` |
| Disk usage | `df -hT`, `du -sh /*`, `lsblk` |
| Memory | `free -h`, `vmstat 1 5` |
| CPU | `top`, `mpstat -P ALL 1 3` |
| Network | `ip addr`, `ss -tulnp`, `nmcli` |
| Services | `systemctl status`, `journalctl -u` |
| Users | `id`, `who`, `last`, `passwd -S` |
| Logs | `journalctl -xe`, `tail -f /var/log/messages` |
| LVM | `pvs`, `vgs`, `lvs`, `lvextend -r` |
| Firewall | `firewall-cmd --list-all` |
| SELinux | `getenforce`, `sestatus`, `restorecon` |

---

**Prepared by [Krishna Yada](https://github.com/yadakrishna245)**

*Senior System Administrator | 8+ Years Enterprise Experience | 4000+ Servers Managed*

*Based on real-world production experience across RHEL 7/8/9 and Ubuntu environments*

⭐ **Star this repo if it helped you!** ⭐

</div>


## Chapter 10 - SELinux & Security

### 10.1 SELinux Modes

| Mode | Behavior |
|------|----------|
| Enforcing | Enforces policy, denies and logs |
| Permissive | Logs but does not deny |
| Disabled | SELinux completely off |

```bash
getenforce
sestatus
setenforce 0                          # Permissive (temp)
setenforce 1                          # Enforcing (temp)
# Permanent: /etc/selinux/config → SELINUX=enforcing
```

### 10.2 SELinux Context Management

```bash
ls -Z /var/www/html
chcon -t httpd_sys_content_t /web/html          # Temporary
semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"  # Permanent
restorecon -Rv /web

# Ports
semanage port -l | grep http
semanage port -a -t http_port_t -p tcp 8443

# Booleans
getsebool -a | grep httpd
setsebool -P httpd_can_network_connect on

# Troubleshoot
ausearch -m avc --start recent
sealert -a /var/log/audit/audit.log
```

### 10.3 Security Hardening

```bash
# Disable unused services
systemctl disable --now cups bluetooth avahi-daemon

# Kernel parameters (/etc/sysctl.d/99-security.conf)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.tcp_syncookies = 1
kernel.randomize_va_space = 2

# Auditd
auditctl -w /etc/passwd -p wa -k identity
auditctl -w /etc/sudoers -p wa -k sudoers
ausearch -k identity --start today
aureport --auth
```

---

## Chapter 11 - Performance & Monitoring

### 11.1 Key Commands

```bash
# CPU
top -bn1 | head -20
mpstat -P ALL 1 5
sar -u 1 5
pidstat 1 5

# Memory
free -h
vmstat 1 5
sar -r 1 5
pmap -x <PID>

# Disk I/O
iostat -xz 1 5          # %util > 80% = saturated
iotop -oP
sar -d 1 5

# Network
iftop / nload / vnstat
sar -n DEV 1 5
ss -s

# Process
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10
strace -p <PID>
lsof -p <PID>
```

### 11.2 SAR (System Activity Reporter)

```bash
yum install sysstat -y
systemctl enable --now sysstat
sar -u -f /var/log/sa/sa15            # CPU on 15th
sar -r -s 08:00:00 -e 17:00:00       # Memory 8am-5pm
```

---

## Chapter 12 - Backup & Recovery

```bash
# tar
tar -czvf /backup/full_$(date +%Y%m%d).tar.gz /opt/app
tar -xzvf backup.tar.gz -C /restore/

# rsync
rsync -avz --delete /source/ /backup/
rsync -avz -e ssh /data/ user@backup:/backup/

# dd (disk image)
dd if=/dev/sda of=/backup/disk.img bs=4M status=progress

# Recover deleted file (still open)
lsof | grep deleted
cp /proc/<PID>/fd/<FD> /recovered_file

# GRUB recovery (from rescue media)
mount /dev/sda2 /mnt/sysroot
chroot /mnt/sysroot
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## Chapter 13 - Real-World Scenarios

### Common Production Issues

| Issue | Quick Fix |
|-------|-----------|
| Disk full | `du -sh /* \| sort -rh`, truncate logs, extend LV |
| High CPU | `top`, identify process, check runaway cron |
| High memory | `free -h`, check OOM, restart leaking process |
| Service down | `systemctl status`, `journalctl -u`, restart |
| Cannot SSH | Check firewall, sshd, disk space, network |
| Boot failure | GRUB rescue, `rd.break`, fsck |
| Network down | `ip addr`, `nmcli`, check bond/cable |

### Patching Workflow

```bash
# Pre-check
uname -r; df -h; systemctl list-units --state=failed

# Patch
yum update -y

# Post-check
needs-restarting -r                   # Reboot needed?
rpm -qa --last | head -10
```

### Server Decommission

```
1. Suppress monitoring alerts
2. Stop applications/databases
3. Unmount filesystems
4. Wait 1 week observation
5. Release network ports
6. Datacenter removes cables
7. Update CMDB
```

---

<div align="center">

**Prepared by [Krishna Yada](https://github.com/yadakrishna245)**

*Senior System Administrator | 8+ Years | 4000+ Production Servers*

*Real-world reference from RHEL 7/8/9 and Ubuntu environments*

⭐ **Star this repo if it helped you!** ⭐

</div>
