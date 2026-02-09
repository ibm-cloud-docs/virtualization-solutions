---

copyright:
  years: 2025
lastupdated: "2026-02-09"

keywords: VSI, File Storage, Block Storage, Encryption, Migration, virtual server instance

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Copying direct volume (multi-disk method)
{: #virt-sol-vpc-migration-design-method2}

For multi-disk virtual machines where you want to avoid image proliferation and scenarios requiring precise control over volume configuration, you can migrate to a virtual server using the direct volume copy (multi-disk migration) migration method. Instead of importing a virtual machine as an image, you instead create empty volumes with the exact specifications you need. You then directly write your virtual machine's contents to those volumes.
{: shortdesc}

## Architecture Components
{: #virt-sol-vpc-migration-design-method2-architecture}

The following table lists the architecture components of a direct volume copy migration.

| Architecture components | Description |
| ----------- | ------------------ |
| Worker virtual server instance | A temporary virtual server instance that serves as your migration workspace. This virtual server instance needs the following:  \n  \n - Adequate CPU and memory to run conversion tools  \n  \n - Sufficient workspace storage to hold exported VMDKs (or large ephemeral disk)  \n  \n - Network connectivity to your VMware environment (if using live transfer)  \n  \n - The `qemu-img` tool and optionally `libguestfs` (virt-v2v) for transformations |
| Ephemeral virtual server instance | A short-lived virtual server instance created solely to generate boot and data volumes with the correct configuration. You'll delete this virtual server instance immediately but keep its volumes. |
| Target Volumes | The actual volumes that will become your migrated VM's disks. |
{: caption="Architecture components for direct volume copy migration method" caption-side="bottom"}


## Overview of the copying direct volume migration process
{: #virt-sol-vpc-migration-design-method2-process}

The following steps layout the process to migrate using direct volume copy.

1. Provision Worker virtual server instance
   1. Ubuntu or RHEL instance with adequate workspace
   1. Attach a large secondary volume if workspace is needed for VMDKs
   1. Install required tools
      -  `qemu-img`
      - `libguestfs-tools` (for virt-v2v)
1. Create Ephemeral virtual server instance
   1. Configure it to match your target VM (OS, boot disk size, secondary disk count/sizes)
   1. **Critical**: Disable auto-delete on all volumes
   1. **Critical**: Use `general-purpose` storage profile for boot volume
   1. Network configuration can be throwaway
   1. Note the volume sizes and order
1. Delete Ephemeral virtual server instance, Retain Volumes
   1. Delete the virtual server instance via UI or CLI
   1. Confirm volumes still exist and are available for attachment
1. Attach Volumes to Worker virtual server instance
   1. Attach them in the same order they were created
   1. Note device names (e.g., /dev/vdb, /dev/vdc, etc.)
   1. Verify sizes: `blockdev --getsize64 /dev/vdb`
1. Transfer and Convert VM Disks
   1. If exported: Copy VMDK to worker virtual server instance
   1. Convert and write in one step:

     ```bash
     qemu-img convert -f vmdk -O raw source-vm-boot.vmdk /dev/vdb
     qemu-img convert -f vmdk -O raw source-vm-data.vmdk /dev/vdc
     ```
     {: codeblock}

   1. Optionally use virt-v2v for Windows driver injection (see Windows section below)
1. Verify and Flush
   1. Spot check partition tables: `fdisk -l /dev/vdb`
   1. Flush buffers: `blockdev --flushbufs /dev/vdb`
1. Detach Volumes from Worker
   1. Detach all target volumes
   1. They're now ready to be attached to the final virtual server instance
1. Create Final virtual server instance from Existing Boot Volume
   1. Instead of selecting an image, select "existing boot volume"
   1. Choose the boot volume you populated
   1. Configure network, security groups, SSH key (required even though it won't be used if this is an existing VM)
   1. For secondary volumes: Use CLI/API or attach after creation and reboot
1. Post-Migration Configuration
   1. Boot virtual server instance, access via VNC console if network config needs adjustment
   1. Verify all disks are present and mounted
   1. Expand boot volume partition if you resized it upward

## Direct volume copy design advantages
{: #virt-sol-vpc-migration-design-method2-advantages}

The following table lists the design advantages of direct volume copy migration.

| Design advantage | Description |
| ----------- | ------------------ |
| Multi-Disk Support | Multi-disk support handles virtual machines with any number of disks, up to VPC's 12-disk limit. |
| No Image Proliferation | You're not creating a custom image for each virtual machine. Your custom image list stays clean. |
| Flexible Transformation | Enables easy integration with virt-v2v for driver injection, OS tweaks, etc. |
| Storage Efficiency Option | If you import a base template as a custom image and use it as the boot volume source for your ephemeral virtual server instance (step 2), the final boot volume inherits the linked-clone space efficiency. |
{: caption="Design advantages for direct volume copy migration method" caption-side="bottom"}

## Direct volume copy design constraints and limitations
{: #virt-sol-vpc-migration-design-method2-constraints}

The following table lists the constraints and limitations of a direct volume copy migration.

| Limitation or Constraint | Description |
| ----------- | ------------------ |
| Orchestration Complexity | There are more steps and moving parts. You need solid runbooks and preferably automation (Terraform, Ansible, scripts). |
| Volume attachment limitations. | The IBM Cloud UI doesn't support attaching secondary volumes during virtual server instance creation. You must do one of the following:  \n  \n - Use CLI: `ibmcloud is instance-create ... --volume-attach ...`  \n  \n - Use API/Terraform for full automation  \n  \n - Create the virtual server instance, stop it, attach volumes, then start it |
| Export overhead | If you're exporting VMDKs from VMware, you still incur that overhead (though less than OVA export). |
{: caption="Limitations and constraints for direct volume copy migration method" caption-side="bottom"}

## Skip export by using network transfer
{: #virt-sol-vpc-migration-design-method2-network-transfer}

You can combine Method 2 with network transfer techniques (detailed in Method 3) to avoid exporting VMDKs entirely. Boot your source VM from an ISO, establish network connectivity to your worker virtual server instance, and stream the disk contents directly:

1. On the worker virtual server instance (destination), issue the following command:

   ```bash
   nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdb bs=16M status=progress
   ```
   {: codeblock}

1. On the source virtual machine (booted from ISO), issue the following command:

   ```bash
   dd if=/dev/sda bs=16M | gzip | nc -N -v 192.168.100.5 8080
   ```
   {: codeblock}

This process eliminates export time and export storage requirements.


Using this process for multi-disk VMs, for scenarios where you want precise control, or where avoiding custom image proliferation is important. You can improve efficiency with network transfer too.
