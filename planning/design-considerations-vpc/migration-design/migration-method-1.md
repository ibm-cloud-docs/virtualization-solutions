---

copyright:
  years: 2025
lastupdated: "2026-07-21"

keywords: image import migration, template-based migration VPC, QCOW2 conversion, OVF export VMware, custom image VPC, Cloud Object Storage migration, qemu-img convert, single-disk migration, VMDK to QCOW2, VPC custom image creation


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migrate to IBM Cloud virtual servers using Image Import
{: #virt-sol-vpc-migration-design-method1}

Migrate single-disk VMware virtual machines to IBM Cloud VPC by exporting Open Virtualization Archive (OVA) or Virtual Machine Disk (VMDK) files and importing them as custom images.
{: shortdesc}

## Overview of the migration process
{: #virt-sol-vpc-migration-design-method1-process}

The following steps layout the process to migrate by using image imports.

1. Export virtual machine from VMware
   - From vCenter: Shut down virtual machine, use "Actions → Template → Export Open Virtualization Format (OVF) Template"
   - From VMware Cloud Foundation as a Service (VCFaaS): Shut down virtual application (vApp), download OVA file, extract VMDK

   The preceding approach preserves thin provisioning better than data store browser downloads

2. Convert VMDK to QCOW2

   ```bash
   qemu-img convert -f vmdk -O qcow2 source-vm.vmdk destination-vm.qcow2
   ```
   {: pre}

3. Upload to IBM Cloud Object Storage
   1. Create an {{site.data.keyword.cos_full}} instance and bucket if needed
   2. Use web upload (for smaller files) or Aspera high-speed transfer (for large files)
   3. Configure bucket access (public read for import, or use authorized service access)
4. Create Custom Image in VPC
   1. Go to VPC → Compute → Images
   2. Create new image, point to {{site.data.keyword.cos_full_notm}} URL
   3. Select the appropriate OS type (includes BYOL variants)
   4. Configure encryption (provider-managed or customer-managed)
5. Provision virtual server instance from Custom Image
   1. Create virtual server instance by selecting your custom image as the boot source
   2. Configure network, security groups, SSH keys
   3. Virtual server instance boots with a boot volume that's a linked clone of your custom image

## Image import design advantages
{: #virt-sol-vpc-migration-design-method1-advantages}

The following table lists the design advantages of image import migration.

| Design advantage | Description |
| ----------- | ------------------ |
| Process | The process is a straight-forward process, well-documented, and easy to understand. |
| Template | You can reuse the template. If you have 10 web servers from the same template, you import once and provision 10 times. |
| Managed | The process is IBM-managed. The process is native to VPC and no third-party tools are required. |
{: caption="Design advantages for image import migration method" caption-side="bottom"}

## Image import design constraints and limitations
{: #virt-sol-vpc-migration-design-method1-constraints}

The following table describes the constraints and limitations of an image import migration.

| Limitation or Constraint | Description |
| ----------- | ------------------ |
| Single disk limitation | This method handles only the boot disk. \n \n If your virtual machine has multiple disks, you need to:  \n  \n - Use a different method for the entire virtual machine \n \n OR \n \n - Use this method for the boot disk and Method 2 for secondary disks (hybrid approach) |
| Image proliferation | Each unique virtual machine creates a unique custom image. Unlike VMware templates, these aren't true reusable templates—they're snapshots of individual virtual machines. Over time, you'll have dozens or hundreds of one-off custom images in your list. |
| Linked clone constraint | The boot volume maintains a space-efficient linkage to the custom image. This means:  \n  \n - You **cannot delete the custom image** while any virtual server instance is using a boot volume derived from it \n  \n - Storage savings are realized through this linkage \n  \n - Breaking the linkage requires creating a new image from the boot volume and reprovisioning |
| Cloud-init first boot | If cloud-init is installed and configured on your image, VPC might treat the boot as a first boot, potentially:  \n  \n - Resetting the root password \n  \n - Generating new Secure Shell (SSH) host keys \n  \n - Running provisioning scripts |
{: caption="Limitations and constraints for image import migration method" caption-side="bottom"}

Use image import migration for true template scenarios (deploying multiple identical virtual machines from a base image) and for simple single-disk virtual machines where image management overhead is acceptable.
