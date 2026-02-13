---

copyright:
  years: 2025
lastupdated: "2026-02-09"

keywords: virtual server instance, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# VDDK Direct Extraction (vCenter Only)
{: #virt-sol-vpc-migration-design-method4}

For vCenter environments only, automation-first migrations, scenarios where you need a single-command migration process, you can migrate to a virtual server using VDDK direct extraction (vCenter only). VMware's Virtual Disk Development Kit (VDDK) allows programmatic access to virtual machine disks through vCenter and vSphere APIs. The `virt-v2v` tool from the libguestfs project can leverage VDDK to connect directly to vCenter, locate a virtual machine, stream its disks to your worker virtual server instance, and perform transformations (like driver injection) in one integrated operation.
{: shortdesc}

## Architecture Components
{: #virt-sol-vpc-migration-design-method4-architecture}

The following table describes the architecture components of a VDDK Direct Extraction migration.

| Architecture components | Description |
| ----------- | ------------------ |
| VDDK | VMware's API library for accessing virtual disks. You'll download this from VMware's developer portal. |
| libguestfs with VDDK Support | The `virt-v2v` tool built with the nbdkit VDDK plugin. This requires RHEL (Ubuntu builds don't include VDDK support). |
| Worker virtual server instance | RHEL-based instance with libguestfs-tools and VDDK installed. |
| vCenter Access | Network connectivity from worker virtual server instance to vCenter and vSphere hosts, with credentials that allow virtual machine disk access. |
{: caption="Architecture components for VDDK direct extraction migration method" caption-side="bottom"}

## Overview of the VDDK Direct Extraction migration process
{: #virt-sol-vpc-migration-design-method4-process}

The following steps layout the process to migrate using VDDK Direct Extraction.

1. Provision RHEL Worker virtual server instance
   1. RHEL 8 or 9 instance
   1. Install libguestfs-tools: `dnf install libguestfs-tools`
   1. Network connectivity to vCenter
1. Install VMware VDDK
   1. Download from VMware developer portal (requires VMware account)
   1. Extract to a directory on worker virtual server instance (e.g., `/opt/vmware-vix-disklib-distrib`)
1. Gather vCenter Information
   1. vCenter hostname/IP
   1. vCenter credentials (domain\user format)
   1. Target virtual machine name
   1. ESXi host where virtual machine is running (discover via vCenter)
   1. vCenter certificate thumbprint:

     ```bash
     openssl s_client -connect vcenter.example.com:443 </dev/null 2>/dev/null | \
       openssl x509 -fingerprint -noout -in /dev/stdin | \
       cut -d= -f2
     ```
     {: codeblock}

1. Create Target Volumes
   1. Use ephemeral virtual server instance method (Method 2 steps 2-4)
   1. Attach to worker virtual server instance
1. Configure /etc/hosts (Often Required)
   1. Add vCenter and ESXi hosts if DNS isn't resolving correctly
   1. VDDK can be finicky about hostname resolution
1. Execute virt-v2v with VDDK
   1. Create password file

      ```bash
      echo 'YourPasswordHere' > /tmp/vcenter-passwd
      chmod 600 /tmp/vcenter-passwd
      ```
      {: codeblock}

   1. Run virt-v2v

      ```bash
      virt-v2v \
        -ic 'vpx://vsphere.local\%5cAdministrator\@vcenter.example.com/Datacenter/Cluster/esxi-host.example.com?no_verify=1' \
        'VM-Name' \
        -ip /tmp/vcenter-passwd \
        -o disk \
        -os /tmp \
        -it vddk \
        -io vddk-libdir=/opt/vmware-vix-disklib-distrib \
        -io vddk-thumbprint=A2:41:6A:FA:81:CA:4B:06:AE:EB:C4:1B:0F:FE:23:22:D0:E8:89:02 \
        --block-driver virtio-scsi
      ```
      {: codeblock}

      Parameters:
         - `-ic`: Input connection string (vpx:// for vCenter)
            - `\%5c` is URL encoding for backslash in domain\user
            - `?no_verify=1` skips SSL cert verification
         - `'VM-Name'`: Exact name of virtual machine in vCenter
         - `-ip`: Password file path
         - `-o disk -os /tmp`: Output to directory /tmp
         - `-it vddk`: Input transport VDDK
         - `-io vddk-libdir`: Path to VDDK installation
         - `-io vddk-thumbprint`: vCenter certificate thumbprint
         - `--block-driver virtio-scsi`: For Windows virtual machines (first disk needs SCSI driver)
1. Direct to Device with Symlink Trick

   Normally virt-v2v writes to files in a directory. To write directly to a block device:

   ```bash
   # virt-v2v will create file named VM-Name-sda
   ln -fs /dev/vdb /tmp/VM-Name-sda

   # Run virt-v2v, it writes to symlink which points to device
   virt-v2v ... (same command as above)
   ```
   {: codeblock}

1. Create virtual server instance from Volumes
   - Same as Method 2/3: detach from worker, create virtual server instance from existing volumes

## Design Advantages
{: #virt-sol-vpc-migration-design-method4-advantages}

The following table lists the design advantages of VDDK Direct Extraction migration.

| Design advantage | Description |
| ----------- | ------------------ |
| Single-Command Migration | One virt-v2v invocation does everything—extract from vCenter, convert format, inject drivers, write to destination. |
| No Export Step | Similar to live network transfer migration, eliminates the export overhead. |
| Automation-Friendly | Easily scriptable for large-scale migrations once working. |
| Integrated Transformation | Driver injection and OS preparation happen automatically. |
{: caption="Design advantages for VDDK direct extraction migration method" caption-side="bottom"}

## Design Constraints and Limitations
{: #virt-sol-vpc-migration-design-method4-constraints}

The following table lists the constraints and limitations of a VDDK Direct Extraction migration.

| Limitation or Constraint | Description |
| ----------- | ------------------ |
| vCenter Only | Does not work with VCFaaS (no vCenter API access). |
| RHEL/Ubuntu Tool Gaps | Critical challenge  \n  \n - RHEL build of libguestfs includes VDDK support (nbdkit plugin)  \n  \n - RHEL build of virt-v2v does NOT support `--block-driver virtio-scsi` (required for Windows)  \n  \n - Ubuntu build of virt-v2v supports `--block-driver virtio-scsi`  \n  \n - Ubuntu build does NOT include VDDK support  \n  \n You must either:  \n  \n - Build libguestfs yourself on Ubuntu with VDDK plugin  \n  \n OR  \n  \n - Use RHEL for VDDK extraction, write to file, transfer to Ubuntu system for virt-v2v transformation |
| Complex Prerequisites | Requires VDDK installation, vCenter API access, certificate thumbprints, precise connection strings. More setup than other methods. |
| Network Requirements | Worker virtual server instance must reach vCenter and ESXi hosts directly. May require additional firewall rules. |
{: caption="Limitations and constraints for VDDK direct extraction migration method" caption-side="bottom"}

VDDK Direct Extraction migration is powerful for large-scale vCenter migrations where the upfront investment in setup and tooling builds pays off across many virtual machines. Not recommended for small migrations or VCFaaS environments. If you choose this method, budget time for tool setup and testing.
