---

copyright:
  years: 2025
lastupdated: "2026-01-12"

keywords: virtual server instance, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Live network transfer (Recommended for Scale)
{: #virt-sol-vpc-migration-design-method3}

For Large-scale migrations, minimizing downtime, maximum efficiency, scenarios where exporting virtual machineDKs is impractical, you can migrate to a virtual server using live network transfer. Instead of shutting down your virtual machine, exporting it, transferring the export, and converting it, you boot the source virtual machine from a live ISO (like a rescue disk), read its disks from within this live environment, and stream them directly over the network to your worker virtual server instance in VPC.
{: shortdesc}

## Architecture Components
{: #virt-sol-vpc-migration-design-method3-architecture}

The architecture components of a live network transfer migration are:

- Transit Gateway
   - Connects your VMware environment (Classic, NSX, or VCFaaS) to your VPC. This provides the Layer 3 routing necessary for your source virtual machine (in VMware) to communicate with your worker virtual server instance (in VPC).
- Worker virtual server instance
   - Similar to Method 2, but here it's purely a receiver—listening on a network port for incoming disk data.
- Live ISO
   - A bootable ISO that provides:
      - Network configuration tools
      - Disk reading utilities (`dd`)
      - Compression tools (`gzip`, `pigz`)
      - Network transfer tools (`netcat`, `socat`, `ssh`)
- Boot Source Options
   - Ubuntu Install ISO: Has a "Enter shell" option in the help menu, includes most needed tools
   - TinyCore Linux: Extremely small but requires package installation for ssh/qemu
   - virt-p2v ISO: Purpose-built by Red Hat for P2V migrations, integrates with virt-v2v on receiver
   - G4L (Ghost for Linux): Imaging-focused live Linux

## Overview of the live network transfer migration process
{: #virt-sol-vpc-migration-design-method3-process}

The following steps layout the process to migrate using live network transfer.

1. Provision Transit Gateway
   1. Create Transit Gateway in IBM Cloud
   1. Connect it to your VMware environment:
     - Classic: Direct connection to Classic account
     - NSX: GRE tunnels to NSX edges
     - VCFaaS: GRE tunnels to VCFaaS edges
   1. Configure routing between VMware networks and VPC subnets
   1. Test connectivity thoroughly before first migration
1. Provision Worker virtual server instance in VPC
   1. Ubuntu or RHEL with adequate resources
   1. Create target volumes using ephemeral virtual server instance technique (Method 2 steps 2-4)
   1. Attach target volumes to worker
   1. Install netcat: `apt-get install netcat` or `yum install nc`
1. Prepare Source VM for Transfer
   1. Attach live ISO to virtual machine in vCenter/VCFaaS
   1. Configure virtual machine boot order to boot from CD/ISO first
   1. Note current IP configuration for network setup in live environment
1. Boot Source virtual machine from ISO
   1. Reboot virtual machine, it boots into the live environment
   1. Your virtual machine's disks are accessible but the OS isn't running (clean shutdown equivalent)
1. Configure Networking in Live Environment**
   1. Determine network interface name (may vary: eth0, ens192, etc.)
   1. Configure IP and routing:

     ```bash
     ip addr add 10.50.200.3/26 dev ens192
     ip route add 0.0.0.0/0 via 10.50.200.1
     ping 192.168.100.5  # Test connectivity to worker virtual server instance
     ```
     {: codeblock}

1. Initiate Transfer

   1. On Worker virtual server instance (start listener first):

   ```bash
   # Listen for incoming data, decompress, write to volume
   nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdb bs=16M status=progress
   ```
   {: codeblock}

   1. On Source virtual machine (in live ISO):

   ```bash
   # Read disk, compress, send to worker
   dd if=/dev/sda bs=16M | gzip | nc -N -v 192.168.100.5 8080
   ```
   {: codeblock}

   - Monitor progress on both sides
   - Transfer time depends on disk size and network bandwidth
   - Compression typically gives 2-4x improvement for OS disks
1. Repeat for Additional Disks**
   - For multi-disk virtual machines, repeat for each disk:

     ```bash
     # Worker (disk 2)
     nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdc bs=16M status=progress

     # Source (disk 2)
     dd if=/dev/sdb bs=16M | gzip | nc -N -v 192.168.100.5 8080
     ```
     {: codeblock}

1. Post-Transfer Processing
   1. Verify transfers: `fdisk -l /dev/vdb` on worker
   1. Optionally use virt-v2v for transformations: `virt-v2v-in-place -i disk /dev/vdb`
   1. Flush buffers: `blockdev --flushbufs /dev/vdb`
1. Create virtual server instance from Volumes**
   1. Detach volumes from worker
   1. Create final virtual server instance using existing boot volume (same as Method 2 step 8)
1. Shutdown Source virtual machine**
    1. After verifying virtual server instance boots successfully, shut down source virtual machine
    1. Optionally create a snapshot in VMware as a rollback point

## Design Advantages
{: #virt-sol-vpc-migration-design-method3-advantages}

The design advantages of live network transfer migration are:

- No Export Overhead
   - Eliminates the entire export step—no time spent exporting VMDKs, no export storage needed, no transferring exports to VPC.
- Efficient Network Utilization
   - Direct streaming with compression makes optimal use of available bandwidth.
- Parallel Migration Capability
   - Provision multiple worker virtual server instances and migrate multiple virtual machines concurrently, limited only by network bandwidth and worker resources.
- Maximum Flexibility
   - Easy to integrate with virt-v2v transformations, supports both VCFaaS and vCenter, works with any virtual machine regardless of disk count.
- Clean Disk State
   - Booting from ISO ensures the source OS isn't running, providing a clean, consistent disk state (similar to a cold snapshot).

## Design Constraints and Limitations
{: #virt-sol-vpc-migration-design-method3-constraints}

The following are the constratins and limitations of a live network transfer migration.

- Transit Gateway Requirement
   - Requires upfront investment in setting up and testing Transit Gateway connectivity.
- Live ISO Preparation
   - You need to prepare and test your chosen live ISO, potentially customizing it with needed tools.
- Network Bandwidth Dependency
   - Transfer speed is limited by network bandwidth between environments. Monitor and plan accordingly.
- Manual Network Configuration
   - For each source virtual machine, you need to configure networking in the live ISO environment (can be scripted).
- Not Suitable for Warm Migration
   - This is a cold migration approach—source virtual machine is offline during transfer.

## Advanced Pattern: virt-p2v Integration
{: #virt-sol-vpc-migration-design-method3-advanced}

virt-p2v Integration is more automated but requires building libguestfs with RHEL/Ubuntu hybrid components.

Red Hat's `virt-p2v` ISO is purpose-built for this use case. It provides:
- Graphical interface for selecting disks to transfer
- Built-in network connectivity to a `virt-v2v` receiver
- Automatic driver injection and OS preparation

virt-p2v Integration Process
1. Boot source virtual machine from virt-p2v ISO
1. Configure network and connect to worker virtual server instance running virt-v2v in server mode
1. virt-p2v transfers disks and virt-v2v transforms them automatically
1. Resulting volumes are ready to attach to virtual server instance

Live network transfer migration is ideal for large-scale migrations (10+ virtual machines), for scenarios where export overhead is prohibitive, and when you have the expertise to set up Transit Gateway and live ISO environments. The initial setup cost is higher, but the per-virtual machine migration efficiency is superior.
