---

copyright:
  years: 2025, 2026
lastupdated: "2026-10-06"

keywords: direct volume copy migration VPC, multi-disk VMware migration IBM Cloud, virt-v2v driver injection VPC, worker VM migration IBM Cloud, ephemeral instance VPC, qemu-img raw conversion VPC, libguestfs tools migration, VPC volume attachment migration, IBM Cloud VPC volume copy, multi-disk VM migration method


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# {{site.data.keyword.cloud_notm}} VPC: Multi-disk VMware VM migration by direct volume copy
{: #virt-sol-vpc-migration-design-method2}

Migrate multi-disk VMware VMs to {{site.data.keyword.cloud_notm}} VPC virtual servers by using direct volume copy with qemu-img or virt-v2v for full disk-level migration control.
{: shortdesc}

## Architecture components
{: #virt-sol-vpc-migration-design-method2-architecture}

The following table lists the architecture components of a direct volume copy migration.

| Architecture components | Description |
| ----------- | ------------------ |
| Worker virtual server instance | A temporary virtual server instance that serves as your migration workspace. This virtual server instance needs the following prerequisites:  \n  \n - Adequate central processing unit (CPU) and memory to run conversion tools \n  \n - Sufficient workspace storage to hold exported virtual machine disk (VMDK) files (or large ephemeral disk) \n  \n - Network connectivity to your VMware environment (if you use live transfer) \n  \n - The `qemu-img` tool and optionally `libguestfs` (virt-v2v) for transformations |
| Ephemeral virtual server instance | A short-lived virtual server instance created solely to generate boot and data volumes with the correct configuration. You delete this virtual server instance immediately but keep its volumes. |
| Target Volumes | The actual volumes that become your migrated VM's disks. |
{: caption="Architecture components for direct volume copy migration method" caption-side="bottom"}


## Overview of the copying direct volume migration process
{: #virt-sol-vpc-migration-design-method2-process}

Direct volume copy is a volume-level migration approach that converts VMDK files to raw format and writes them directly to pre-provisioned VPC Block Storage volumes.

Before you begin, ensure that you have provisioned an ephemeral VPC worker instance, confirmed network connectivity or transfer storage for VMDK files, and installed `qemu-img` and `libguestfs-tools`.

1. Provision worker virtual server instance
   1. Ubuntu or RHEL instance with adequate workspace
   1. Attach a large secondary volume if workspace is needed for VMDKs
   1. Install the required tools
      -  `qemu-img`
      - `libguestfs-tools` (for virt-v2v)
1. Create Ephemeral virtual server instance
   1. Configure it to match your target VM (OS, boot disk size, secondary disk count/sizes)
   1. **Critical**: Disable auto-delete on all volumes
   1. **Critical**: Use `general-purpose` storage profile for boot volume
   1. Network configuration can be throwaway
   1. Note the volume sizes and order
1. Delete Ephemeral virtual server instance, Retain Volumes
   1. Delete the virtual server instance through UI or CLI
   1. Confirm that volumes still exist and are available for attachment
1. Attach Volumes to worker virtual server instance
   1. Attach them in the same order that they were created
   1. Note device names (for example, /dev/vdb, /dev/vdc, and so on)
   1. Verify sizes: `blockdev --getsize64 /dev/vdb`
1. Transfer and convert VM Disks
   1. If exported: Copy VMDK to worker virtual server instance
   1. Convert and write in one step:

     ```bash
     qemu-img convert -f vmdk -O raw source-vm-boot.vmdk /dev/vdb
     qemu-img convert -f vmdk -O raw source-vm-data.vmdk /dev/vdc
     ```
     {: codeblock}

   1. Optionally use virt-v2v for Windows driver injection (see the following Windows section)
1. Verify and Flush
   1. Spot check partition tables: `fdisk -l /dev/vdb`
   1. Flush buffers: `blockdev --flushbufs /dev/vdb`
1. Detach Volumes from worker
   1. Detach all target volumes
   1. They're now ready to be attached to the final virtual server instance
1. Create Final virtual server instance from Existing Boot Volume
   1. Instead of selecting an image, select "existing boot volume"
   1. Choose the boot volume that you populated
   1. Configure network, security groups, SSH key (required even though it is not used if it is an existing VM)
   1. For secondary volumes: Use CLI/API or attach after creation and restart
1. Post-Migration Configuration
   1. Boot virtual server instance, access through VNC console if network config needs adjustment
   1. Verify that all disks are present and mounted
   1. Expand boot volume partition if you resized it upward

## Direct volume copy design advantages
{: #virt-sol-vpc-migration-design-method2-advantages}

The following table lists the design advantages of direct volume copy migration.

| Design advantage | Description |
| ----------- | ------------------ |
| Multi-disk support | Multi-disk support handles virtual machines with any number of disks, up to VPC's 12-disk limit. |
| No image proliferation | You're not creating a custom image for each virtual machine. Your custom image list stays clean. |
| Flexible transformation | Enables easy integration with virt-v2v for driver injection, OS tweaks, and so on. |
| Storage efficiency option | If you import a base template as a custom image and use it as the boot volume source for your ephemeral virtual server instance (step 2), the final boot volume inherits the linked-clone space efficiency. |
{: caption="Design advantages for direct volume copy migration method" caption-side="bottom"}

## Direct volume copy design constraints and limitations
{: #virt-sol-vpc-migration-design-method2-constraints}

The following table lists the constraints and limitations of a direct volume copy migration.

| Limitation or Constraint | Description |
| ----------- | ------------------ |
| Orchestration complexity | There are more steps and moving parts. You need solid runbooks and preferably automation (Terraform, Ansible, scripts). |
| Volume attachment limitations | The {{site.data.keyword.cloud_notm}} user interface (UI) doesn't support attaching secondary volumes during virtual server instance creation. You must do one of the following:  \n  \n - Use command-line interface (CLI): `ibmcloud is instance-create ... --volume-attach ...`  \n  \n - Use API/Terraform for full automation \n  \n - Create the virtual server instance, stop it, attach volumes, then start it |
| Export overhead | If you're exporting VMDKs from VMware, you still incur that overhead (though less than OVA export). |
{: caption="Limitations and constraints for direct volume copy migration method" caption-side="bottom"}

## Skip export by using network transfer
{: #virt-sol-vpc-migration-design-method2-network-transfer}

Network streaming transfers live disk blocks across a private network directly into target VPC volumes without creating intermediate export files. You can combine Method 2 with network transfer techniques (detailed in Method 3) to avoid exporting VMDKs entirely.

Before executing the stream, verify network routing between the source host and the VPC worker instance, ensure port 8080 is accessible in security group rules, and boot the source VM from a live Linux ISO.

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

## Related topics
{: #virt-sol-vpc-migration-design-method2-related}

To explore alternative migration methods and operational troubleshooting, review the following resources:

- [VPC migration methods overview](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-methods)
- [Method 1: Image import](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method1)
- [Method 3: Live network transfer](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method3)
- [Method 4: VDDK extraction](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method4)
- [Linux migration considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-linux)
- [Windows migration considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-windows)
- [Frequently asked questions](/docs/virtualization-solutions?topic=virtualization-solutions-virtualization-solutions-faqs)
