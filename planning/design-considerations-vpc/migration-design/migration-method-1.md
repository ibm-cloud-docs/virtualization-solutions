---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Method 1: Image Import (Template-Based Migration)
{: #virt-sol-vpc-migration-design-method1}

**Use Case**: Single-disk VMs, template reuse scenarios, simple migrations

**Conceptual Model**: This is the most VMware-familiar approach. You export a VM, convert it to a format VPC understands (QCOW2), upload it to Cloud Object Storage, and create a custom image. You then boot a VSI from this custom image, much like deploying from a template in VMware.

## Process Overview
{: #virt-sol-vpc-migration-design-method1-process}

1. **Export VM from VMware**
   - From vCenter: Shut down VM, use "Actions → Template → Export OVF Template"
   - From VCFaaS: Shut down vApp, download OVA file, extract VMDK
   - This preserves thin provisioning better than datastore browser downloads

2. **Convert VMDK to QCOW2**
   ```bash
   qemu-img convert -f vmdk -O qcow2 source-vm.vmdk destination-vm.qcow2
   ```

3. **Upload to IBM Cloud Object Storage**
   - Create a COS instance and bucket if needed
   - Use web upload (for smaller files) or Aspera (for large files)
   - Configure bucket access (public read for import, or use authorized service access)

4. **Create Custom Image in VPC**
   - Navigate to VPC → Compute → Images
   - Create new image, point to COS URL
   - Select appropriate OS type (includes BYOL variants)
   - Configure encryption (provider-managed or customer-managed)

5. **Provision VSI from Custom Image**
   - Create VSI selecting your custom image as the boot source
   - Configure network, security groups, SSH keys
   - VSI boots with a boot volume that's a linked clone of your custom image

## Design Advantages
{: #virt-sol-vpc-migration-design-method1-advantages}

- **Straightforward process**: Well-documented, easy to understand
- **Template reuse**: If you have 10 web servers from the same template, you import once and provision 10 times
- **IBM-managed**: The process is native to VPC, no third-party tools required

## Design Constraints and Considerations
{: #virt-sol-vpc-migration-design-method1-constraints}

**Single Disk Limitation**: This method only handles the boot disk. If your VM has multiple disks, you'll need to:
- Use a different method for the entire VM, OR
- Use this method for the boot disk and Method 2 for secondary disks (hybrid approach)

**Image Proliferation**: Each unique VM creates a unique custom image. Unlike VMware templates, these aren't true reusable templates—they're snapshots of individual VMs. Over time, you'll have dozens or hundreds of one-off custom images in your list.

**Linked Clone Constraint**: The boot volume maintains a space-efficient linkage to the custom image. This means:
- You **cannot delete the custom image** while any VSI is using a boot volume derived from it
- Storage savings are realized through this linkage
- Breaking the linkage requires creating a new image from the boot volume and re-provisioning

**Cloud-Init First Boot**: If cloud-init is installed and configured on your image, VPC may treat the boot as a first boot, potentially:
- Resetting the root password
- Generating new SSH host keys
- Running provisioning scripts

**Design Decision**: Use Method 1 for true template scenarios (deploying multiple identical VMs from a base image) and for simple single-disk VMs where image management overhead is acceptable.
