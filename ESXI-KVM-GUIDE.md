<div align="center">

# 🖥️ VMware ESXi & KVM — Complete Guide

**From Beginner to Advanced | Installation on Blade Servers | Interview Q&A**

*Prepared by Krishna Yada | Senior System Administrator | 8+ Years | 4000+ Servers*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Krishna_Yada-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/krishna-yada-8a8441239)
[![GitHub](https://img.shields.io/badge/GitHub-yadakrishna245-black?style=flat-square&logo=github)](https://github.com/yadakrishna245)

</div>

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [VMware ESXi — Introduction](#1-vmware-esxi--introduction) |
| 2 | [ESXi Installation on Blade Server](#2-esxi-installation-on-blade-server) |
| 3 | [ESXi Administration](#3-esxi-administration) |
| 4 | [vCenter & vSphere Management](#4-vcenter--vsphere-management) |
| 5 | [ESXi Networking](#5-esxi-networking) |
| 6 | [ESXi Storage](#6-esxi-storage) |
| 7 | [ESXi High Availability & Clustering](#7-esxi-high-availability--clustering) |
| 8 | [KVM — Introduction](#8-kvm--introduction) |
| 9 | [KVM Installation on Bare Metal / Blade Server](#9-kvm-installation-on-bare-metal--blade-server) |
| 10 | [KVM Administration](#10-kvm-administration) |
| 11 | [KVM Networking](#11-kvm-networking) |
| 12 | [KVM Storage](#12-kvm-storage) |
| 13 | [KVM Advanced Topics](#13-kvm-advanced-topics) |
| 14 | [ESXi vs KVM Comparison](#14-esxi-vs-kvm-comparison) |
| 15 | [Interview Questions & Answers](#15-interview-questions--answers) |

---

## 1. VMware ESXi — Introduction

### 1.1 What is ESXi?

- **Type-1 (bare-metal) hypervisor** — installs directly on hardware, no OS underneath
- Part of VMware vSphere suite
- Provides server virtualization — run multiple VMs on one physical host
- Commercial product by Broadcom (formerly VMware)

### 1.2 ESXi Architecture

```
┌─────────────────────────────────────────────┐
│              Virtual Machines                │
│  ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐   │
│  │VM 1│  │VM 2│  │VM 3│  │VM 4│  │VM 5│   │
│  └────┘  └────┘  └────┘  └────┘  └────┘   │
├─────────────────────────────────────────────┤
│            VMkernel (ESXi Hypervisor)        │
├─────────────────────────────────────────────┤
│         Physical Hardware (CPU/RAM/NIC/HBA)  │
└─────────────────────────────────────────────┘
```

### 1.3 Key Components

| Component | Purpose |
|-----------|---------|
| **ESXi Host** | Bare-metal hypervisor on physical server |
| **vCenter Server** | Centralized management for multiple hosts |
| **vSphere Client** | Web UI to manage ESXi/vCenter |
| **VMFS** | VMware filesystem for VM storage |
| **vMotion** | Live migration of VMs between hosts |
| **HA** | High Availability — auto-restart VMs on failure |
| **DRS** | Distributed Resource Scheduler — load balance |
| **vDS** | Virtual Distributed Switch |

### 1.4 ESXi Versions & Requirements

| Version | Min RAM | Min CPU | Storage |
|---------|---------|---------|---------|
| ESXi 7.0 | 4 GB | 2 cores (64-bit) | 32 GB boot |
| ESXi 8.0 | 8 GB | 2 cores (64-bit) | 32 GB boot |

**Hardware requirements:**
- 64-bit x86 CPU with VT-x/AMD-V
- NX/XD bit enabled
- Minimum 2 physical CPU cores
- At least one Gigabit NIC (on HCL)
- Hardware RAID controller recommended

---

## 2. ESXi Installation on Blade Server

### 2.1 Pre-Installation (Blade Server Setup)

**Common blade servers:** HP ProLiant BL460c, Dell PowerEdge M-series, Cisco UCS B-series

#### Step 1: Access Blade Management Console

```
HP:    iLO (Integrated Lights-Out)    → https://<iLO-IP>
Dell:  iDRAC (Integrated Dell Remote Access Controller) → https://<iDRAC-IP>
Cisco: CIMC (Cisco IMC) or UCS Manager → https://<CIMC-IP>
```

#### Step 2: Configure BIOS

1. Access BIOS via remote console (iLO/iDRAC)
2. Enable settings:
   - **Intel VT-x / AMD-V** (Virtualization Technology)
   - **Intel VT-d** (Directed I/O for passthrough)
   - **Execute Disable (NX/XD bit)**
   - **Hyper-Threading** (recommended ON)
   - **Boot Mode: UEFI** (recommended for ESXi 7+)
   - **NUMA** (enable for multi-socket)

#### Step 3: Configure RAID

```
RAID 1   → Boot drive (2 disks mirrored) — for ESXi OS
RAID 5/6 → Datastore (if using local storage)
RAID 10  → High-performance datastore
```

Using HP Smart Array / Dell PERC:
1. Enter RAID controller config (F8 at boot or via iLO)
2. Create RAID 1 with 2x 300GB SAS → for ESXi boot
3. Create RAID 5 with remaining disks → for local datastore

#### Step 4: Network Configuration

- Configure blade network adapters in chassis management
- Assign uplink ports and VLAN trunks
- Ensure management VLAN is accessible

### 2.2 ESXi Installation Steps

#### Method 1: Interactive Install (via Remote Console)

```
1. Mount ESXi ISO via iLO/iDRAC virtual media
2. Set boot order → CD/DVD first
3. Reboot blade
4. ESXi installer loads:
   - Press Enter to start
   - Accept EULA (F11)
   - Select disk (RAID 1 LUN for boot)
   - Choose keyboard layout
   - Set root password (8+ chars, complexity required)
   - Confirm install (F11)
   - Remove media and reboot
5. After reboot → ESXi DCUI console shows IP/hostname
6. Press F2 → Configure Management Network:
   - Set static IP
   - Set VLAN ID (if tagged)
   - Set DNS and hostname
   - Restart management agents
```

#### Method 2: Scripted/Kickstart Install (Automated)

```bash
# kickstart.cfg (on HTTP/FTP server)
vmaccepteula
rootpw MyP@ss2024!
clearpart --firstdisk=local --overwritevmfs
install --firstdisk=local --overwritevmfs
network --bootproto=static --device=vmnic0 --ip=192.168.1.50 --netmask=255.255.255.0 --gateway=192.168.1.1 --nameserver=8.8.8.8 --hostname=esxi01.lab.local

# Boot with kickstart
# Append to boot options:
ks=http://192.168.1.10/kickstart/esxi-ks.cfg
```

#### Method 3: Auto Deploy (PXE Boot — Large Scale)

```
1. Set up DHCP with Option 66 (TFTP server) and Option 67 (boot file)
2. Configure TFTP server with ESXi boot files
3. Set up Auto Deploy rules in vCenter
4. Blade PXE boots → downloads ESXi image → installs
```

### 2.3 Post-Installation Tasks

```
1. Access ESXi Host Client: https://<ESXi-IP>/ui
2. Apply license key (Host → Manage → Licensing)
3. Configure NTP: Host → Manage → System → Time
4. Enable SSH: Host → Manage → Services → TSM-SSH → Start
5. Add to vCenter (if available)
6. Configure networking (vSwitches, port groups)
7. Configure storage (local/SAN/NAS)
8. Set up syslog: esxcli system syslog config set --loghost=udp://syslog:514
```

---

## 3. ESXi Administration

### 3.1 ESXCLI Commands (SSH)

```bash
# Host info
esxcli system version get
esxcli system hostname get
esxcli hardware cpu global get
esxcli hardware memory get
esxcli hardware platform get

# Network
esxcli network ip interface list
esxcli network ip interface ipv4 get
esxcli network nic list
esxcli network vswitch standard list
esxcli network firewall ruleset list

# Storage
esxcli storage filesystem list
esxcli storage core device list
esxcli storage vmfs extent list
esxcli storage nmp path list

# VM management
vim-cmd vmsvc/getallvms                # List all VMs
vim-cmd vmsvc/power.on <vmid>          # Power on
vim-cmd vmsvc/power.off <vmid>         # Power off
vim-cmd vmsvc/power.reboot <vmid>      # Reboot
vim-cmd vmsvc/get.summary <vmid>       # VM summary
vim-cmd vmsvc/snapshot.create <vmid> "snap1" "description"
vim-cmd vmsvc/snapshot.removeall <vmid>

# Maintenance mode
esxcli system maintenanceMode set --enable true
esxcli system maintenanceMode set --enable false

# NTP
esxcli system ntp set --server=pool.ntp.org
esxcli system ntp set --enabled=true

# Patching
esxcli software profile get
esxcli software vib install -d /vmfs/volumes/datastore1/patch.zip
esxcli software vib list | grep -i <name>
```

### 3.2 VM Operations

```bash
# Create VM (via CLI)
vim-cmd solo/registervm /vmfs/volumes/datastore1/myvm/myvm.vmx

# Clone (PowerCLI)
New-VM -Name "clone-vm" -VM "source-vm" -Datastore "ds1" -VMHost "esxi01"

# Snapshot management
vim-cmd vmsvc/snapshot.create <vmid> "before-patch" "pre-patch snapshot"
vim-cmd vmsvc/snapshot.get <vmid>
vim-cmd vmsvc/snapshot.revert <vmid> <snap-id>
vim-cmd vmsvc/snapshot.removeall <vmid>

# Resource allocation
# CPU: Shares (Low/Normal/High), Reservation, Limit
# Memory: Shares, Reservation, Limit, Balloon driver
```

### 3.3 ESXi Logs

| Log File | Location | Purpose |
|----------|----------|---------|
| vmkernel.log | /var/log/ | Kernel, drivers, storage, network |
| hostd.log | /var/log/ | Host management (vSphere client ops) |
| vpxa.log | /var/log/ | vCenter agent on host |
| vobd.log | /var/log/ | Events and alerts |
| fdm.log | /var/log/ | HA (Fault Domain Manager) |
| shell.log | /var/log/ | SSH/shell commands |
| syslog.log | /var/log/ | General system log |

```bash
# View logs
tail -f /var/log/vmkernel.log
grep -i error /var/log/hostd.log
cat /var/log/boot.gz | gunzip | less
```

---


## 4. vCenter & vSphere Management

### 4.1 vCenter Server

- Centralized management for multiple ESXi hosts
- Deployed as VCSA (vCenter Server Appliance) — Linux-based VM
- Manages clusters, DRS, HA, vMotion, templates, permissions

### 4.2 vCenter Deployment

```
1. Mount VCSA ISO
2. Run installer → Stage 1: Deploy appliance
   - Target ESXi host IP
   - Appliance size (Tiny: up to 10 hosts, Small: up to 100)
   - Storage location
3. Stage 2: Configure
   - NTP, SSO domain (vsphere.local)
   - Create SSO admin password
4. Access: https://<vcenter-ip>/ui
```

### 4.3 Key vCenter Operations

```
# Datacenter → Cluster → Host → VM hierarchy

Datacenter  → Logical container for hosts
Cluster     → Group of hosts sharing resources
Resource Pool → Subdivide cluster resources
Template    → Master VM image for cloning
Content Library → Centralized template/ISO storage
```

### 4.4 PowerCLI (Automation)

```powershell
# Connect
Connect-VIServer -Server vcenter.lab.local -User admin@vsphere.local

# Get info
Get-VMHost | Select Name, ConnectionState, MemoryTotalGB
Get-VM | Select Name, PowerState, NumCpu, MemoryGB
Get-Datastore | Select Name, FreeSpaceGB, CapacityGB

# VM operations
New-VM -Name "web01" -Template "rhel8-template" -VMHost "esxi01" -Datastore "ds1"
Start-VM -VM "web01"
Stop-VM -VM "web01" -Confirm:$false
Restart-VM -VM "web01"
Remove-VM -VM "web01" -DeletePermanently

# Snapshot
New-Snapshot -VM "web01" -Name "pre-patch" -Description "before patching"
Get-Snapshot -VM "web01"
Set-VM -VM "web01" -Snapshot (Get-Snapshot -VM "web01" -Name "pre-patch")
Remove-Snapshot -Snapshot (Get-Snapshot -VM "web01" -Name "pre-patch")

# vMotion
Move-VM -VM "web01" -Destination (Get-VMHost "esxi02")

# Bulk operations
Get-VM -Tag "webservers" | Restart-VM -Confirm:$false
```

---

## 5. ESXi Networking

### 5.1 Virtual Switch Types

| Type | Scope | Management |
|------|-------|-----------|
| **vSS** (Standard Switch) | Per-host | Each host configured separately |
| **vDS** (Distributed Switch) | Cluster-wide | Managed from vCenter |

### 5.2 Network Components

```
Physical NIC (vmnic0, vmnic1...) → Uplinks
     │
vSwitch (vSS or vDS)
     │
Port Groups (Management, vMotion, VM Network, Storage)
     │
VMkernel Ports (vmk0, vmk1...) → Host services
     │
VM vNICs (attached to port groups)
```

### 5.3 Configure Standard vSwitch

```bash
# CLI
esxcli network vswitch standard add --vswitch-name=vSwitch1
esxcli network vswitch standard uplink add --uplink-name=vmnic2 --vswitch-name=vSwitch1
esxcli network vswitch standard portgroup add --portgroup-name="Production" --vswitch-name=vSwitch1
esxcli network vswitch standard portgroup set --portgroup-name="Production" --vlan-id=100
```

### 5.4 NIC Teaming & Failover

| Policy | Description |
|--------|-------------|
| Route based on originating virtual port | Default, balanced |
| Route based on IP hash | Requires EtherChannel on switch |
| Route based on source MAC | Simple, less balanced |
| Explicit failover order | Active/standby |

### 5.5 VMkernel Ports

| Service | Purpose | Typical VLAN |
|---------|---------|--------------|
| Management | ESXi host management | VLAN 10 |
| vMotion | Live VM migration | VLAN 20 |
| iSCSI/NFS | Storage traffic | VLAN 30 |
| vSAN | vSAN cluster traffic | VLAN 40 |
| Fault Tolerance | FT logging | VLAN 50 |

---

## 6. ESXi Storage

### 6.1 Storage Types

| Type | Protocol | Use Case |
|------|----------|----------|
| **VMFS** | Block (FC/iSCSI/Local) | Default VM datastore |
| **NFS** | File (NFS v3/v4.1) | Shared storage, templates |
| **vSAN** | Distributed | HCI (Hyper-converged) |
| **RDM** | Raw Device Mapping | Direct LUN to VM |
| **Local** | SATA/SAS/NVMe | Boot, non-shared |

### 6.2 VMFS Datastore Operations

```bash
# List datastores
esxcli storage filesystem list

# Create VMFS datastore (via UI preferred)
# Rescan storage
esxcli storage core adapter rescan --all

# Extend datastore
# UI: Storage → Datastore → Increase Capacity

# Thin vs Thick provisioning
# Thin: allocates on demand (over-commit possible)
# Thick Lazy Zeroed: allocates full space, zeros on first write
# Thick Eager Zeroed: allocates + zeros immediately (best for FT)
```

### 6.3 iSCSI Configuration

```bash
# Enable software iSCSI adapter
esxcli iscsi software set --enabled=true

# Add iSCSI target
esxcli iscsi adapter discovery sendtarget add --adapter=vmhba65 --address=192.168.30.100

# Rescan
esxcli storage core adapter rescan --adapter=vmhba65

# CHAP authentication
esxcli iscsi adapter auth chap set --adapter=vmhba65 --direction=uni --authname=user --secret=password --level=required
```

### 6.4 Storage Multipathing (PSA)

```bash
# Check paths
esxcli storage nmp path list
esxcli storage core path list

# Path policies
# Fixed    → Uses preferred path
# Round Robin → Rotates across paths (recommended)
# MRU      → Most Recently Used

# Set policy
esxcli storage nmp device set --device=naa.xxx --psp=VMW_PSP_RR
```

---

## 7. ESXi High Availability & Clustering

### 7.1 vSphere HA

- Monitors hosts and VMs
- Auto-restarts VMs on a failed host on surviving hosts
- Requires shared storage and vCenter

```
Settings:
- Host Monitoring: Enabled
- VM Restart Priority: High/Medium/Low
- Admission Control: Reserve % of cluster resources
- Datastore Heartbeating: Secondary check for host isolation
```

### 7.2 vMotion (Live Migration)

**Requirements:**
- Shared storage (or Storage vMotion for different datastores)
- Compatible CPUs (or EVC enabled)
- VMkernel port for vMotion on both hosts
- Gigabit+ network between hosts

### 7.3 DRS (Distributed Resource Scheduler)

| Mode | Behavior |
|------|----------|
| Manual | Recommends migrations, admin applies |
| Partially Automated | Auto place on power-on, recommend balancing |
| Fully Automated | Auto-migrates VMs to balance load |

### 7.4 vSphere Fault Tolerance (FT)

- Maintains a live shadow copy of VM on another host
- Zero downtime on host failure
- Limitations: max 8 vCPUs, thick eager zeroed disk, no snapshots

### 7.5 vSAN (Virtual SAN)

- HCI storage using local disks across cluster
- Minimum 3 hosts
- Requires SSD for caching + HDD/SSD for capacity
- Storage policies define protection level (FTT=1 means 1 failure tolerable)

---


## 8. KVM — Introduction

### 8.1 What is KVM?

- **Kernel-based Virtual Machine** — Type-1 hypervisor built into Linux kernel
- Open-source, part of mainline Linux since kernel 2.6.20
- Turns Linux into a hypervisor
- Uses QEMU for hardware emulation, libvirt for management

### 8.2 KVM Architecture

```
┌──────────────────────────────────────────────┐
│              Virtual Machines                  │
│  ┌────┐  ┌────┐  ┌────┐  ┌────┐  ┌────┐    │
│  │VM 1│  │VM 2│  │VM 3│  │VM 4│  │VM 5│    │
│  └────┘  └────┘  └────┘  └────┘  └────┘    │
├──────────────────────────────────────────────┤
│  QEMU (emulation) + libvirt (management API) │
├──────────────────────────────────────────────┤
│  KVM Module (kvm.ko + kvm_intel/kvm_amd.ko)  │
├──────────────────────────────────────────────┤
│  Linux Kernel (RHEL / Ubuntu / CentOS)       │
├──────────────────────────────────────────────┤
│  Physical Hardware (CPU/RAM/NIC/Disk)        │
└──────────────────────────────────────────────┘
```

### 8.3 Key Components

| Component | Purpose |
|-----------|---------|
| **KVM** | Kernel module for CPU/memory virtualization |
| **QEMU** | Hardware emulation (disk, NIC, USB) |
| **libvirt** | Management API and daemon (libvirtd) |
| **virsh** | CLI management tool |
| **virt-manager** | GUI management tool |
| **virt-install** | CLI for creating VMs |
| **qcow2** | Default disk image format (thin provisioning, snapshots) |
| **Open vSwitch** | Advanced virtual networking |

### 8.4 KVM vs QEMU

| Feature | KVM | QEMU (without KVM) |
|---------|-----|---------------------|
| Performance | Near-native (hardware assist) | Slow (full emulation) |
| CPU requirement | VT-x/AMD-V required | Any CPU |
| Use case | Production VMs | Cross-architecture emulation |

---

## 9. KVM Installation on Bare Metal / Blade Server

### 9.1 Pre-Installation (Blade Server)

#### Step 1: BIOS Configuration (same as ESXi)

```
1. Access iLO/iDRAC/CIMC
2. Enable:
   - Intel VT-x / AMD-V
   - Intel VT-d (for PCI passthrough)
   - NX/XD bit
   - Hyper-Threading
   - Boot Mode: UEFI
3. Configure RAID:
   - RAID 1 → OS boot (2x disks)
   - RAID 5/10 → VM storage
```

#### Step 2: Install Linux OS

```
1. Mount RHEL/CentOS/Ubuntu ISO via iLO virtual media
2. Boot from ISO
3. Install with these recommendations:
   - Partition layout:
     /boot     → 1 GB (ext4)
     /boot/efi → 512 MB (UEFI)
     swap      → 16–32 GB
     /         → 100 GB (xfs)
     /var/lib/libvirt → Remaining (for VM images)
   - Select "Virtualization Host" package group (RHEL)
   - Or "Server with GUI" + add virtualization later
4. Set static IP during install
5. Reboot
```

### 9.2 KVM Installation (RHEL/CentOS)

```bash
# Verify CPU supports virtualization
grep -E '(vmx|svm)' /proc/cpuinfo
lscpu | grep Virtualization

# Install KVM packages
yum install -y qemu-kvm libvirt libvirt-client virt-install virt-viewer virt-manager bridge-utils

# RHEL 9 / CentOS Stream 9
dnf install -y qemu-kvm libvirt virt-install virt-viewer cockpit-machines

# Enable and start libvirtd
systemctl enable --now libvirtd

# Verify KVM module loaded
lsmod | grep kvm
# kvm_intel or kvm_amd should appear

# Verify libvirt is running
systemctl status libvirtd
virsh list --all
```

### 9.3 KVM Installation (Ubuntu)

```bash
# Check virtualization support
egrep -c '(vmx|svm)' /proc/cpuinfo    # Should be > 0
kvm-ok                                 # Should say "KVM acceleration can be used"

# Install
apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager virtinst

# Add user to libvirt group
usermod -aG libvirt $USER
usermod -aG kvm $USER

# Enable
systemctl enable --now libvirtd
virsh list --all
```

### 9.4 Post-Installation Configuration

```bash
# Configure bridged networking (for VMs to get real IPs)
# See Section 11 for detailed bridge config

# Set default storage pool
virsh pool-define-as default dir --target /var/lib/libvirt/images
virsh pool-start default
virsh pool-autostart default
virsh pool-list --all

# Configure firewall
firewall-cmd --permanent --add-service=libvirt
firewall-cmd --reload

# Enable console access
systemctl enable --now serial-getty@ttyS0.service

# Web management (Cockpit)
yum install cockpit cockpit-machines -y
systemctl enable --now cockpit.socket
# Access: https://<host-ip>:9090
```

---

## 10. KVM Administration

### 10.1 Creating VMs

```bash
# Method 1: virt-install (CLI — recommended for automation)
virt-install \
  --name rhel9-web01 \
  --ram 4096 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/rhel9-web01.qcow2,size=50,format=qcow2 \
  --os-variant rhel9.0 \
  --network bridge=br0 \
  --graphics vnc,listen=0.0.0.0 \
  --location /iso/rhel-9.0-x86_64-dvd.iso \
  --extra-args "console=ttyS0" \
  --noautoconsole

# Method 2: From existing disk image
virt-install \
  --name ubuntu-vm \
  --ram 2048 \
  --vcpus 2 \
  --import \
  --disk /var/lib/libvirt/images/ubuntu.qcow2 \
  --os-variant ubuntu22.04 \
  --network bridge=br0 \
  --noautoconsole

# Method 3: Using kickstart (automated)
virt-install \
  --name auto-vm \
  --ram 4096 \
  --vcpus 4 \
  --disk size=80 \
  --os-variant rhel9.0 \
  --location /iso/rhel9.iso \
  --initrd-inject=/path/to/ks.cfg \
  --extra-args "ks=file:/ks.cfg console=ttyS0" \
  --noautoconsole
```

### 10.2 virsh Commands (Daily Operations)

```bash
# List VMs
virsh list --all                       # All VMs
virsh list                             # Running only

# Power operations
virsh start <vm>
virsh shutdown <vm>                    # Graceful (ACPI)
virsh destroy <vm>                     # Force power off
virsh reboot <vm>
virsh suspend <vm>                     # Pause
virsh resume <vm>                      # Unpause
virsh autostart <vm>                   # Start on host boot
virsh autostart --disable <vm>

# VM info
virsh dominfo <vm>
virsh domblklist <vm>                  # Disk list
virsh domiflist <vm>                   # NIC list
virsh vcpuinfo <vm>
virsh dommemstat <vm>

# Console access
virsh console <vm>                     # Serial console
virt-viewer <vm>                       # Graphical console

# Edit VM config (XML)
virsh edit <vm>
virsh dumpxml <vm> > vm.xml            # Export config
virsh define vm.xml                    # Import/update config
virsh undefine <vm>                    # Remove VM definition
virsh undefine <vm> --remove-all-storage  # Remove VM + disks
```

### 10.3 Snapshot Management

```bash
# Create snapshot
virsh snapshot-create-as <vm> --name "pre-patch" --description "before patching"

# List snapshots
virsh snapshot-list <vm>

# Revert to snapshot
virsh snapshot-revert <vm> --snapshotname "pre-patch"

# Delete snapshot
virsh snapshot-delete <vm> --snapshotname "pre-patch"

# Internal vs External snapshots
# Internal: stored in qcow2 (simple, supports revert)
# External: separate overlay file (better for production, harder to manage)
```

### 10.4 Resource Management

```bash
# Change vCPU (hot-add if supported)
virsh setvcpus <vm> 4 --config         # Next boot
virsh setvcpus <vm> 4 --live           # Hot-add (if enabled)
virsh setmaxmem <vm> 8G --config

# Change memory
virsh setmem <vm> 4G --live            # Hot-plug (if enabled)
virsh setmem <vm> 4G --config          # Next boot

# Add disk
virsh attach-disk <vm> /var/lib/libvirt/images/extra.qcow2 vdb --persistent --subdriver qcow2

# Remove disk
virsh detach-disk <vm> vdb --persistent

# Add network interface
virsh attach-interface <vm> --type bridge --source br0 --model virtio --persistent

# CPU pinning (for performance)
virsh vcpupin <vm> 0 2                 # Pin vCPU 0 to physical CPU 2
virsh emulatorpin <vm> 0-1             # Pin emulator threads
```

### 10.5 VM Migration

```bash
# Live migration (shared storage required)
virsh migrate --live <vm> qemu+ssh://dest-host/system

# Offline migration
virsh migrate --offline <vm> qemu+ssh://dest-host/system

# With storage (no shared storage needed)
virsh migrate --live --copy-storage-all <vm> qemu+ssh://dest-host/system

# Prerequisites:
# - Same libvirt version on both hosts
# - SSH key-based auth between hosts
# - Same network (bridge) name on destination
# - Firewall allows libvirt ports (16509, 49152-49215)
```

### 10.6 Templates & Cloning

```bash
# Prepare template (inside VM)
# Remove unique identifiers:
rm -f /etc/machine-id
rm -f /etc/ssh/ssh_host_*
# Or use virt-sysprep:
virt-sysprep -d <vm>

# Clone VM
virt-clone --original rhel9-template --name web01 --auto-clone

# Clone with custom disk
virt-clone --original rhel9-template --name web02 --file /var/lib/libvirt/images/web02.qcow2
```

---


## 11. KVM Networking

### 11.1 Network Modes

| Mode | Description | VM gets IP from |
|------|-------------|-----------------|
| **NAT** (default) | VMs share host IP via NAT | libvirt DHCP (192.168.122.x) |
| **Bridged** | VMs appear on physical network | Physical network DHCP/static |
| **Isolated** | VMs talk only to each other | libvirt DHCP |
| **Macvtap** | Direct connection to physical NIC | Physical network |
| **Open vSwitch** | Advanced SDN switching | Depends on config |

### 11.2 Configure Bridge Network (Production)

```bash
# Method 1: nmcli (RHEL 7/8/9)
nmcli con add type bridge con-name br0 ifname br0
nmcli con add type ethernet con-name br0-slave0 ifname eth0 master br0
nmcli con mod br0 ipv4.addresses 192.168.1.50/24
nmcli con mod br0 ipv4.gateway 192.168.1.1
nmcli con mod br0 ipv4.dns "8.8.8.8"
nmcli con mod br0 ipv4.method manual
nmcli con up br0

# Verify
brctl show
ip addr show br0
bridge link show

# Method 2: Netplan (Ubuntu)
cat > /etc/netplan/01-bridge.yaml << 'EOF'
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
  bridges:
    br0:
      interfaces: [eth0]
      addresses: [192.168.1.50/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
EOF
netplan apply
```

### 11.3 Libvirt Network Management

```bash
# List networks
virsh net-list --all

# Default NAT network
virsh net-dumpxml default

# Create custom bridged network
cat > bridge-net.xml << 'EOF'
<network>
  <name>br0-net</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>
EOF
virsh net-define bridge-net.xml
virsh net-start br0-net
virsh net-autostart br0-net

# Create isolated network
cat > isolated-net.xml << 'EOF'
<network>
  <name>internal</name>
  <ip address="10.10.10.1" netmask="255.255.255.0">
    <dhcp>
      <range start="10.10.10.100" end="10.10.10.200"/>
    </dhcp>
  </ip>
</network>
EOF
virsh net-define isolated-net.xml
virsh net-start internal
virsh net-autostart internal
```

### 11.4 VLAN Configuration with KVM

```bash
# Create VLAN interface
nmcli con add type vlan con-name vlan100 dev eth0 id 100

# Create bridge on VLAN
nmcli con add type bridge con-name br-vlan100 ifname br-vlan100
nmcli con add type vlan con-name br-vlan100-slave dev eth0 id 100 master br-vlan100
nmcli con mod br-vlan100 ipv4.method disabled
nmcli con up br-vlan100

# Attach VM to VLAN bridge
virsh attach-interface <vm> --type bridge --source br-vlan100 --model virtio --persistent
```

---

## 12. KVM Storage

### 12.1 Disk Image Formats

| Format | Features | Use Case |
|--------|----------|----------|
| **qcow2** | Thin provisioning, snapshots, compression, encryption | Default, recommended |
| **raw** | Full allocation, no features, fastest I/O | Performance-critical |
| **vmdk** | VMware format | Migration from ESXi |
| **vdi** | VirtualBox format | Migration from VBox |

### 12.2 Storage Pool Management

```bash
# List pools
virsh pool-list --all

# Create directory-based pool
virsh pool-define-as vmpool dir --target /data/vms
virsh pool-build vmpool
virsh pool-start vmpool
virsh pool-autostart vmpool

# Create LVM-based pool
virsh pool-define-as lvm-pool logical --source-name vg_vms --target /dev/vg_vms
virsh pool-start lvm-pool
virsh pool-autostart lvm-pool

# Create iSCSI pool
virsh pool-define-as iscsi-pool iscsi --source-host 192.168.30.100 --source-dev iqn.2024.com.storage:lun1 --target /dev/disk/by-path

# Pool operations
virsh pool-info vmpool
virsh pool-refresh vmpool
virsh pool-destroy vmpool              # Stop pool
virsh pool-undefine vmpool             # Remove pool definition
```

### 12.3 Volume Management

```bash
# Create volume
virsh vol-create-as vmpool web01.qcow2 50G --format qcow2

# List volumes
virsh vol-list vmpool

# Volume info
virsh vol-info /var/lib/libvirt/images/web01.qcow2

# Delete volume
virsh vol-delete /var/lib/libvirt/images/web01.qcow2

# Resize volume
virsh vol-resize /var/lib/libvirt/images/web01.qcow2 100G

# Convert format
qemu-img convert -f vmdk -O qcow2 input.vmdk output.qcow2
qemu-img convert -f raw -O qcow2 input.raw output.qcow2

# Image info
qemu-img info /var/lib/libvirt/images/web01.qcow2
```

### 12.4 Disk Operations

```bash
# Create disk image
qemu-img create -f qcow2 /var/lib/libvirt/images/data.qcow2 100G

# Resize disk (grow)
qemu-img resize /var/lib/libvirt/images/disk.qcow2 +20G
# Then inside VM: growpart, resize2fs/xfs_growfs

# Attach disk to running VM
virsh attach-disk <vm> /path/to/disk.qcow2 vdb --driver qemu --subdriver qcow2 --persistent

# Detach disk
virsh detach-disk <vm> vdb --persistent

# Snapshot with qemu-img
qemu-img snapshot -c "snap1" disk.qcow2
qemu-img snapshot -l disk.qcow2        # List
qemu-img snapshot -a "snap1" disk.qcow2 # Apply/revert
```

---

## 13. KVM Advanced Topics

### 13.1 PCI Passthrough (GPU/NIC to VM)

```bash
# Enable IOMMU in GRUB
# /etc/default/grub
GRUB_CMDLINE_LINUX="intel_iommu=on"    # Intel
GRUB_CMDLINE_LINUX="amd_iommu=on"     # AMD
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot

# Find device IOMMU group
find /sys/kernel/iommu_groups/ -type l

# Detach device from host
virsh nodedev-detach pci_0000_03_00_0

# Attach to VM (edit XML)
virsh edit <vm>
# Add under <devices>:
<hostdev mode='subsystem' type='pci' managed='yes'>
  <source>
    <address domain='0x0000' bus='0x03' slot='0x00' function='0x0'/>
  </source>
</hostdev>
```

### 13.2 Virtio Drivers (Performance)

| Driver | Purpose | vs Emulated |
|--------|---------|-------------|
| virtio-blk | Disk I/O | 2-3x faster than IDE |
| virtio-net | Network | 10x faster than e1000 |
| virtio-scsi | SCSI storage | Better scalability |
| virtio-balloon | Memory management | Dynamic memory |

```bash
# Use virtio when creating VM
virt-install --disk ...,bus=virtio --network ...,model=virtio
```

### 13.3 NUMA Tuning

```bash
# Check NUMA topology
numactl --hardware
virsh capabilities | grep -A 10 topology

# Pin VM to NUMA node (for performance)
virsh edit <vm>
# Add:
<numatune>
  <memory mode='strict' nodeset='0'/>
</numatune>

# Hugepages (better TLB performance)
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
# In VM XML:
<memoryBacking>
  <hugepages/>
</memoryBacking>
```

### 13.4 KVM Security

```bash
# SELinux (sVirt) — automatic VM isolation
# Each VM gets unique SELinux label
ps -eZ | grep qemu

# Limit VM resources (cgroups)
virsh schedinfo <vm>
virsh schedinfo <vm> --set cpu_shares=1024
virsh blkiotune <vm> --weight 500

# Network security
# Use nwfilter for VM firewall rules
virsh nwfilter-list
virsh nwfilter-dumpxml clean-traffic
```

### 13.5 Backup & Disaster Recovery

```bash
# Full VM backup (offline)
virsh shutdown <vm>
virsh dumpxml <vm> > /backup/vm.xml
cp /var/lib/libvirt/images/vm.qcow2 /backup/

# Online backup (external snapshot)
virsh snapshot-create-as <vm> snap1 --disk-only --atomic
# Copy base image, then commit:
virsh blockcommit <vm> vda --active --pivot

# Restore VM
virsh define /backup/vm.xml
cp /backup/vm.qcow2 /var/lib/libvirt/images/
virsh start <vm>
```

### 13.6 Performance Monitoring

```bash
# VM CPU/Memory stats
virsh domstats <vm>
virsh cpu-stats <vm>
virsh dommemstat <vm>

# Disk I/O stats
virsh domblkstat <vm> vda

# Network stats
virsh domifstat <vm> vnet0

# virt-top (like top for VMs)
yum install virt-top -y
virt-top
```

---

## 14. ESXi vs KVM Comparison

| Feature | VMware ESXi | KVM |
|---------|-------------|-----|
| **Type** | Type-1 (bare-metal) | Type-1 (kernel module) |
| **License** | Commercial (expensive) | Free/Open-source |
| **Management** | vCenter, vSphere Client | virsh, virt-manager, Cockpit |
| **Live Migration** | vMotion | virsh migrate --live |
| **HA** | vSphere HA | Pacemaker/Corosync, oVirt |
| **Storage** | VMFS, vSAN, NFS | qcow2, LVM, Ceph, NFS |
| **Networking** | vDS, NSX | Bridge, OVS, OVN |
| **Snapshot** | Yes (UI/CLI) | Yes (virsh/qemu-img) |
| **Max VMs/host** | 1024 | Limited by resources |
| **Max vCPU/VM** | 768 (ESXi 8) | 710+ |
| **Max RAM/VM** | 24 TB | Limited by host |
| **Enterprise GUI** | vCenter | oVirt, Proxmox |
| **Cloud integration** | VMware Cloud | OpenStack, CloudStack |
| **Support** | Broadcom/VMware | Red Hat, Canonical, community |
| **Best for** | Enterprise, existing VMware shops | Cloud providers, cost-sensitive |

---


## 15. Interview Questions & Answers

### ESXi Questions

**Q1. What is the difference between Type-1 and Type-2 hypervisor?**

| Type-1 (Bare-metal) | Type-2 (Hosted) |
|---------------------|-----------------|
| Runs directly on hardware | Runs on top of an OS |
| ESXi, KVM, Hyper-V | VirtualBox, VMware Workstation |
| Better performance | More overhead |
| Production use | Development/testing |

---

**Q2. What is vMotion and what are its requirements?**

vMotion is live migration of a running VM from one ESXi host to another with zero downtime.

Requirements:
- Shared storage accessible by both hosts
- VMkernel port enabled for vMotion on both hosts
- Compatible CPUs (or EVC mode enabled)
- Gigabit+ network between hosts
- VMs cannot have local resources (USB passthrough, etc.)

---

**Q3. What is the difference between thick and thin provisioning?**

| Thin | Thick Lazy Zeroed | Thick Eager Zeroed |
|------|-------------------|-------------------|
| Allocates on demand | Full space allocated | Full space allocated + zeroed |
| Can over-commit | Cannot over-commit | Cannot over-commit |
| Slight write penalty | First-write zeroing | Best performance |
| General VMs | General VMs | FT, databases |

---

**Q4. How do you troubleshoot a VM that won't power on?**

1. Check datastore space: `esxcli storage filesystem list`
2. Check VM logs: `/vmfs/volumes/<ds>/<vm>/vmware.log`
3. Check if .vmx file is valid
4. Check if disk file (.vmdk) is locked: `vmkfstools -D <vmdk>`
5. Check host resources (CPU/RAM availability)
6. Check if VM is still registered
7. Check HA/DRS admission control

---

**Q5. What is VMFS and what version are you using?**

- VMFS = Virtual Machine File System (clustered filesystem)
- Allows multiple ESXi hosts to access same datastore simultaneously
- VMFS 6 = current version (ESXi 6.5+)
- Supports files up to 62 TB
- Block size: 1 MB (fixed in VMFS 6)

---

**Q6. Explain ESXi snapshots. Are they backups?**

- Snapshots capture VM state (disk, memory, settings) at a point in time
- They are NOT backups — stored on same datastore
- Use delta disks (.delta.vmdk) that grow over time
- Performance degrades with many snapshots
- Should not keep > 72 hours in production
- Use for: pre-patch safety, testing changes

---

**Q7. What is HA admission control?**

- Reserves cluster resources to guarantee VM restart capacity
- Ensures that if X hosts fail, remaining hosts have enough resources
- Policies: Host failures tolerated, % of resources reserved, slot policy
- Can prevent powering on VMs if it would violate policy

---

**Q8. How do you upgrade ESXi?**

```bash
# Method 1: CLI (single host)
esxcli software profile update -p ESXi-7.0U3-profile -d /vmfs/volumes/ds1/VMware-ESXi-7.0U3.zip

# Method 2: vSphere Lifecycle Manager (vLCM) — cluster-wide
# Method 3: Auto Deploy (stateless)

# Always: put host in maintenance mode first
esxcli system maintenanceMode set --enable true
```

---

**Q9. What is DRS and how does it work?**

- Distributed Resource Scheduler
- Monitors CPU/memory usage across cluster
- Automatically migrates VMs (via vMotion) to balance load
- Modes: Manual, Partially Automated, Fully Automated
- Uses star ratings (1-5) for migration recommendations
- Runs every 5 minutes by default

---

**Q10. How to recover ESXi from PSOD (Purple Screen of Death)?**

1. Note error message and screenshot
2. Check hardware (RAM, CPU, NIC, HBA)
3. Check VMware KB for specific error code
4. Review `/var/log/vmkernel.log` after reboot
5. Run hardware diagnostics
6. Update ESXi, drivers, firmware
7. If recurring: replace faulty hardware

---

### KVM Questions

**Q11. How to check if KVM is properly installed and running?**

```bash
lsmod | grep kvm                      # kvm_intel or kvm_amd loaded
virsh list --all                      # libvirt working
systemctl status libvirtd             # Daemon running
virt-host-validate                    # Full validation check
```

---

**Q12. What is the difference between qcow2 and raw disk format?**

| qcow2 | raw |
|-------|-----|
| Thin provisioned | Full allocation |
| Supports snapshots | No snapshots |
| Supports compression | No compression |
| Supports encryption | No built-in encryption |
| Slightly slower I/O | Fastest I/O |
| Default for KVM | Used for performance workloads |

---

**Q13. How to perform live migration in KVM?**

```bash
# Requirements:
# - Same libvirt version
# - SSH key-based auth
# - Shared storage (or use --copy-storage-all)
# - Same bridge/network names

virsh migrate --live --verbose <vm> qemu+ssh://destination/system
```

---

**Q14. How to resize a KVM VM disk?**

```bash
# 1. Resize image file (host side)
qemu-img resize /var/lib/libvirt/images/vm.qcow2 +20G

# 2. Inside VM: extend partition and filesystem
growpart /dev/vda 2
xfs_growfs /                          # XFS
resize2fs /dev/vda2                   # ext4
```

---

**Q15. How to convert VMware VM to KVM?**

```bash
# 1. Export from VMware as OVA/OVF or copy VMDK
# 2. Convert disk
qemu-img convert -f vmdk -O qcow2 vm-disk.vmdk vm-disk.qcow2

# 3. Create VM with converted disk
virt-install --name converted-vm --ram 4096 --vcpus 2 \
  --import --disk vm-disk.qcow2 --os-variant rhel9.0 \
  --network bridge=br0 --noautoconsole

# 4. Inside VM: remove VMware tools, install virtio drivers
```

---

**Q16. What is libvirt and why is it used?**

- Abstraction API for managing hypervisors (KVM, Xen, LXC, QEMU)
- Provides: virsh CLI, XML-based VM definitions, API for tools
- Manages: domains (VMs), storage pools, networks, snapshots
- Daemon: `libvirtd` (monolithic) or modular daemons (RHEL 9+)

---

**Q17. How to troubleshoot a KVM VM that won't start?**

```bash
# 1. Check error message
virsh start <vm>   # Read the error

# 2. Validate XML
virsh edit <vm>
virt-xml-validate /etc/libvirt/qemu/<vm>.xml

# 3. Check disk path exists
virsh domblklist <vm>
ls -la /var/lib/libvirt/images/<disk>

# 4. Check permissions
ls -Z /var/lib/libvirt/images/        # SELinux context
chown qemu:qemu <disk_file>

# 5. Check libvirtd logs
journalctl -u libvirtd --since "5 min ago"

# 6. Check if another process holds the disk
fuser /var/lib/libvirt/images/<disk>
```

---

**Q18. Explain KVM networking modes.**

- **NAT (default/virbr0):** VMs access external network via host NAT. External cannot reach VMs directly. Good for development.
- **Bridged (br0):** VMs get IPs from physical network. Appear as real machines. Required for production.
- **Isolated:** VMs only communicate with each other. No external access. Good for lab/testing.
- **Macvtap:** Direct attach to physical NIC. Better performance but host cannot communicate with VM.

---

**Q19. How to take a consistent backup of a running KVM VM?**

```bash
# Method 1: External snapshot + backup + commit
virsh snapshot-create-as <vm> backup-snap --disk-only --atomic --quiesce
# Backup the base image (it's now read-only)
cp /var/lib/libvirt/images/vm.qcow2 /backup/
# Commit overlay back and remove snapshot
virsh blockcommit <vm> vda --active --pivot
virsh snapshot-delete <vm> backup-snap --metadata

# Method 2: Use guest agent for filesystem quiesce
virsh domfsfreeze <vm>
# backup disk
virsh domfsthaw <vm>
```

---

**Q20. What is oVirt and how does it compare to vCenter?**

| Feature | oVirt | vCenter |
|---------|-------|---------|
| License | Free (open-source) | Commercial |
| Based on | KVM + libvirt | ESXi |
| Web UI | Yes (oVirt Engine) | Yes (vSphere Client) |
| HA | Yes | Yes |
| Live migration | Yes | Yes (vMotion) |
| Storage | NFS, iSCSI, GlusterFS, Ceph | VMFS, NFS, vSAN |
| Template/Clone | Yes | Yes |
| API | REST | REST + SOAP |

---

### Scenario-Based Questions

**Q21. Your ESXi host shows "No space left" on VMFS datastore. What do you do?**

1. Check snapshot sizes: may have forgotten snapshots growing
2. Identify large files: browse datastore in vSphere client
3. Delete old snapshots (consolidate if needed)
4. Storage vMotion VMs to another datastore
5. Thin-provisioned disks may have grown — reclaim space
6. Extend VMFS datastore (add extent or grow LUN)

---

**Q22. A KVM VM is extremely slow. How to troubleshoot?**

```bash
# 1. Check host resources
top / htop / virt-top

# 2. Check if using virtio (not IDE/e1000)
virsh dumpxml <vm> | grep -i "model type"
# Should show: virtio-blk, virtio-net

# 3. Check I/O wait
iostat -xz 1 5
virsh domblkstat <vm> vda

# 4. Check NUMA alignment
numactl --hardware
virsh numatune <vm>

# 5. Check for memory balloon stealing RAM
virsh dommemstat <vm>

# 6. Check overcommit ratio
# If host RAM overcommitted, VMs will swap

# Fix: use virtio, pin NUMA, allocate hugepages, avoid overcommit
```

---

**Q23. You need to migrate 50 VMs from ESXi to KVM. Describe your approach.**

```
1. Inventory: List all VMs, OS types, disk sizes, networks
2. Pilot: Convert 2-3 non-critical VMs first
3. Process per VM:
   a. Shutdown VM in ESXi
   b. Export as OVA or copy VMDK files
   c. Convert: qemu-img convert -f vmdk -O qcow2 disk.vmdk disk.qcow2
   d. Create VM: virt-install --import ...
   e. Inside VM:
      - Remove VMware tools
      - Install virtio drivers (if Windows)
      - Update NIC config (new MAC)
      - Verify hostname, IP, services
4. Validate: test all applications
5. Cutover: DNS/IP switch
6. Decommission ESXi VM after validation period
```

---

**Q24. ESXi host is in "Not Responding" state in vCenter. How to fix?**

1. Check host via DCUI (direct console) — is it alive?
2. Check network: ping management IP from vCenter
3. Check vpxa agent: `/etc/init.d/vpxa restart` (on ESXi)
4. Check management agents: `services.sh restart`
5. Check if host is in PSOD (purple screen)
6. Check storage: `esxcli storage filesystem list`
7. Disconnect and reconnect host in vCenter
8. If host is truly dead: declare HA event, VMs restart on other hosts

---

**Q25. How to set up KVM high availability?**

```
Option 1: Pacemaker + Corosync
- Install pacemaker, corosync, fence-agents
- Configure shared storage (NFS/iSCSI/Ceph)
- Define VM as cluster resource
- Configure fencing (IPMI/iLO)

Option 2: oVirt (full solution)
- Deploy oVirt Engine
- Add KVM hosts to cluster
- Configure HA policy per VM
- Shared storage required

Option 3: Custom with libvirt + monitoring
- Watchdog daemon monitors VMs
- If host fails, script starts VMs on standby host
- Requires shared storage + fencing
```

---

<div align="center">

## 📌 Quick Reference Card

| Task | ESXi | KVM |
|------|------|-----|
| List VMs | `vim-cmd vmsvc/getallvms` | `virsh list --all` |
| Power on | `vim-cmd vmsvc/power.on <id>` | `virsh start <vm>` |
| Power off | `vim-cmd vmsvc/power.off <id>` | `virsh destroy <vm>` |
| Snapshot | UI or `vim-cmd vmsvc/snapshot.create` | `virsh snapshot-create-as` |
| Live migrate | vMotion (vCenter) | `virsh migrate --live` |
| Console | vSphere Client / VMRC | `virsh console` / `virt-viewer` |
| Storage | VMFS datastore | qcow2 / LVM pool |
| Network | vSwitch / vDS | Bridge / OVS |
| Logs | `/var/log/vmkernel.log` | `journalctl -u libvirtd` |

---

**Prepared by [Krishna Yada](https://github.com/yadakrishna245)**

*Senior System Administrator | 8+ Years | VMware & KVM Production Environments*

⭐ **Star this repo if it helped you!** ⭐

</div>
