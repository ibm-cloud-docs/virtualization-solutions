---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Linux Migration Considerations
{: #virt-sol-vpc-migration-design-linux}

Linux migrations are generally simpler than Windows, but there are still important considerations.

## VirtIO Driver Verification
{: #virt-sol-vpc-migration-design-linux-virtio}

Most modern Linux distributions include VirtIO drivers in the kernel:
- **Ubuntu**: 16.04 and later
- **RHEL/CentOS**: 6.x and later  
- **Debian**: 8 and later
- **SUSE**: 12 and later

**Verify drivers are present**:
```bash
lsmod | grep virtio
```

You should see:
- `virtio_blk` - Block device driver
- `virtio_net` - Network driver  
- `virtio_scsi` - SCSI driver
- `virtio_pci` - PCI bus driver

If these are missing, you'll need to rebuild the kernel with VirtIO support or install a newer distribution.

## No Sysprep Equivalent Needed
{: #virt-sol-vpc-migration-design-linux-nosysprep}

Linux doesn't bind drivers to hardware the way Windows does. The kernel detects new hardware at boot and loads appropriate drivers. This makes migration significantly easier with no special preparation typically required.

## Network Configuration Adjustments
{: #virt-sol-vpc-migration-design-linux-network}

**Common Issue**: Network interface names change during migration.

In VMware, your interface might be:
- `ens192` (systemd predictable naming)
- `eth0` (traditional naming)

In VPC, it might become:
- `ens3` or `ens33` (common in VirtIO environments)
- `eth0` (if using traditional naming)

**Fix for Static IP Configuration**:

**NetworkManager-based (RHEL 7+, newer Ubuntu)**:
```bash
# Identify new interface name
ip link show

# Edit connection
nmcli con edit "System eth0"
# Change interface-name to new name
# Save and quit

# Restart NetworkManager
systemctl restart NetworkManager
```

**netplan-based (Ubuntu 18.04+)**:
```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    ens3:  # Updated from ens192
      addresses: [10.240.0.10/24]
      gateway4: 10.240.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

**Traditional /etc/network/interfaces (Debian, older Ubuntu)**:
```bash
auto ens3
iface ens3 inet static
  address 10.240.0.10
  netmask 255.255.255.0
  gateway 10.240.0.1
```

**Fix for DHCP Configuration**:

If using DHCP, the configuration should work automatically, but you may still need to update interface names in config files.

## Cloud-Init Considerations
{: #virt-sol-vpc-migration-design-linux-cloudinit}

**What is cloud-init**: A tool for initializing cloud instances, typically used with image templates. It runs on first boot to:
- Set hostname
- Configure networking
- Create users and SSH keys
- Run custom scripts

**Migration Context**:

If cloud-init is installed on your migrated VM, VPC may treat the first boot as a "first boot," triggering:
- Hostname changes
- Network reconfiguration
- User account changes
- Execution of cloud-init scripts

### Design Decisions
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions}

### Option 1: Disable cloud-init
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions1}

```bash
# Before migration
sudo touch /etc/cloud/cloud-init.disabled

# Or after migration via VNC console
```

### Option 2: Accept first-boot behavior
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions2}

- Useful if you want VPC to auto-configure networking via DHCP
- May require manual adjustments post-boot (hostname, users, etc.)

### Option 3: Configure cloud-init for VPC
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions3}

- Create cloud-init config that preserves your settings
- More complex but gives you control

**Recommendation**: For individual VM migrations (not template deployments), disable cloud-init to preserve existing configuration. For template-based deployments, leverage cloud-init for automatic configuration.

## Partition and Filesystem Considerations
{: #virt-sol-vpc-migration-design-linux-partitions}

**Partition Table Verification**:

After transferring disks, verify partition tables are intact:
```bash
# On worker VSI after transfer
fdisk -l /dev/vdb

# For GPT
gdisk -l /dev/vdb
```

**Boot Volume Resize**:

If you resized the boot volume upward (from 80GB in VMware to 100GB in VPC):

1. The partition table may need updating:
   ```bash
   # For MBR
   fdisk /dev/vda
   # Delete and recreate partition with same start sector, new end sector
   
   # For GPT (automatic backup GPT update)
   gdisk /dev/vda
   ```

2. Resize filesystem:
   ```bash
   # For ext4
   resize2fs /dev/vda1
   
   # For xfs
   xfs_growfs /
   
   # For LVM
   pvresize /dev/vda2
   lvextend -l +100%FREE /dev/mapper/vg-root
   resize2fs /dev/mapper/vg-root
   ```

## fstrim and Thin Provisioning
{: #virt-sol-vpc-migration-design-linux-fstrim}

In VMware, you might use `fstrim` to reclaim unused space in thin-provisioned VMDKs. In VPC, all volumes are thin-provisioned at the storage layer, but **fstrim has no effect on space reclamation** in VPC. Running it won't harm anything, but it doesn't free up storage in the way it does with VMware.

You can leave existing fstrim cron jobs in place (they're harmless), or remove them to avoid unnecessary I/O.
