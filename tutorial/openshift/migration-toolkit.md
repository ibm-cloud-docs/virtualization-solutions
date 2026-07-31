---

copyright:
  years: 2026
lastupdated: "2026-07-31"

keywords: VMware vSphere migration OpenShift Virtualization, Migration Toolkit for Virtualization MTV, MTV Operator installation IBM Cloud, warm migration cold migration MTV, VDDK image configuration MTV, VMware to OpenShift VM migration, IBM Cloud VPC bare metal migration, MTV migration plan creation, MTV troubleshooting ForkliftController, preserve static IP migration OpenShift, LUKS BitLocker encrypted VM migration, concurrent VM migration performance tuning

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 60m
use-case: ApplicationModernization
industry: Software and platform applications
compliance: HIPAA

---

{{site.data.keyword.attribute-definition-list}}

# Migrate VMware VMs to Red Hat OpenShift Virtualization by using MTV
{: #vsphere-openshift-migration}
{: #tutorial-rove-migration-toolkit}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}
{: toc-use-case="ApplicationModernization"}
{: toc-industry="Software and platform applications"}
{: toc-compliance="HIPAA"}

Migrate VMware vSphere virtual machines (VMs) to {{site.data.keyword.redhat_openshift_notm}} Virtualization on {{site.data.keyword.cloud_notm}} by using the Migration Toolkit for Virtualization (MTV). Configure VDDK images, install the MTV Operator, create migration plans, run warm and cold migrations, tune performance for large-scale workloads, and resolve common migration errors.
{: shortdesc}

## Overview
{: #mtv-overview}

MTV migrates VMs to {{site.data.keyword.redhat_openshift_notm}} Virtualization running on {{site.data.keyword.redhat_openshift_notm}}.

MTV supports migration from the following VMware vSphere source providers:

- VMware vSphere
- Open Virtual Appliances (OVAs) created by VMware vSphere

## Migration prerequisites and environment requirements
{: #prerequisites-requirements}

### Software compatibility
{: #software-compatibility}

To successfully migrate, install compatible versions of {{site.data.keyword.redhat_openshift_notm}} and {{site.data.keyword.redhat_openshift_notm}} Virtualization.

For MTV 2.9, the compatible software versions include:

- {{site.data.keyword.redhat_openshift_notm}}: 4.19, 4.18, 4.17
- {{site.data.keyword.redhat_openshift_notm}} Virtualization: 4.19, 4.18, 4.17
- VMware vSphere: 6.5 or later

### Network requirements
{: #network-requirements}

Network prerequisites apply across all migrations:

Network stability
:   The network connections between the source environment and the {{site.data.keyword.redhat_openshift_notm}} Virtualization cluster must be reliable and uninterrupted.

Configuration integrity
:   Do not change IP addresses, virtual local area networks (VLANs), or other network configuration settings during migration, as the VM's MAC addresses are preserved.

Network address uniqueness
:   If you are migrating workloads from {{site.data.keyword.cloud_notm}} classic infrastructure, connect your virtual private cloud (VPC) to the classic infrastructure by using {{site.data.keyword.cloud_notm}} Transit Gateway. When you create the VPC to host your workloads, ensure that the network address prefixes do not overlap with prefixes that are used by other VPCs connected to your {{site.data.keyword.cloud_notm}} classic infrastructure. When you create your VPC, deselect the **Create a default prefix for each zone** checkbox. The default prefix remains the same across VPCs and can introduce routing conflicts. After the VPC is created, you can configure address prefixes and subnets as needed.

Destination networks
:   If multiple source and destination networks are mapped, create a network attachment definition for each additional destination network.

Required ports
:   Firewalls must permit traffic over specific ports based on the source provider:
    - VMware vSphere: Transmission Control Protocol (TCP) ports 443 (for inventory and disk transfer authentication) and 902 (for disk transfer data copy) from {{site.data.keyword.redhat_openshift_notm}} nodes to VMware vCenter&reg;/ESXi hosts.
    - OVA: TCP port 2049 (for Network File System (NFS) service) and TCP or User Datagram Protocol (UDP) port 111 (for remote procedure call (RPC) portmapper, only required for NFSv4.0) from {{site.data.keyword.redhat_openshift_notm}} nodes to the server containing the OVA files.

### Source VM prerequisites
{: #source-vm-prerequisites}

Source VMs across all migrations must meet the following prerequisites:

Media status
:   Unmount ISO images and CD-ROMs before migration.

IP addressing
:   Each network interface card (NIC) must contain either an IPv4 address or an IPv6 address, and can use both.

Operating system (OS) certification
:   The VM operating system must be certified and supported as a guest operating system for conversion.

Boot features
:   Virtual machines with Secure Boot enabled might not be migrated automatically, as this prevents them from booting on the destination provider. To resolve this, disable Secure Boot on the destination.

VMware guest agents
:   VMware Tools or open-vm-tools are required when **Preserve static IPs** is enabled.

### VM naming
{: #vm-naming}

VM names must comply with these guidelines:

- The name of a VM must not contain a period (`.`). MTV automatically changes any period in a VM name to a dash (`-`).
- The VM name must be unique within the {{site.data.keyword.redhat_openshift_notm}} Virtualization environment (that is, it must not match any other VM name).
- If a VM name fails to comply with the rules, MTV automatically generates a new name. MTV removes excluded characters, converts uppercase letters to lowercase, and changes any underscore (`_`) to a dash (`-`).

### Encryption support
{: #encryption-support}

MTV supports migrating VMs by using the following encryption types:

- Linux&reg; VMs: Linux Unified Key Setup (LUKS).
- Windows&reg; VMs: BitLocker.

## VMware vSphere specific prerequisites
{: #vmware-prerequisites}
{: step}

### VMware Tools
{: #vmware-tools}

To use a pre-migration hook to access the virtual machine, install VMware Tools or `open-vm-tools` on the source virtual machine.

### VDDK image
{: #vddk-image}

Use the VMware Virtual Disk Development Kit (VDDK) when you transfer virtual disks from VMware vSphere to accelerate migrations.

- A VDDK image is optional. However, omitting it can significantly decrease migration speeds, and it is required when the VM is backed by VMware vSAN.

### VMware privileges
{: #vmware-privileges}

You must be logged in with at least the minimum set of required VMware privileges. For more information, see [VMware privileges](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/prerequisites-per-provider_mtv#vmware-privileges_mtv){: external}.

### Warm migration requirement
{: #warm-migration-requirement}

To run a warm migration, you must enable changed block tracking (CBT) on the virtual machine (VM) and on each individual VM disk. The incremental copying during the precopy stage relies on CBT snapshots. A single VM supports up to 28 CBT snapshots.

Disable hibernation for all VMs because MTV does not support migrating hibernated VMs, and migration fails if hibernation is not disabled.

### ESXi host configuration
{: #esxi-host-configuration}

If a migration plan requires migrating more than 10 VMs concurrently from a single ESXi (VMware vSphere hypervisor) host, increase the Network File Copy (NFC) service memory of that host.

- The NFC service memory defaults to supporting only 10 parallel connections, which is insufficient for more than 10 concurrent VM migrations.
- Change the `maxMemory` value to `1000000000` (1 GB) and restart the `hostd` service.

### Windows warm migration
{: #windows-warm-migration}

For warm migrations of Microsoft Windows&reg; VMs from VMware, the Volume Shadow Copy Service (VSS) inside the guest VM must be running.
For more information, see [Source virtual machine prerequisites](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/prerequisites-for-all-providers_mtv#source-vm-prerequisites_mtv){: external}.

## Installation and configuration of the MTV Operator
{: #installation-configuration}
{: step}

You can install the MTV Operator, which includes the MTV plug-in for the {{site.data.keyword.redhat_openshift_notm}} web console, by using either the {{site.data.keyword.redhat_openshift_notm}} web console or the command-line interface (CLI).

Before you install the MTV Operator, ensure that the following prerequisites are met:

- Install {{site.data.keyword.redhat_openshift_notm}} 4.19, 4.18, or 4.17.
- Install the {{site.data.keyword.redhat_openshift_notm}} Virtualization Operator on the {{site.data.keyword.redhat_openshift_notm}} migration target cluster.
- Log in with `cluster-admin` permissions.

For more information, see [Installing the MTV Operator by using the {{site.data.keyword.redhat_openshift_notm}} web console](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/installing-the-operator_mtv#installing-mtv-operator_web){: external}.

Configure the MTV Operator settings by modifying the `ForkliftController` custom resource (CR) or by using the **Settings** section of the **Overview** page in the web console, unless specified otherwise.

Use the **Settings** tab in the MTV **Overview** page to adjust the following key parameters:

- Maximum concurrent VM migrations (default is 20)
- Controller main container CPU limit
- Controller main container memory limit
- Controller inventory container memory limit
- Precopy interval (minutes)
- Snapshot polling interval

For more information, see [Configuring the MTV Operator](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/installing-the-operator_mtv#configuring-mtv-operator_mtv){: external}.

## Migrating virtual machines from VMware vSphere (UI workflow)
{: #migrating-vms-ui}
{: step}

### Add VMware vSphere source provider
{: #add-vmware-source}

1. Go to **Migration for Virtualization** > **Providers** and click **Create provider**.
2. Select **VMware** and specify the provider resource name and endpoint type (vCenter or ESXi).
3. Provide the URL of the SDK endpoint, for example, `https://vCenter-host-example.com/sdk`.
4. Create a VDDK image and specify its path. Using a VDDK image accelerates migrations and is required if VMs are backed by VMware vSAN.
5. Enter vCenter or ESXi credentials.
6. Choose a CA certificate validation option: **Use a custom CA certificate**, **Use the system CA certificate**, or **Skip certificate validation**.
7. Click **Create provider**.

### Creating a migration plan
{: #creating-migration-plan}

1. General: Define plan name, project, source provider, and target provider/project.

   - Start the wizard from **Migration for Virtualization** > **Migration plans** by clicking **Create plan**.
   - Specify the **Plan name**, **Plan project**, **Source provider**, **Target provider**, and **Target project**.

2. Virtual machines

   - Select the virtual machines to migrate.

    A single plan cannot contain more than 500 VMs or 500 disks.
    {: important}

3. Define the network map

   - Choose to use an existing, ownerless network map (which creates a copy that is attached to the plan), or use a new network map (owned by the plan).
   - If you are creating a new map, define the mappings between the source network and the target network.

4. Select the migration type

   - Select **Cold migration** (default) (VM is shut down during data copy) or **Warm migration** (VM runs during the precopy stage, minimizing downtime).

5. Other settings (optional)

   - Disk decryption passphrases: Enter passphrases for LUKS-encrypted devices.
   - Transfer network: Optionally override the provider's default transfer network. If the {{site.data.keyword.redhat_openshift_notm}} transfer network maximum transmission unit (MTU) is changed, the VMware migration network MTU must also be adjusted.
   - Preserve static IPs: Select the **Preserve static IPs** checkbox to attempt to preserve static IP addresses, mitigating loss due to vNIC changes during migration.
   - Root device: For multi-boot VMs, manually specify the disk location for the root device, for example, `/dev/sdb2`.
   - Shared disks: Shared disks are enabled by default for cold migrations. Shared disks use the multi-writer option and can slow down the migration process.
   - Select the **Enable hook** checkbox for pre-migration (operations on source VM before migration) or post-migration (operations on migrated VM after migration).
   - You must specify the hook runner image (default is `quay.io/kubev2v/hook-runner`) and provide the Ansible&reg; playbook. Only one pre-migration and one post-migration hook are allowed per plan.
   - When you use **Preserve static IPs** and move to a Layer 2 primary user-defined network (UDN) or cluster user-defined network (CUDN), the source VM must be powered on and running a VMware Guest Agent. **Preserve static IP** does not work with secondary networks.

6. Review and create

   - Review all plan details. To edit any detail, use the **Edit step** link.
   - Click **Create plan**. MTV validates the plan. If validation is successful, the **Plan details** page opens.

### Post-creation configuration
{: #post-creation-configuration}
{: step}

1. Review the **Plan details** page, especially for settings not in the wizard.

   - The **Plan details** page contains important settings not visible in the wizard.
   - Review the **Plan settings** section for optional configurations. To edit any setting, use the **Options** menu.
   - Check the **Conditions** section; all listed conditions must be resolved before the plan can run successfully.

### Running and monitoring
{: #running-monitoring}
{: step}

1. Pre-migration activities

   - Before you start a warm migration, check for VM snapshots and commit/delete snapshots to ensure the VMs have enough available CBT snapshots.
   - Grant access for the `default` service account of the target namespace to pull the VDDK image from the {{site.data.keyword.redhat_openshift_notm}} MTV namespace.
   - Validate the migration plan and resolve any warnings or errors.
   - Take a full backup of the source VMs if needed.

2. Start migration

   - Go to **Migration** > **Plans for virtualization**.
   - Click **Start** next to the migration plan and confirm.
   - Disable vMotion, svMotion, and relocation for the VMs that you are importing to prevent data corruption.
   {: important}

3. Warm migration cutover

   - For warm migrations, the initial data transfer (precopy stage) begins immediately.
   - To initiate the cutover stage, which shuts down the source VM and transfers the final delta, click **Cutover**.
   - The cutover window allows you to **Set cutover** (schedule time and date) or **Remove cutover** (cancel a scheduled time).
   - Before cutover, stop applications, middleware, and database services on source VMs to minimize data changes. MTV does not require this action. However, stopping these services reduces the risk of data inconsistency during cutover.

4. Monitoring

   - The migration **Status** link displays overall progress, success/failure counts, and ongoing status.
   - The **Virtual Machines** tab provides specific VM status, start/end times, data copied, and a progress pipeline.
   - To access logs for running or completed migrations, select a VM on the **Virtual Machines** tab and click the **Logs** link in the **Pods** section.

5. Post-migration restriction

   - Do not take a snapshot of a VM after the migration starts, because this can cause the migration to fail.

6. Post-migration activities

   - Configure network settings and IP changes if required.
   - Configure role-based access control (RBAC) and service accounts for VM management.
   - Update Domain Name System (DNS) records if required.
   - Validate DNS resolution and network policies.
   - Start applications and databases on target VMs if they were stopped before cutover.
   - Verify application functionality on the migrated VMs.

## Performance and troubleshooting
{: #performance-troubleshooting}
{: step}

1. Networking, storage, and host tuning for migration throughput

   - Ensure fast networking and storage: Both VMware and {{site.data.keyword.redhat_openshift_notm}} Container Platform (OCP) environments require fast storage and network speeds.
   - High throughput: For best results, ensure that VMware network connectivity offers high throughput (at least 10-Gigabit Ethernet or 10 GiB network connection) so that reception rates align with ESXi data store read rates.
   - Observed speeds: Average network transfer rates ranged from 200 to 325 MiB/s from the `vmnic` for each ESXi host.
   - Host settings: Set ESXi hosts BIOS profiles and host power management settings to **High Performance** where possible. Testing showed an increase of 15 MiB in the average data store read rate when transferring more than 10 VMs.

2. Concurrency

   - Maximum concurrent transfers (`MAX_VM_INFLIGHT`): Use the `MAX_VM_INFLIGHT` MTV variable to control the maximum number of simultaneous VM transfers permitted per ESXi host; the default value is 20.
   - For VMware cold migrations to a local {{site.data.keyword.redhat_openshift_notm}} environment, this controls the number of VMs per ESXi host that migrates concurrently.
   - For VMware warm migrations, this controls the number of disks per ESXi host that migrate simultaneously.
   - Benefit of parallelism: Starting concurrent migrations for multiple VMs from a single ESXi host significantly reduces total migration time compared to migrating them sequentially. Migrating 10 VMs concurrently is three times faster than migrating them sequentially.
   - Multiple hosts: Using multiple ESXi hosts with VMs distributed evenly among them results in faster migration times. For example, testing showed that migrating 80 VMs using 8 ESXi hosts concurrently is four times faster than using a single host.

3. Large migrations

   - Plan limit: A single migration plan cannot exceed 500 VMs or 500 disks.
   - Multiple plans: When you migrate many VMs, breaking a single large plan into multiple moderately sized plans (for example, 100 VMs per plan) can reduce the total migration time by allowing concurrent starts.

4. Warm migration tuning

   - Disk concurrency limit: Testing that is involved up to 400 parallel disk transfers (200 VMs with two disks each). Limit parallel disk migrations to 200 or fewer disks for the fastest migration rate, as speeds decrease by about 25% beyond this threshold.
   - Immediate cutover: To minimize overall warm migration time and ensure only one precopy runs per VM, set the cutover to occur immediately after the migration plan starts.
   - Adjusting precopy interval: If you have adequate time between the migration start and cutover, increase the `controller_precopy_interval` parameter (default 60 minutes) to between 120 and 240 minutes to reduce the total number of snapshots and disk transfers before cutover.

5. Large disks (1 TB+)

   - Prioritization: For migrations that involve large disks, prioritize MTV activities and ensure no other heavy network or storage activities run concurrently.
   - High churn VMs: For large VMs with a high churn rate (100 GB+ data change between snapshots), consider reducing the default 60-minute warm migration `controller_precopy_interval` at least 24 hours before the scheduled cutover.
   - Cold versus warm: If some downtime is possible, choose cold migration over warm migration for particularly large single-disk VMs, especially when VM snapshots are large.
   - Databases: For large database disks with continuous writes where snapshots are impossible, consider using database vendor-specific replication options outside of MTV.

6. Asynchronous input/output (AIO) buffering (cold migration only)

   - Function: AIO buffering changes Network Block Device (NBD) transport NFC parameters to potentially increase cold migration performance.
   - Cold migration requirement: AIO buffering is suitable only for cold migration use cases and you must disable it before you initiate warm migrations.

## Troubleshooting
{: #troubleshooting}

If you encounter errors during migration, see [Troubleshooting MTV migrations from VMware vSphere to Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-mtv-migration) for resolutions to common issues, including snapshot limits, VDDK pull failures, DNS resolution failures, and static IP preservation problems.

## Additional resources
{: #additional-resources}

- [MTV documentation (v2.9)](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/index){: external}

## Next steps
{: #migration-toolkit-next-steps}

After you complete your migration with MTV, you can take the following steps:

- Implement backup: Set up [backup and recovery with Veeam Kasten](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-backup) to protect your migrated workloads
- Review architecture: Explore the complete [{{site.data.keyword.redhat_openshift_notm}} Virtualization reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-rove-architecture)
- Design considerations: Learn about [networking](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-network-design), [storage](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-storage-design-overview), and [security](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-security-design-overview) best practices
- Optimize performance: Review [observability and monitoring](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-openshift-observability-design-overview) for your environment
- [Preparing virtual machine networking for migration from vSphere to {{site.data.keyword.redhat_openshift_notm}} Virtualization](https://www.redhat.com/en/blog/openshift-virtualization-networking-for-vsphere-migration){: external}
