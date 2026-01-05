---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Method 4: VDDK Direct Extraction (vCenter Only)
{: #virt-sol-vpc-migration-design-method4}

**Use Case**: vCenter environments only, automation-first migrations, scenarios where you need a single-command migration process

**Conceptual Model**: VMware's Virtual Disk Development Kit (VDDK) allows programmatic access to virtual machine disks through vCenter and vSphere APIs. The `virt-v2v` tool from the libguestfs project can leverage VDDK to connect directly to vCenter, locate a VM, stream its disks to your worker VSI, and perform transformations (like driver injection) in one integrated operation.

## Architecture Components
{: #virt-sol-vpc-migration-design-method4-architecture}

**VDDK**: VMware's API library for accessing virtual disks. You'll download this from VMware's developer portal.

**libguestfs with VDDK Support**: The `virt-v2v` tool built with the nbdkit VDDK plugin. This requires RHEL (Ubuntu builds don't include VDDK support).

**Worker VSI**: RHEL-based instance with libguestfs-tools and VDDK installed.

**vCenter Access**: Network connectivity from worker VSI to vCenter and vSphere hosts, with credentials that allow VM disk access.

## Process Overview
{: #virt-sol-vpc-migration-design-method4-process}

1. **Provision RHEL Worker VSI**
   - RHEL 8 or 9 instance
   - Install libguestfs-tools: `dnf install libguestfs-tools`
   - Network connectivity to vCenter

2. **Install VMware VDDK**
   - Download from VMware developer portal (requires VMware account)
   - Extract to a directory on worker VSI (e.g., `/opt/vmware-vix-disklib-distrib`)

3. **Gather vCenter Information**
   - vCenter hostname/IP
   - vCenter credentials (domain\user format)
   - Target VM name
   - ESXi host where VM is running (discover via vCenter)
   - vCenter certificate thumbprint:
     ```bash
     openssl s_client -connect vcenter.example.com:443 </dev/null 2>/dev/null | \
       openssl x509 -fingerprint -noout -in /dev/stdin | \
       cut -d= -f2
     ```

4. **Create Target Volumes**
   - Use ephemeral VSI method (Method 2 steps 2-4)
   - Attach to worker VSI

5. **Configure /etc/hosts (Often Required)**
   - Add vCenter and ESXi hosts if DNS isn't resolving correctly
   - VDDK can be finicky about hostname resolution

6. **Execute virt-v2v with VDDK**
   
   **Create password file**:
   ```bash
   echo 'YourPasswordHere' > /tmp/vcenter-passwd
   chmod 600 /tmp/vcenter-passwd
   ```

   **Run virt-v2v**:
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

   **Parameters Explained**:
   - `-ic`: Input connection string (vpx:// for vCenter)
     - `\%5c` is URL encoding for backslash in domain\user
     - `?no_verify=1` skips SSL cert verification
   - `'VM-Name'`: Exact name of VM in vCenter
   - `-ip`: Password file path
   - `-o disk -os /tmp`: Output to directory /tmp
   - `-it vddk`: Input transport VDDK
   - `-io vddk-libdir`: Path to VDDK installation
   - `-io vddk-thumbprint`: vCenter certificate thumbprint
   - `--block-driver virtio-scsi`: For Windows VMs (first disk needs SCSI driver)

7. **Direct to Device with Symlink Trick**
   
   Normally virt-v2v writes to files in a directory. To write directly to a block device:
   
   ```bash
   # virt-v2v will create file named VM-Name-sda
   ln -fs /dev/vdb /tmp/VM-Name-sda
   
   # Run virt-v2v, it writes to symlink which points to device
   virt-v2v ... (same command as above)
   ```

8. **Create VSI from Volumes**
   - Same as Method 2/3: detach from worker, create VSI from existing volumes

## Design Advantages
{: #virt-sol-vpc-migration-design-method4-advantages}

**Single-Command Migration**: One virt-v2v invocation does everything—extract from vCenter, convert format, inject drivers, write to destination.

**No Export Step**: Like Method 3, eliminates the export overhead.

**Automation-Friendly**: Easily scriptable for large-scale migrations once working.

**Integrated Transformation**: Driver injection and OS preparation happen automatically.

## Design Constraints and Limitations
{: #virt-sol-vpc-migration-design-method4-constraints}

**vCenter Only**: Does not work with VCFaaS (no vCenter API access).

**RHEL/Ubuntu Tool Gaps**: Critical challenge—
- RHEL build of libguestfs includes VDDK support (nbdkit plugin)
- RHEL build of virt-v2v does NOT support `--block-driver virtio-scsi` (required for Windows)
- Ubuntu build of virt-v2v supports `--block-driver virtio-scsi`
- Ubuntu build does NOT include VDDK support

**Resolution**: You must either:
- Build libguestfs yourself on Ubuntu with VDDK plugin, OR
- Use RHEL for VDDK extraction, write to file, transfer to Ubuntu system for virt-v2v transformation

**Complex Prerequisites**: Requires VDDK installation, vCenter API access, certificate thumbprints, precise connection strings. More setup than other methods.

**Network Requirements**: Worker VSI must reach vCenter and ESXi hosts directly. May require additional firewall rules.

**Design Decision**: Method 4 is powerful for large-scale vCenter migrations where the upfront investment in setup and tooling builds pays off across many VMs. Not recommended for small migrations or VCFaaS environments. If you choose this method, budget time for tool setup and testing.
