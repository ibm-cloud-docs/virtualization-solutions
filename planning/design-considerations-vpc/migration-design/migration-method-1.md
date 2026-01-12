---

copyright:
  years: 2025
lastupdated: "2026-01-12"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Image Import (Template-Based Migration)
{: #virt-sol-vpc-migration-design-method1}

For single-disk virtual machines, template reuse scenarios, and simple migrations, you can migrate to a virtual server using the image import migration method. This is the most VMware-familiar approach. You export a virtual machine, convert it to a format VPC understands (QCOW2), upload it to Cloud Object Storage, and create a custom image. You then boot a VSI from this custom image, much like deploying from a template in VMware.
{: shortdesc}

## Overview of the migration process
{: #virt-sol-vpc-migration-design-method1-process}

The following steps layout the process to migrate using direct volume copy.

1. Export virtual machine from VMware
   - From vCenter: Shut down virtual machine, use "Actions → Template → Export OVF Template"
   - From VCFaaS: Shut down vApp, download OVA file, extract VMDK

   This preserves thin provisioning better than datastore browser downloads

1. Convert VMDK to QCOW2

   ```bash
   qemu-img convert -f vmdk -O qcow2 source-vm.vmdk destination-vm.qcow2
   ```
   {: pre}

1. Upload to IBM Cloud Object Storage
   1. Create a COS instance and bucket if needed
   1. Use web upload (for smaller files) or Aspera (for large files)
   1. Configure bucket access (public read for import, or use authorized service access)
1. Create Custom Image in VPC
   1. Navigate to VPC → Compute → Images
   1. Create new image, point to COS URL
   1. Select appropriate OS type (includes BYOL variants)
   1. Configure encryption (provider-managed or customer-managed)
1. Provision VSI from Custom Image
   1. Create VSI selecting your custom image as the boot source
   1. Configure network, security groups, SSH keys
   1. VSI boots with a boot volume that's a linked clone of your custom image

## Image import design advantages
{: #virt-sol-vpc-migration-design-method1-advantages}

The design advantages of image import migration are:

-  The process is straight-forward process, well-documented, and easy to understand.
- You can reuse the template reuse. If you have 10 web servers from the same template, you import once and provision 10 times.
- The process is IBM-managed. The process is native to VPC and no third-party tools are required.

## Image import design constraints and limitations
{: #virt-sol-vpc-migration-design-method1-constraints}

The following are the constratins and limitations of a image import migration.

- Single Disk Limitation
   - This method only handles the boot disk. If your virtual machine has multiple disks, you'll need to:
      - Use a different method for the entire virtual machine

      OR

      - Use this method for the boot disk and Method 2 for secondary disks (hybrid approach)

- Image Proliferation
   - Each unique virtual machine creates a unique custom image. Unlike VMware templates, these aren't true reusable templates—they're snapshots of individual virtual machines. Over time, you'll have dozens or hundreds of one-off custom images in your list.
- Linked Clone Constraint
   - The boot volume maintains a space-efficient linkage to the custom image. This means:
      - You **cannot delete the custom image** while any VSI is using a boot volume derived from it
      - Storage savings are realized through this linkage
      - Breaking the linkage requires creating a new image from the boot volume and re-provisioning
- Cloud-Init First Boot
   - If cloud-init is installed and configured on your image, VPC may treat the boot as a first boot, potentially:
      - Resetting the root password
      - Generating new SSH host keys
      - Running provisioning scripts

**Design Decision**: Use Method 1 for true template scenarios (deploying multiple identical virtual machines from a base image) and for simple single-disk virtual machines where image management overhead is acceptable.
