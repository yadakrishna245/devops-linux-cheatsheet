# Day 4 — Disk & Storage (LVM) 💾

## LinkedIn Post:

```
💾 [Day 4/15] DevOps Cheat Sheet: Disk & Storage (LVM)

2AM. Pager goes off.
"/opt is at 100%. App is DOWN."

If you don't know LVM, you're rebuilding the server.
If you do — it's a 2-minute fix.

━━━━━━━━━━━━━━━━━

# The LVM workflow that saves you every time:

# 1. Create physical volume
pvcreate /dev/sdb

# 2. Create/extend volume group
vgcreate vg_data /dev/sdb
vgextend vg_data /dev/sdc      # Add more disks later

# 3. Create logical volume
lvcreate -L 50G -n lv_app vg_data

# 4. THE LIFESAVER — extend without downtime
lvextend -L +20G /dev/vg_data/lv_app
resize2fs /dev/vg_data/lv_app    # ext4
xfs_growfs /dev/vg_data/lv_app   # xfs

# 5. Mount permanently
echo "/dev/vg_data/lv_app /opt/app xfs defaults 0 0" >> /etc/fstab
mount -a

━━━━━━━━━━━━━━━━━

# NFS — share storage across servers:

# Server
echo "/shared 192.168.1.0/24(rw,sync,no_root_squash)" >> /etc/exports
exportfs -rav

# Client
mount -t nfs server:/shared /mnt/nfs

━━━━━━━━━━━━━━━━━

💡 Pro tip: Always use LVM on production servers.
Raw partitions = future pain. LVM = flexibility.

The 5 extra minutes during setup saves hours during incidents.

♻️ Repost — every Linux admin needs LVM skills
#Linux #DevOps #Storage #LVM #SysAdmin #SRE
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ 400+ commands covering 16 DevOps topics!

Tomorrow: Networking & Firewall — firewalld vs iptables explained 🌐
```
