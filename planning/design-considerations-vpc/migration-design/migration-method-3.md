---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Method 3: Live Network Transfer (Recommended for Scale)
{: #virt-sol-vpc-migration-design-method3}

**Use Case**: Large-scale migrations, minimizing downtime, maximum efficiency, scenarios where exporting VMDKs is impractical.

**Conceptual Model**: Instead of shutting down your VM, exporting it, transferring the export, and converting it, you boot the source VM from a live ISO (like a rescue disk), read its disks from within this live environment, and stream them directly over the network to your worker VSI in VPC.

## Architecture Components
{: #virt-sol-vpc-migration-design-method3-architecture}

**Transit Gateway**: Connects your VMware environment (Classic, NSX, or VCFaaS) to your VPC. This provides the Layer 3 routing necessary for your source VM (in VMware) to communicate with your worker VSI (in VPC).

**Worker VSI**: Similar to Method 2, but here it's purely a receiver—listening on a network port for incoming disk data.

**Live ISO**: A bootable ISO that provides:
- Network configuration tools
- Disk reading utilities (`dd`)
- Compression tools (`gzip`, `pigz`)
- Network transfer tools (`netcat`, `socat`, `ssh`)

**Boot Source Options**:
- **Ubuntu Install ISO**: Has a "Enter shell" option in the help menu, includes most needed tools
- **TinyCore Linux**: Extremely small but requires package installation for ssh/qemu
- **virt-p2v ISO**: Purpose-built by Red Hat for P2V migrations, integrates with virt-v2v on receiver
- **G4L (Ghost for Linux)**: Imaging-focused live Linux

## Process Overview
{: #virt-sol-vpc-migration-design-method3-process}

1. **Provision Transit Gateway**
   - Create Transit Gateway in IBM Cloud
   - Connect it to your VMware environment:
     - Classic: Direct connection to Classic account
     - NSX: GRE tunnels to NSX edges
     - VCFaaS: GRE tunnels to VCFaaS edges
   - Configure routing between VMware networks and VPC subnets
   - **Test connectivity thoroughly** before first migration

2. **Provision Worker VSI in VPC**
   - Ubuntu or RHEL with adequate resources
   - Create target volumes using ephemeral VSI technique (Method 2 steps 2-4)
   - Attach target volumes to worker
   - Install netcat: `apt-get install netcat` or `yum install nc`

3. **Prepare Source VM for Transfer**
   - Attach live ISO to VM in vCenter/VCFaaS
   - Configure VM boot order to boot from CD/ISO first
   - Note current IP configuration for network setup in live environment

4. **Boot Source VM from ISO**
   - Reboot VM, it boots into the live environment
   - Your VM's disks are accessible but the OS isn't running (clean shutdown equivalent)

5. **Configure Networking in Live Environment**
   - Determine network interface name (may vary: eth0, ens192, etc.)
   - Configure IP and routing:
     ```bash
     ip addr add 10.50.200.3/26 dev ens192
     ip route add 0.0.0.0/0 via 10.50.200.1
     ping 192.168.100.5  # Test connectivity to worker VSI
     ```

6. **Initiate Transfer**
   
   **On Worker VSI (start listener first)**:
   ```bash
   # Listen for incoming data, decompress, write to volume
   nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdb bs=16M status=progress
   ```

   **On Source VM (in live ISO)**:
   ```bash
   # Read disk, compress, send to worker
   dd if=/dev/sda bs=16M | gzip | nc -N -v 192.168.100.5 8080
   ```

   - Monitor progress on both sides
   - Transfer time depends on disk size and network bandwidth
   - Compression typically gives 2-4x improvement for OS disks

7. **Repeat for Additional Disks**
   - For multi-disk VMs, repeat for each disk:
     ```bash
     # Worker (disk 2)
     nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdc bs=16M status=progress
     
     # Source (disk 2)
     dd if=/dev/sdb bs=16M | gzip | nc -N -v 192.168.100.5 8080
     ```

8. **Post-Transfer Processing**
   - Verify transfers: `fdisk -l /dev/vdb` on worker
   - Optionally use virt-v2v for transformations: `virt-v2v-in-place -i disk /dev/vdb`
   - Flush buffers: `blockdev --flushbufs /dev/vdb`

9. **Create VSI from Volumes**
   - Detach volumes from worker
   - Create final VSI using existing boot volume (same as Method 2 step 8)

10. **Shutdown Source VM**
    - After verifying VSI boots successfully, shut down source VM
    - Optionally create a snapshot in VMware as a rollback point

## Design Advantages
{: #virt-sol-vpc-migration-design-method3-advantages}

**No Export Overhead**: Eliminates the entire export step—no time spent exporting VMDKs, no export storage needed, no transferring exports to VPC.

**Efficient Network Utilization**: Direct streaming with compression makes optimal use of available bandwidth.

**Parallel Migration Capability**: Provision multiple worker VSIs and migrate multiple VMs concurrently, limited only by network bandwidth and worker resources.

**Maximum Flexibility**: Easy to integrate with virt-v2v transformations, supports both VCFaaS and vCenter, works with any VM regardless of disk count.

**Clean Disk State**: Booting from ISO ensures the source OS isn't running, providing a clean, consistent disk state (similar to a cold snapshot).

## Design Constraints and Limitations
{: #virt-sol-vpc-migration-design-method3-constraints}

**Transit Gateway Requirement**: Requires upfront investment in setting up and testing Transit Gateway connectivity.

**Live ISO Preparation**: You need to prepare and test your chosen live ISO, potentially customizing it with needed tools.

**Network Bandwidth Dependency**: Transfer speed is limited by network bandwidth between environments. Monitor and plan accordingly.

**Manual Network Configuration**: For each source VM, you need to configure networking in the live ISO environment (can be scripted).

**Not Suitable for Warm Migration**: This is a cold migration approach—source VM is offline during transfer.

## Advanced Pattern: virt-p2v Integration
{: #virt-sol-vpc-migration-design-method3-advanced}

Red Hat's `virt-p2v` ISO is purpose-built for this use case. It provides:
- Graphical interface for selecting disks to transfer
- Built-in network connectivity to a `virt-v2v` receiver
- Automatic driver injection and OS preparation

**Process**:
1. Boot source VM from virt-p2v ISO
2. Configure network and connect to worker VSI running virt-v2v in server mode
3. virt-p2v transfers disks and virt-v2v transforms them automatically
4. Resulting volumes are ready to attach to VSI

This is more automated but requires building libguestfs with RHEL/Ubuntu hybrid components.

**Design Decision**: Method 3 is ideal for large-scale migrations (10+ VMs), for scenarios where export overhead is prohibitive, and when you have the expertise to set up Transit Gateway and live ISO environments. The initial setup cost is higher, but the per-VM migration efficiency is superior.
