---

copyright:
  years: 2025, 2026
lastupdated: "2026-05-18"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Linux migration considerations
{: #virt-sol-vpc-migration-design-linux}



Before you start a migration, you need to consider the following information.

## Verify VirtIO drivers
{: shortdesc}
{: #virt-sol-vpc-migration-design-linux-virtio}

Linux migrations are generally simpler than Windows, but the guide describes a few important considerations to keep in mind.

The following Linux distributions include VirtIO drivers in the kernel:

- Ubuntu: 16.04 and later
- RHEL/CentOS: 6.x and later
- Debian: 8 and later
- SUSE: 12 and later

Use the following command to verify drivers are present.

```bash
lsmod | grep virtio
```
{: pre}

You see the following drivers:

- `virtio_blk` - Block device driver
- `virtio_net` - Network driver
- `virtio_scsi` - SCSI driver
- `virtio_pci` - PCI bus driver

If these drivers are missing, you need to rebuild the kernel with VirtIO support or install a more recent distribution.

## No Sysprep equivalent needed
{: #virt-sol-vpc-migration-design-linux-nosysprep}



Linux doesn't bind drivers to hardware. The kernel detects new hardware at boot and loads the appropriate drivers. This process makes migration simple with no preparation typically required.

## Adjust the network configuration
{: #virt-sol-vpc-migration-design-linux-adjust-network-configuration}

Linux does not bind drivers to hardware like Windows. The kernel detects new hardware at boot and loads appropriate drivers, which makes migration significantly easier with no special preparation typically required.



Common issue: Network interface names change during migration.

In VMware, your interface might be named:

- `ens192` (systemd predictable naming)
- `eth0` (traditional naming)

In VPC, it might change to:

- `ens3` or `ens33` (common in VirtIO environments)
- `eth0` (if you use traditional naming conventions)

To resolve Static IP configuration, use the following information:

- NetworkManager-based (RHEL 7+ or newer Ubuntu versions)

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
   {: codeblock}

- Netplan-based (Ubuntu 18.04+)

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
   {: codeblock}

- Traditional /etc/network/interfaces (Debian, older Ubuntu)

   ```bash
   auto ens3
   iface ens3 inet static
     address 10.240.0.10
     netmask 255.255.255.0
     gateway 10.240.0.1
   ```
   {: codeblock}

Fix for DHCP configuration:

If you use DHCP, the configuration is automatic, but you might need to update interface names in config files.

## Cloud-init considerations
{: #virt-sol-vpc-migration-design-linux-cloudinit}



Cloud-init is used to initialize cloud instances, typically used with image templates. It runs on the first boot to do the following actions:

- Set a hostname
- Configure networking
- Create users and SSH keys
- Run custom scripts

Migration context: If cloud-init is installed on your migrated virtual server, VPC might treat the first boot as a "first boot", which triggers the following actions:

- Hostname changes
- Network reconfiguration
- User account changes
- Execution of cloud-init scripts

### Design decisions
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions}



### Option 1: Disable cloud-init
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions1}



```bash
# Before migration
sudo touch /etc/cloud/cloud-init.disabled

# Or after migration via VNC console
```
{: codeblock}

### Option 2: Accept first-boot behavior
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions2}

- Useful if you want VPC to auto-configure networking through DHCP
- Might require manual adjustments post-boot (hostname, users)

### Option 3: Configure cloud-init
{: #virt-sol-vpc-migration-design-linux-cloudinit-decisions3}



- Create a cloud-init config to preserve your settings
- More complex but gives you control

For individual VM migrations (not template deployments), disable cloud-init to preserve existing configuration. For template-based deployments, use cloud-init for automatic configuration.
{: important}

## Partition and file system considerations
{: #virt-sol-vpc-migration-design-linux-partitions}

Partition table verification: After you transfer the disks, use the following command to verify that the partition tables are intact:

```bash
# On worker VSI after transfer
fdisk -l /dev/vdb

# For GPT
gdisk -l /dev/vdb
```
{: codeblock}

Boot volume resize:
If you resized the boot volume upward (example: from 80 GB in VMware to 100 GB in VPC),

1. You might need to update the partition table:

If you resized the boot volume upward (from 80 GB in VMware to 100 GB in VPC):

   ```bash
   # For MBR
   fdisk /dev/vda
   # Delete and recreate partition with same start sector, new end sector

   # For GPT (automatic backup GPT update)
   gdisk /dev/vda
   ```
   {: codeblock}

1. Resize file system:

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
   {: codeblock}

## fstrim and Thin Provisioning
{: #virt-sol-vpc-migration-design-linux-fstrim}

In VMware, you might use `fstrim` to reclaim unused space in thin-provisioned VMDKs. In VPC, all volumes are thin-provisioned at the storage layer, but fstrim has no effect on space reclamation in VPC. Keep in mind that fstrim doesn't free up storage.

Existing fstrim cron jobs can stay in place or remove them to avoid unnecessary I/O.
