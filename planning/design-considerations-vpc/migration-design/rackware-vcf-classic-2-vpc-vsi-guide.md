---

copyright:
  years: 2025
lastupdated: "2026-02-06"

keywords: VSI, File Storage, Block Storage, Encryption, Migration, RackWare, RMM

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from IBM Cloud VMware VCF-Automated to VPC VSI with RackWare RMM Technical Guide
{: #virt-sol-vpc-migration-design-rmm-guide}

This guide focuses on using RackWare RMM (RackWare Management Module) to migrate IBM Cloud VCF-Automated Virtual Machines (VMs) to IBM Cloud VPC Virtual Server Instances (VSIs). There is an associated tutorial that steps through the process.

IBM Cloud VCF-Automated VM OS licenses are the responsibility of the customer and are not provided by IBM Cloud. When creating the Target VSI's be aware of the provisioning options of [Bring your own license](/docs/vpc?topic=vpc-byol-vpc-about).

The majority of VMs hosted on IBM Cloud VCF-Automated are connected to NSX overlay segments which don't have native access to the IBM Cloud Classic networks. Additionally as we want the the target VSIs to have the same IP addresses as the source VMs we have to ensure that the NSX overlay segment and the target VSI VPC subnets are isolated. We accomplish this with the following RMM features:

* Direct Sync (Host Sync) - Data transfers directly from the source VM to the target VSI without being stored on the RMM server. The RMM orchestrates the operation.
* Passthrough - The target VSI cannot reach the source VM directly, so RMM will SSH to the target VSI and from there initiate a reverse SSH tunnel to the source VM, hence, data flows: Source → RMM → Target, with the RMM acting as a network relay/proxy for the data transfer.

The RackWare RMM with a bridge server enables migration between isolated networks through:

1. Bridge NAT: Makes source accessible to RMM via network address translation
1. Reverse SSH Tunnel: Allows the target VSI to reach the source VM through the RMM server
1. Key-based Authentication**: Target VSI uses RMM's SSH key to authenticate to the source VM
1. Data sync via the Tunnel: The target VSI pulls data from the source VM through the secure SSH tunnel
1. Bootloader Installation: RMM makes target bootable with proper GRUB configuration

The process is elegant, allowing any-to-any migration while maintaining security and network isolation.

Not covered in this guide is an alternative to Direct Sync, named Staged Sync (Stage 1 + Stage 2):

* Stage 1: Copy data from source to RMM's ZFS storage pool, storing it temporarily.
* Stage 2: Copy data from RMM's storage to target.

Used when Direct Sync isn't desired or when you want to decouple source and target operations.

## Retain IP addressing
{: #virt-sol-vpc-migration-design-rmm-guide-retain}

Most VM to VPC VSI migrations are expected to require the retention of IP addressing in the migrated workloads. For the majority of VMs this is achievable, however, VPC subnets has reserved IP addresses, for example the following are reserved on 192.168.10.0/24:

* ibm-network-address:	 192.168.10.0
* ibm-default-gateway:	 192.168.10.1
* ibm-dns-address:	     192.168.10.2
* ibm-reserved-address:  192.168.10.3
* ibm-broadcast-address: 192.168.10.255

Therefore, you many need to re-IP a few VMs per subnet.

## Supported Systems
{: #virt-sol-vpc-migration-design-rmm-guide-supported}

Review the documentation listed in the references for the latest supported operating systems, but currently they include:

* RHEL 5.2 through 5.11, 6.x, 7.x. 8.x, 9.x
* Centos 5.2 through 5.11, 6.x, 7.x, 8.x
* Oracle Linux 5.6 through 5.11, 6.x, 7.x, 8.x, 9.x
* SLES 11 (including 32-bit version)
* SLES 12 (no btrfs)
* SLES 15 (no btrfs)
* Ubuntu 12 (including 32-bit version), 14, 16, 18, 20, 22, 24
* Debian 8, 9, 10, 11, 12
* AlmaLinux 8, 9
* Rocky Linux 8, 9
* Windows 2008 R2, 2012, 2016, 2019, 2022

## Architecture Components
{: #virt-sol-vpc-migration-design-rmm-guide-architecture}

This guide:

* Uses example IP addresses to assist in knowledge transfer, your IP addresses will be different.
* Focuses on a Linux migration but Microsoft Windows is similar.

```bash
IBM Cloud VMware-Automated    Bridge Server              IBM Cloud VPC
┌──────────────┐             ┌───────────────┐          ┌───────────────┐
│              │             │ens192:        │          │               │
│  Source VM   │             │192.168.10.254 │          │  RMM Server   │
│ 192.168.10.11├────────────►│               │◄─────────┤  10.68.70.11  │
│              │             │   ens224:     │          │               │
└──────────────┘             │   10.134.54.62│          └───────────────┘
                             │               │                  │
                             └───────────────┘                  │
                                                                │
                                                        ┌───────▼───────┐
                                                        │  Target VSI   │
                                                        │ 192.168.10.11 │
                                                        └───────────────┘
```
{: codeblock}

The diagram above shows a logical connection view of the components. The name of `bridge server` is misleading as it does not use layer 2 bridging but layer 3 NAT.

Source VM:
- Location: IBM Cloud VCF Automated VMware instance
- Example: VM running Ubuntu 22.04
- Real IP: 192.168.10.11 (NSX overlay segment)
- Accessible via: VM has access to the Internet via SNAT, and native access to the client network but no access to the IBM Cloud network

Bridge Server:
- Purpose: Provides layer 3 network connectivity between isolated networks
- Function: Uses SNAT to make isolated NSX overlay segment accessible to IBM Cloud VPC networks
- Interfaces:
    - `ens192`: 192.168.10.254 - Inside network (192.168.10.0/24)
    - `ens224`: 10.134.54.62 - Outside network (10.134.54.0/26)

RMM Server:
- Location: IBM Cloud VPC
- Purpose: Orchestrates migration/sync operations
- IP: 10.68.70.11
- Runs: RackWare software, hosts UI, coordinates data transfer, acts as proxy for network comminations in passthrough mode

Target VSI
- Location: IBM Cloud VPC
- Example: Virtual Server Instance (VSI)
- IP: 192.168.10.11
- Purpose: Destination for migrated data

IBM Cloud Private Static Subnet:
- Location: IBM Cloud Classic
- Example: Deploy a /30 portable subnet (4 IPs)
- IP: 10.194.177.82/30. Usable IPs: 10.194.177.82 - 10.194.177.85
- Purpose: Provides IP address for NAT that the IBM Cloud classic network routes to: 10.134.54.62 (bridge server's ens224 IP). IBM's network infrastructure knows that any traffic destined for 10.194.177.82/30 should be sent to 10.134.54.62

## Bridge Server
{: #virt-sol-vpc-migration-design-rmm-guide-bridge}

The bridge server uses `iptables` to provide NAT:

1. When RMM connects to `10.194.177.82`, the bridge translates the destination to `192.168.10.11` (the real source VM IP address).
1. When the source VM replies, it appears to come from the NAT IP `10.194.177.82`

   When RMM tries to reach `10.194.177.82`:

   1. RMM sends packet
      * SRC: 10.68.70.11
      * DST: 10.194.177.82
   1. VPC Routes to Transit Gateway which routes to Classic network which knows that:
      * "10.194.177.82 is in subnet 10.194.177.82/30"
      * "Route this subnet to 10.134.54.62"
   1. Step 3: Packet delivered to bridge server
      * Arrives on ens224 (10.134.54.62)
      * DST: 10.194.177.82 (unchanged)
   1. Bridge's iptables DNAT
      * `iptables -t nat -A PREROUTING -i ens224 -d 10.194.177.82 -j DNAT --to-destination 192.168.10.11`
   1. Packet forwarded to source VM
      * SRC: 10.68.70.11
      * DST: 192.168.10.11 (DNAT'ed)

## SSH Key Requirements
{: #virt-sol-vpc-migration-design-rmm-guide-ssh}

For the tunnel to work for data transfer, the target needs to authenticate to the source. The target VSI needs the SSH private key that matches a public key in the source VM's authorized_keys file.

Key locations:
- RMM: `/root/.ssh/id_rsa` Created as part of the manual process post RMM deployment
- Target Linux VSI: `/root/.ssh/id_rsa` Must be same key as RMM's and transferred via an RMM automated process
- Source Linux VM: `/home/rackware/.ssh/authorized_keys` Created as part of the manual process for source setup

For Windows servers, the RackWare SSHD utility is used. RackWare SSHD for Windows is a lightweight SSH server implementation packaged as an MSI installer specifically for Windows systems to enable RackWare RMM connectivity. The MSI, `RWSSHDService_x64.msi`, can be downloaded directly from the RMM server at: `https://<RMM_IP>/windows/RWSSHDService_x64.msi`

### Authentication Flow
{: #virt-sol-vpc-migration-design-rmm-guide--ssh-authentication}

1. RMM → Source VM (via bridge server)
   - Uses: RMM's SSH key
   - Authenticates as: `rackware` user
   - Purpose: Discovery, filesystem mounting, cleanup
1. RMM → Target VSI
   - Uses: RMM's SSH key
   - Authenticates as: `root` user
   - Purpose: Create tunnel, mount filesystems, data transfer
1. Target VSI → Source VM (through tunnel)
   - Uses: Copied SSH key (same as RMM's)
   - Authenticates as: `rackware` user
   - Purpose: Data transfer

## Core RackWare RMM Operations
{: #virt-sol-vpc-migration-design-rmm-guide-ops}

The following are the core RackWare RMM operations for migration.

### Discover/Examine
{: #virt-sol-vpc-migration-design-rmm-guide-ops-discover}
{: step}

Purpose: Gather information about the source VM
Process:
- User provides IP address or DNS hostname of the source VM. In our use-case we use the NAT IP address.
- RMM connects via SSH to the source server VM
- Performs standard OS queries to gather metadata:
    - CPU cores, RAM, disk configuration
    - Partitions and volume structure
    - OS version and installed packages
    - Network configuration
    - Application information
- Metadata stored in RMM's CMDB (Configuration Management Database)
- Used later for AutoProvisioning target VSIs, if required

**Key Requirement:** SSH keys must be properly configured between RMM and source server

### Capture (Store-and-Forward Approach)
{: #virt-sol-vpc-migration-design-rmm-guide-ops-capture}
{: step}

The Store-and-Forward approach is not used in this use-case, as we use the Direct Assign (Flex Sync/Host Sync) approach.

Purpose: Create a snapshot/clone of the source VM image on the RMM
Process:
- Takes LVM snapshot (Linux) or VSS snapshot (Windows) on source VM
- OS flushes application IOs to disk for consistency
- OS places bookmark in filesystem (non-disruptive)
- RMM copies Image bits from static snapshot to RMM storage
- Only used data is copied (file-level, not block-level)
- Image stored in RMM Server storage location
- Additional metadata about Image stored in CMDB

Key Features:
- Non-disruptive to origin server (production continues running)
- File-based replication (not sector/block)
- Supports compression and encryption
- Can specify include/exclude lists for selective data

### Assign
{: #virt-sol-vpc-migration-design-rmm-guide-ops-assign}
{: step}

Purpose: Deploy the captured image to a target VSI
Process:
1. AutoProvision target VSI (or use pre-provisioned server)
   - RMM uses metadata from Discover to size target appropriately
   - Provisions VSI in VPC
1. Connect to Target Uses SSH
1. Examine Target
   - Verify that the target VSI can run the source VM image
   - Understand underlying hardware
1. Boot into RackWare Microkernel
   - Deploy microkernel to target VSI
   - Insert into bootloader options
   - Boot target VSI from microkernel
1. Prepare Target VSI Disk
   - Reformat disk
   - Recreate Logical Volume structure (exact match to Origin)
   - Create partitions using best-fit algorithm
1. Transfer Image
   - Transfer image bits from RMM storage to target VSI
   - Inject necessary device drivers for new hardware
   - Optionally modify network configuration for target VSI
1. Configure and Reboot
   - Configure bootloader for actual OS
   - Reboot into replicated OS
1. Verification
   - Verify target VSI boots properly
   - Verify correct networking
   - Verify data replicated correctly
   - SSH into replicated server using same credentials as origin

### Direct Assign (Also called Flex Sync or Host Sync)
{: #virt-sol-vpc-migration-design-rmm-guide-ops-direct}
{: step}

This is the approach we use in this use-case.

Purpose: Replicate directly from source VM to target VSI without intermediate storage
Process:
1. Combines Capture + Assign into single operation
1. Performs all Discover functions
1. Provisions target server (or uses existing)
1. Prepares target server
1. Replicates directly from source VM to target VSI
1. Network connection can still route through RMM (no direct Source-to-Target connection required)

### Sync (Delta Synchronization)
{: #virt-sol-vpc-migration-design-rmm-guide-ops-sync}
{: step}

Purpose: Update target with only changed data from the source VM
Process:
1. Takes snapshot on source VM (LVM or VSS)
1. Calculates delta (only changed files)
1. Transfers only changed data to Target
1. Updates either:
   - Captured Image on RMM storage, OR
   - Running Target server, OR
   - Both (Image + Target server)

Sync Options

* Stage I Sync
    - Origin → RMM Storage (captured image)
    - Updates the stored image only
* Stage II Sync
    - RMM Storage → Target Server
    - Updates running target from stored image
* RMM Passthrough
    - Origin → RMM → Target
    - Data flows through RMM but doesn't persist
    - No storage on RMM required
* Direct Sync
    - Origin → Target (direct connection)
* Selective Sync*
    - Sync only specific files/directories
    - Uses include/exclude lists
* Drive/Directory Mapping
    - Map specific origin paths to different target paths

Sync Engines:

* RWSync (Default):
    - Agentless
    - Network outage tolerant
    - Handles massive concurrent updates
    - Includes final checksum
    - Slower than TNG
* TNG (Advanced):
    - Requires delta file tracker installation
    - Much faster and more efficient
    - For large servers, high update rates, aggressive RPO
    - Must whitelist in antivirus
    - More sensitive to network outages
    - Doesn't support remote NFS/CIFS mounts

## The RackWare Microkernel Boot Process
{: #virt-sol-vpc-migration-design-rmm-guide-microkernel}

During the Assign operation, the RMM boots the target VSI into a RackWare microkernel, and the boot disk is reformatted, ensuring the filesystems, their types and sizes will match the source VM. The RMM does a best fit attempt based on the target disk layout and the partitions to fit in based on what the source has.

### Microkernel Deployment
{: #virt-sol-vpc-migration-design-rmm-guide-microkernel-deployment}

When RackWare prepares a target VSI to receive the replicated image:

- RMM deploys a RackWare microkernel onto the target VSI
- This microkernel "inserts itself in the Bootloader options" of the platform OS
- The microkernel boots from the bootloader (GRUB on Linux)

The microkernel can be considered as a LiveCD, and is a minimal bootable environment that:

- Houses the required drivers for the target hardware
- Allows the RMM to communicate with the target server
- Enables disk reformatting and partition creation
- Facilitates the actual data transfer from origin to target
- Configures the system for the new hardware environment

During the Assign phase, RackWare:

1. Boots the target VSI into the microkernel (via GRUB entry)
1. Reformats the disk while in microkernel
1. Recreates partition structure matching the source VM
1. Creates Logical Volumes to create the LVM structure
1. Transfers the source VM data to the target VSI
1. Injects necessary drivers for the target hardware
1. Configures GRUB to boot the actual OS
1. Reboots into the replicated OS

RackWare modifies the GRUB configuration to:

- Add a temporary microkernel boot entry during the Assign process
- Configure the correct boot parameters for the replicated OS
- Set the default boot option to the replicated OS after transfer completes
- Handle any kernel parameters needed for the new hardware

For Windows Systems:
- Windows Boot Manager is used instead of GRUB
- Same microkernel concept applies
- Microkernel inserts into Windows boot configuration
- After replication, boot configuration points to replicated Windows OS

The microkernel environment allows RackWare to:
- Inject appropriate storage drivers
- Inject network drivers
- Configure device drivers for new hardware
- Ensure the replicated OS can boot on different hardware

The replication process for delta syncs, subsequent syncs, works the same way as the initial sync. The delta sync is performed after rebooting the target server into the RackWare microkernel. Once the delta sync is complete the target VSI will be booted back into the host OS,

This microkernel approach is a key differentiator for RackWare as it allows them to handle the complexity of cross-platform, cross-hypervisor, and physical-to-virtual migrations by having complete control over the target environment during the critical transfer and configuration phase.

## RackWare Passthrough with Direct Sync Process
{: #virt-sol-vpc-migration-design-rmm-guide-passthrough}

The process is as follows:

1.  RMM Discovers the Source VM

   ```bash
   RMM → Bridge (10.194.177.82) → Source VM (192.168.10.11)
   ```
   {: pre}

   - RMM connects via SSH to `10.194.177.82` (NAT IP)
   - Bridge translates to `192.168.10.11`
   - RMM gathers metadata: OS, filesystems, disk layout, etc.
1.  RMM Discovers the Target VSI

   ```bash
   RMM → Target VSI (192.168.10.11)
   ```
   {: codeblock}

   - RMM connects via SSH to `192.168.10.11`
   - Examines target hardware and capabilities
1. RMM Creates Reverse SSH Tunnel to Target VSI

   The RMM automation creates a reverse SSH tunnel between the target VSI to the source VM via the RMM server:

   ```bash
   Target VSI (192.168.10.11)
     │
     └─ localhost:23 ──[SSH Tunnel]──► RMM ──► Bridge ──► Source VM:22
   ```
   {: codeblock}

   1. RMM initiates SSH connection to the target VSI
   1. Creates a listening port (23) ON the target
   1. Any connection to `localhost:23` on target gets forwarded:
         - Through the SSH tunnel back to RMM
         - RMM forwards to `10.194.177.82:22` (source via bridge)
         - Bridge DNATs to actual source

       Visual Flow:

      ```bash
      Target:       [App tries localhost:23]
                              ↓
                      [SSH tunnel to RMM]
                              ↓
       RMM: [Receives and forwards to 10.194.177.82:22]
                              ↓
        Bridge: [DNAT: 10.194.177.82 → 192.168.10.11]
                              ↓
      Source:   [Receives connection on port 22]
      ```
      {: codeblock}

1. Mount Filesystems

   RMM mounts filesystems on both source and target using commands similar to the examples below:

   On Source VM (via bridge):

   ```bash
   # RMM executes on source
   mount --bind / /mnt/rackware/tmp.xxxxx
   ```
   {: codeblock}

   On Target VSI (direct):

   ```bash
   # RMM executes on target
   mount /dev/vda2 /mnt/rackware/tmp.yyyyy
   ```
   {: codeblock}

1.  Data Transfer

   RMM executes data transfer on the target VSI to pull data from source VM:

1.  GRUB Installation

   RMM installs and configures GRUB on target for UEFI boot:

   - Copies GRUB bootloader files
   - Generates `grub.cfg` with correct kernel parameters
   - Creates UEFI boot entries
   - Configures for new hardware drivers

1.  Unmount Filesystems

   ```bash
   # On both source and target
   umount /mnt/rackware/tmp.xxxxx
   ```
   {: codeblock}

1. Close Tunnel

   1. RMM closes the SSH tunnel to target
   1. Port 23 stops listening on target

1. Remove Utilities

   ```bash
   # RMM removes temporary files from source and target
   rm -rf /var/tmp/rackware/
   ```
   {: codeblock}

## References
{: #virt-sol-vpc-migration-design-rmm-guide-references}

* [RackWare RMM users Guide for IBM Cloud](https://rackware.attachments9.freshdesk.com/data/helpdesk/attachments/production/5193588906/original/Rackware%20RMM%20Users%20Guide%20for%20IBM%20Cloud%20v2.2.pdf?response-content-type=application%2Fpdf&Expires=1765450579&Signature=IaPrBrAPnhNyJO0aufGdguvyaqFe05gyogvm6~ThDJFVmbnjdDQ~EqGwlURZApSykyAyV7oopPoQSlGDKU83ytjBiFSlmwvmWyzyepUD9pbquPVgL9ytuhvbcp8K1ODUKuyanLWtcqfTEqTuE453zXXd78ST8uGCCeJcGx3LbluMNB6ZY4EEem7uvp6biJ14M9OMLrGYmiAOwjVZ3C~MJKbmeXOQldMPIfHr9ZmQwPx9BXnW5Bbw3dyQTuNPPM4vC~Xb-KpneAuAWC4ays0YNzV17TGmSfbfOYVTaHF0JNqIJkdtvK4tafiFM8sr~Pk1eADAIBZRA9JTmC55VhCYbA__&Key-Pair-Id=APKAJ7JARUX3F6RQIXLA)
* [RackWare RMM Getting Started for IBM Cloud](https://rackware.attachments9.freshdesk.com/data/helpdesk/attachments/production/5194762642/original/RackWare%20RMM%20Getting%20Started%20for%20IBM%20Cloud-1.1.pdf?response-content-type=application%2Fpdf&Expires=1765450748&Signature=eQVX8qOpyYbgW2FAcYi9JSle5z56iBrermpYesqzCJlbOOGssqgf2Sh6RK73Bh8c84n4-82acsKAnlU6xQToBynvt9JEy5YSA1~SEAh1-JAPDZRjneyKpzS--MgyRKrYIFk5emAefyExDEGlRUPnNseQqExaFrmkUtmPKZVxv22cR6WkPQCgCfvT6VVs0EfDUZs~GmvgbvzYhWuAz7rRi6vgQloSrqXy8~X41XzLPyROOGuHU8iBK2oy1dxU1PA6tDIQz07IFvZAsxu6lYaaQv1M6gstmEYpF7233ujCKVDsAwAQ6QXIbAxnbA1t1hRSB7ccuf121BtkVbGc7Dv8Kw__&Key-Pair-Id=APKAJ7JARUX3F6RQIXLA)
