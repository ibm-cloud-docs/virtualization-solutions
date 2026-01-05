---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Windows Migration Considerations
{: #virt-sol-vpc-migration-design-windows}

Windows migrations require additional attention due to driver binding and licensing considerations.

## The Driver Challenge
{: #virt-sol-vpc-migration-design-windows-drivers}

In your VMware environment, Windows has loaded VMware paravirtualized drivers (vmxnet3 for network, pvscsi for storage, etc.) and bound them to specific hardware identifiers. When you move the disk to VPC, the hardware changes:

- Network: VMware vmxnet3 → VirtIO network adapter
- Storage (boot): VMware PVSCSI → VirtIO SCSI adapter
- Storage (data): VMware PVSCSI → VirtIO block adapter

If Windows boots and finds different hardware IDs for its boot storage controller, it will fail to boot (INACCESSIBLE_BOOT_DEVICE blue screen).

## Solution A: Sysprep Approach
{: #virt-sol-vpc-migration-design-windows-sysprep}

Microsoft's `sysprep` utility "generalizes" a Windows installation, resetting it to a first-boot state. This:
- Releases driver bindings
- Resets the Windows Security Identifier (SID)
- Removes computer-specific information
- Prepares the image for redeployment

**Process**:
1. Install VirtIO drivers in your Windows VM while still hosted in VMware:
   - Download virtio-win ISO from a RHEL VSI (`/usr/share/virtio-win`)
   - Mount ISO, run `virtio-win-gt-x64.exe` and `virtio-win-guest-tools.exe`
   - Install drivers for both OS and recovery partition

2. Run sysprep:
   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown
   ```

3. Export and migrate using any method (1, 2, or 3)

4. First boot in VPC:
   - Windows runs mini-setup wizard (OOBE)
   - Detects new hardware, loads VirtIO drivers
   - May require re-entering product key
   - May require re-joining domain

**Advantages**:
- Well-documented Microsoft process
- Works reliably for Windows deployments

**Disadvantages**:
- Resets machine identity (problematic for domain-joined servers)
- May trigger Windows re-activation
- Application-specific issues (some apps don't handle sysprep well)
- Requires OOBE completion on first boot

**Design Decision**: Use sysprep for template-based deployments or when migrating development/test VMs where identity reset is acceptable. Avoid for production domain-joined servers with complex app dependencies.

## Solution B: virt-v2v Driver Injection
{: #virt-sol-vpc-migration-design-windows-virtv2v}

The libguestfs `virt-v2v` tool can inject VirtIO drivers into a Windows installation without running sysprep. It:
- Mounts the Windows filesystem (without booting Windows)
- Injects VirtIO drivers into the driver store
- Modifies registry to force Windows to load these drivers
- Preserves machine identity, domain membership, and application state

**Prerequisites**:
- VM must be cleanly shut down (not crashed, not forced off)
- Windows version must be supported (Server 2008 R2 through 2025, Windows 7 through 11)
- virtio-win driver package (available on RHEL systems: `/usr/share/virtio-win`)

**Process**:
1. Install VirtIO drivers in Windows boot and recovery partitions (same as sysprep approach)

2. Cleanly shut down Windows VM

3. Export/transfer disk to worker VSI (Methods 1-4)

4. Run virt-v2v:
   ```bash
   virt-v2v -i disk windows-vm.img -o disk -os /target --block-driver virtio-scsi
   ```
   - `-i disk`: Input is a disk image file
   - `-o disk -os /target`: Output to directory
   - `--block-driver virtio-scsi`: Use SCSI driver for first disk (required for VPC boot volumes)

5. If writing directly to device:
   ```bash
   ln -fs /dev/vdb /target/windows-vm-sda
   virt-v2v -i disk windows-vm.img -o disk -os /target --block-driver virtio-scsi
   ```

**The RHEL/Ubuntu Challenge**:

Critical issue: The `--block-driver virtio-scsi` option is required for VPC Windows VMs (boot disk uses VirtIO SCSI, not VirtIO block), but:

- **RHEL virt-v2v**: Does not support `--block-driver virtio-scsi`
- **Ubuntu virt-v2v**: Supports `--block-driver virtio-scsi`
- **RHEL libguestfs**: Includes virtio-win drivers at `/usr/share/virtio-win`
- **Ubuntu libguestfs**: Does not include virtio-win drivers

**Workaround**:

**Option 1: Build libguestfs on Ubuntu**
```bash
# On Ubuntu worker VSI
apt-get install libguestfs-tools

# Copy virtio-win from a RHEL system
# On RHEL: tar czf virtio-win.tar.gz /usr/share/virtio-win
# Transfer to Ubuntu and extract:
tar xzf virtio-win.tar.gz -C /usr/share/

# Now virt-v2v on Ubuntu has both SCSI support and drivers
virt-v2v -i disk windows.img -o disk -os /target --block-driver virtio-scsi
```

**Option 2: Two-stage conversion**
```bash
# On RHEL worker (has drivers, no SCSI support)
virt-v2v -i disk windows.img -o disk -os /tmp

# Transfer to Ubuntu worker
scp /tmp/windows-sda ubuntu-worker:/tmp/

# On Ubuntu worker (has SCSI support)
virt-v2v -i disk /tmp/windows-sda -o disk -os /target --block-driver virtio-scsi
```

**Advantages of virt-v2v**:
- Preserves machine identity (no re-activation, no re-domain-join)
- No first-boot setup wizard
- Application state intact
- Works for production servers

**Disadvantages**:
- Requires RHEL/Ubuntu hybrid setup
- More complex than sysprep
- Requires clean shutdown (won't process crashed/forced-off VMs)

**Design Decision**: Use virt-v2v for production Windows servers where preserving identity is critical. Accept the additional tooling complexity as a trade-off for cleaner migrations.

## Windows Storage Driver Architecture in VPC
{: #virt-sol-vpc-migration-design-windows-storage}

Understanding how VPC presents storage to Windows helps troubleshoot boot issues:

**First Volume (Boot Disk)**:
- Presented as **VirtIO SCSI device**
- Requires virtio-scsi driver
- This is why `--block-driver virtio-scsi` is mandatory

**Subsequent Volumes (Data Disks)**:
- Presented as **VirtIO block devices**
- Require virtio-blk driver
- Different driver than boot disk

**Both drivers must be installed** in both:
- The running OS
- The recovery environment (WinRE)

## Recovery Environment Driver Installation
{: #virt-sol-vpc-migration-design-windows-re}

Windows Recovery Environment (WinRE) is a separate mini-Windows used for recovery operations. If it doesn't have VirtIO drivers, you can't use it for recovery after migration.

**Locating WinRE**:
```cmd
reagentc /info
```

This might report a recovery volume, but the actual WinRE image might be at:
```cmd
C:\Windows\System32\Recovery\winre.wim
```

**Installing Drivers in WinRE**:
1. Mount virtio-win ISO
2. Identify WinRE location via `reagentc /info`
3. If on a separate volume, mount it temporarily
4. Use DISM to inject drivers:
   ```cmd
   dism /mount-wim /wimfile:C:\Windows\System32\Recovery\winre.wim /index:1 /mountdir:C:\mount
   dism /image:C:\mount /add-driver /driver:E:\viostor\w10\amd64 /recurse
   dism /image:C:\mount /add-driver /driver:E:\netkvm\w10\amd64 /recurse
   dism /unmount-wim /mountdir:C:\mount /commit
   ```

**GPT Partition Considerations**:

If your Windows disk uses GPT (not MBR):
- Use `list volume` and `select volume` instead of `list partition`
- Setting volume IDs differs:
  - Data volume: `set id=ebd0a0a2-b9e5-4433-87c0-68b6b72699c7`
  - System volume: `set id=c12a7328-f81f-11d2-ba4b-00a0c93ec93b`

## Supported Windows Versions
{: #virt-sol-vpc-migration-design-windows-versions}

Red Hat's virtio-win package provides drivers for:

**Windows Server**:
- 2008 R2
- 2012, 2012 R2
- 2016, 2019, 2022, 2025

**Windows Client**:
- 7, 8, 8.1, 10, 11

Older versions (Server 2003, 2008 non-R2, Vista) are not supported.

**Design Decision Matrix for Windows**:

| Scenario | Recommended Approach |
|----------|---------------------|
| Development/test VMs | Sysprep (simple, identity reset acceptable) |
| Production standalone servers | virt-v2v (preserves identity) |
| Domain-joined production servers | virt-v2v (avoids domain re-join) |
| Template-based deployments | Sysprep (appropriate for templates) |
| Servers with licensing tied to hardware ID | virt-v2v + careful license review |
| Very old Windows (2003, 2008 non-R2) | Not supported for migration |
{: caption="Design Decision Matrix for Windows" caption-side="bottom"}
