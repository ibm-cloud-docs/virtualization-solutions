---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Method 2: Direct Volume Copy (Multi-Disk Migration)
{: #virt-sol-vpc-migration-design-method2}

**Use Case**: Multi-disk VMs, avoiding image proliferation, scenarios requiring precise control over volume configuration

**Conceptual Model**: Instead of importing a VM as an image, you create empty volumes with the exact specifications you need and directly write your VM's disk contents to them.

## Architecture Components
{: #virt-sol-vpc-migration-design-method2-architecture}

**Worker VSI**: A temporary VSI that serves as your migration workspace. It needs:
- Adequate CPU and memory to run conversion tools
- Sufficient workspace storage to hold exported VMDKs (or large ephemeral disk)
- Network connectivity to your VMware environment (if using live transfer)
- The `qemu-img` tool and optionally `libguestfs` (virt-v2v) for transformations

**Ephemeral VSI**: A short-lived VSI created solely to generate boot and data volumes with the correct configuration. You'll delete this VSI immediately but keep its volumes.

**Target Volumes**: The actual volumes that will become your migrated VM's disks.

## Process Overview
{: #virt-sol-vpc-migration-design-method2-process}

1. **Provision Worker VSI**
   - Ubuntu or RHEL instance with adequate workspace
   - Attach a large secondary volume if workspace is needed for VMDKs
   - Install required tools: `qemu-img`, `libguestfs-tools` (for virt-v2v)

2. **Create Ephemeral VSI**
   - Configure it to match your target VM (OS, boot disk size, secondary disk count/sizes)
   - **Critical**: Disable auto-delete on all volumes
   - **Critical**: Use `general-purpose` storage profile for boot volume
   - Network configuration can be throwaway
   - Note the volume sizes and order

3. **Delete Ephemeral VSI, Retain Volumes**
   - Delete the VSI via UI or CLI
   - Confirm volumes still exist and are available for attachment

4. **Attach Volumes to Worker VSI**
   - Attach them in the same order they were created
   - Note device names (e.g., /dev/vdb, /dev/vdc, etc.)
   - Verify sizes: `blockdev --getsize64 /dev/vdb`

5. **Transfer and Convert VM Disks**
   - If exported: Copy VMDK to worker VSI
   - Convert and write in one step:
     ```bash
     qemu-img convert -f vmdk -O raw source-vm-boot.vmdk /dev/vdb
     qemu-img convert -f vmdk -O raw source-vm-data.vmdk /dev/vdc
     ```
   - Optionally use virt-v2v for Windows driver injection (see Windows section below)

6. **Verify and Flush**
   - Spot check partition tables: `fdisk -l /dev/vdb`
   - Flush buffers: `blockdev --flushbufs /dev/vdb`

7. **Detach Volumes from Worker**
   - Detach all target volumes
   - They're now ready to be attached to the final VSI

8. **Create Final VSI from Existing Boot Volume**
   - Instead of selecting an image, select "existing boot volume"
   - Choose the boot volume you populated
   - Configure network, security groups, SSH key (required even though it won't be used if this is an existing VM)
   - For secondary volumes: Use CLI/API or attach after creation and reboot

9. **Post-Migration Configuration**
   - Boot VSI, access via VNC console if network config needs adjustment
   - Verify all disks are present and mounted
   - Expand boot volume partition if you resized it upward

## Design Advantages
{: #virt-sol-vpc-migration-design-method2-advantages}

**Multi-Disk Support**: Handle VMs with any number of disks (up to VPC's 12-disk limit).

**No Image Proliferation**: You're not creating a custom image for each VM. Your custom image list stays clean.

**Flexible Transformation**: Easy integration with virt-v2v for driver injection, OS tweaks, etc.

**Storage Efficiency Option**: If you import a base template as a custom image and use it as the boot volume source for your ephemeral VSI (step 2), the final boot volume inherits the linked-clone space efficiency.

## Design Constraints and Limitations
{: #virt-sol-vpc-migration-design-method2-constraints}

**Orchestration Complexity**: More steps, more moving parts. You need solid runbooks and preferably automation (Terraform, Ansible, scripts).

**Volume Attachment Limitation**: The IBM Cloud UI doesn't support attaching secondary volumes during VSI creation. You must:
- Use CLI: `ibmcloud is instance-create ... --volume-attach ...`
- Use API/Terraform for full automation
- Or create the VSI, stop it, attach volumes, then start it

**Export Overhead**: If you're exporting VMDKs from VMware, you still incur that overhead (though less than OVA export).

## Enhanced Pattern: Skip Export with Network Transfer
{: #virt-sol-vpc-migration-design-method2-enhanced}

You can combine Method 2 with network transfer techniques (detailed in Method 3) to avoid exporting VMDKs entirely. Boot your source VM from an ISO, establish network connectivity to your worker VSI, and stream the disk contents directly:

**On Worker VSI (destination)**:
```bash
nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdb bs=16M status=progress
```

**On Source VM (booted from ISO)**:
```bash
dd if=/dev/sda bs=16M | gzip | nc -N -v 192.168.100.5 8080
```

This eliminates export time and export storage requirements.

**Design Decision**: Use Method 2 for multi-disk VMs, for scenarios where you want precise control, or where avoiding custom image proliferation is important. Enhance with network transfer to improve efficiency.
