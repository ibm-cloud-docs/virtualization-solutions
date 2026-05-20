---

copyright:
  years: 2025
lastupdated: "2026-05-19"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Image Import (Template-Based Migration)
{: #virt-sol-vpc-migration-design-method1}

For single-disk virtual machines, template reuse scenarios, and simple migrations, you can migrate to a virtual server by using the image import migration method, which is the most VMware-familiar approach. You export a virtual machine, convert it to a format VPC understands (QCOW2), upload it to Cloud Object Storage, and create a custom image. You then boot a virtual server instance from this custom image, much like deploying from a template in VMware.
{: shortdesc}

## Overview of the migration process
{: #virt-sol-vpc-migration-design-method1-process}

The following steps layout the process to migrate by using image imports.

1. Export virtual machine from VMware
   - From vCenter: Shut down virtual machine, use "Actions → Template → Export OVF Template"
   - From VCFaaS: Shut down vApp, download OVA file, extract VMDK

   The preceding approach preserves thin provisioning better than data store browser downloads

2. Convert VMDK to QCOW2

   ```bash
   qemu-img convert -f vmdk -O qcow2 source-vm.vmdk destination-vm.qcow2
   ```
   {: pre}

3. Upload to IBM Cloud Object Storage
   1. Create an {{site.data.keyword.cos_full}} instance and bucket if needed
   2. Use web upload (for smaller files) or Aspera (for large files)
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
| Single Disk Limitation | This method handles only the boot disk. \n \n If your virtual machine has multiple disks, you need to:  \n  \n - Use a different method for the entire virtual machine \n \n OR \n \n - Use this method for the boot disk and Method 2 for secondary disks (hybrid approach) |
| Image Proliferation | Each unique virtual machine creates a unique custom image. Unlike VMware templates, these aren't true reusable templates—they're snapshots of individual virtual machines. Over time, you'll have dozens or hundreds of one-off custom images in your list. |
| Linked Clone Constraint | The boot volume maintains a space-efficient linkage to the custom image. This means:  \n  \n - You **cannot delete the custom image** while any virtual server instance is using a boot volume derived from it \n  \n - Storage savings are realized through this linkage \n  \n - Breaking the linkage requires creating a new image from the boot volume and reprovisioning |
| Cloud-Init First Boot | If cloud-init is installed and configured on your image, VPC might treat the boot as a first boot, potentially:  \n  \n - Resetting the root password \n  \n - Generating new SSH host keys \n  \n - Running provisioning scripts |
{: caption="Limitations and constraints for image import migration method" caption-side="bottom"}

Use image import migration for true template scenarios (deploying multiple identical virtual machines from a base image) and for simple single-disk virtual machines where image management overhead is acceptable.
