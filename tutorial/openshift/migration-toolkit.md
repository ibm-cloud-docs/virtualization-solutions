---

copyright:
   years: 2025
lastupdated: "2025-12-15"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption, MTV, Migration

subcollection: virtualization-solutions

content-type: tutorial
services: Registry, OpenShift, OpenShift Virtualization, VMware 
account-plan: paid 
completion-time: 60m 

---


{{site.data.keyword.attribute-definition-list}}

# Migrate from VMware to Red Hat OpenShift Virtualization Tutorial
{: #vsphere-openshift-migration}
{: #tutorial-migration-toolkit}
{: toc-content-type="tutorial"}
{: toc-services="Registry, OpenShift, OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}

This document provides a comprehensive guide to migrating virtual machines from VMware vSphere to OpenShift Virtualization using the Migration Toolkit for Virtualization (MTV), including prerequisites, installation, and step-by-step migration processes.
{: shortdesc}

## Migration Toolkit for Virtualization
{: #introduction-overview}

The Migration Toolkit for Virtualization (MTV) is designed to migrate virtual machines (VMs) to OpenShift Virtualization running on Red Hat OpenShift.
MTV supports migration from the following VMware vSphere source providers:
- VMware vSphere
- Open Virtual Appliances (OVAs) created by VMware vSphere

### Migration Types Supported
{: #migration-types-supported}

Migration Toolkit for Virtualization supports the following types of VM migration.

1.  **Cold Migration:**
- Migrates a powered off VM to a separate host.

2.  **Warm Migration:**
- Migrates a powered on VM to a separate host.
- The source host state is cloned to the destination host.
- Most data is copied during the precopy stage while the source VMs are running.
- The precopy stage uses incremental copying of VM disks via Changed Block Tracking (CBT) snapshots (snapshots are created at one-hour intervals by default, which can be changed).
- The cutover stage requires the VMs to be shut down so the remaining delta data can be copied. Data stored in RAM is not migrated.

3.  **OVA Migration:**

The table below compares the characteristics, advantages, and disadvantages of cold migration, warm migration, and OVA migration.

| Feature | Cold Migration | Warm Migration | OVA Migration (Source Type) |
| :--- | :--- | :--- | :--- |
| **VM Power Status (During Data Transfer)** | Shut down. | Running (during precopy phase). | Shut down. |
| **VM Downtime** | **High**. Correlates directly to the amount of data transferred. | **Low**. VMs are shut down only during the final cutover stage. | **High** (Similar to cold migration). |
| **Data Copy Strategy** | Each block is copied once. | Blocks may be copied multiple times due to VM utilization. Uses incremental CBT snapshots. | Each block is copied once. |
| **Conversion Timing** | **Fail Fast:** Conversion to OpenShift compatibility happens *before* disk transfer. If conversion fails, migration stops immediately. | Transfer (snapshots) happens *before* conversion. Conversion runs after the final snapshot transfer during cutover. | Conversion typically happens *before* disk transfer (as it uses cold migration flow). |
| **Best Use Case** | Shortest duration for VMs with a large amount of data on a single disk. | Shortest downtime for VMs. Shortest duration for VMs with data spread across multiple disks. | Migrating offline VM images stored as OVAs on an NFS share. |
{: caption="Migration Option Comparison" caption-side="bottom"}


## Migration Prerequisites and Environment Requirements
{: #prerequisites-requirements}

### Software Compatibility
{: #software-compatibility}

Successful migration requires installing compatible versions of Red Hat OpenShift and OpenShift Virtualization.

For Migration Toolkit for Virtualization (MTV) 2.9, the compatible software versions include:
- **Red Hat OpenShift:** 4.19, 4.18, 4.17
- **OpenShift Virtualization:** 4.19, 4.18, 4.17
- **VMware vSphere:** 6.5 or later

### Network Requirements
{: #network-requirements}

The following network prerequisites apply across all migrations:
- **Network Stability:** The network connections between the source environment and the OpenShift Virtualization cluster must be reliable and uninterrupted.
- **Configuration Integrity:** IP addresses, VLANs, and other network configuration settings must not be changed during migration, as the VM's MAC addresses are preserved.
- **Destination Networks:** If multiple source and destination networks are mapped, a network attachment definition must be created for each additional destination network.
- **Required Ports:** Firewalls must permit traffic over specific ports based on the source provider:
- **VMware vSphere:** TCP ports 443 (for inventory and disk transfer authentication) and 902 (for disk transfer data copy) from OpenShift nodes to VMware vCenter/ESXi hosts.
- **Open Virtual Appliance (OVA):** TCP port 2049 (for NFS service) and TCP or UCP port 111 (for RPC Portmapper, only required for NFSv4.0) from OpenShift nodes to the server containing the OVA files.

### Source VM Prerequisites
{: #source-vm-prerequisites}

Prerequisites for source VMs across all migrations include:
- **Media Status:** ISO images and CD-ROMs must be unmounted.
- **IP Addressing:** Each NIC must contain either an IPv4 address or an IPv6 address, and may utilize both.
- **OS Certification:** The VM operating system must be certified and supported for conversion as a guest operating system.
- **Boot Features:** Virtual machines with Secure Boot enabled currently might not be migrated automatically, as this prevents them from booting on the destination provider. The current workaround for this issue is to disable Secure Boot on the destination.

### VM Naming
{: #vm-naming}

VM names must comply with these guidelines:
- The name of a VM must not contain a period (`.`). MTV automatically changes any period in a VM name to a dash (`-`).
- The VM name must be unique within the OpenShift Virtualization environment (i.e., it must not match any other VM name).
- If a VM name fails to comply with the rules, MTV will automatically generate a new name by removing excluded characters, switching uppercase letters to lowercase, and changing any underscore (`_`) to a dash (`-`).

### Encryption Support
{: #encryption-support}

MTV supports the migration of VMs using the following encryption types:
- **Linux VMs:** Linux Unified Key Setup (LUKS).
- **Windows VMs:** BitLocker.

## VMware vSphere Specific Prerequisites
{: #vmware-prerequisites}
{: step}

### VMware Tools
{: #vmware-tools}

To utilize a pre-migration hook to access the virtual machine, VMware Tools must be installed on the source virtual machine.

### VDDK Image (Strongly Recommended)
{: #vddk-image}

It is strongly recommended to use the VMware Virtual Disk Development Kit (VDDK) SDK when transferring virtual disks from VMware vSphere, as it accelerates migrations.
- The creation of a VDDK image is optional but highly recommended; using MTV without VDDK is generally not recommended and can significantly decrease migration speeds.
- VM migrations will not work without VDDK when the VM is backed by VMware vSAN.

### VMware Privileges
{: #vmware-privileges}

The user performing the migration must be logged in with at least the minimal set of required VMware privileges. For more information, see [VMware privileges](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/prerequisites-per-provider_mtv#vmware-privileges_mtv).

### Warm Migration Requirement
{: #warm-migration-requirement}

To run a warm migration, you must enable changed block tracking (CBT) on the VM and on each individual VM disk. The incremental copying during the precopy stage relies on CBT snapshots. A single VM supports up to 28 CBT snapshots.

It is also strongly recommended to disable hibernation for all VMs because MTV does not support migrating hibernated VMs, and migration will fail if hibernation is not disabled.

### ESXi Host Configuration
{: #esxi-host-configuration}

If a migration plan involves migrating more than 10 VMs concurrently from a single ESXi host, you must increase the Network File Copy (NFC) service memory of that host.
- This is necessary because the NFC service memory defaults to supporting only 10 parallel connections.
- The required modification involves changing the `maxMemory` value to `1000000000` (1 GB) and restarting the `hostd` service.

### Windows Warm Migration
{: #windows-warm-migration}

For warm migrations of Microsoft Windows virtual machines from VMware, the Volume Shadow Copy Service (VSS) inside the guest VM must be running.
For more information, see [Source virtual machine prerequisites](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/prerequisites-for-all-providers_mtv#source-vm-prerequisites_mtv).


## Installation and Configuration of the MTV Operator
{: #installation-configuration}
{: step}

The MTV Operator, which includes the MTV plugin for the Red Hat OpenShift web console, can be installed using either the Red Hat OpenShift web console or the command-line interface (CLI).

The following prerequisites must be met:
- Red Hat OpenShift 4.19, 4.18, or 4.17 must be installed.
- The OpenShift Virtualization Operator must be installed on the OpenShift migration target cluster.
- The user must be logged in with cluster-admin permissions.

For more information, see [Installing the MTV Operator by using the Red Hat OpenShift web console](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/installing-the-operator_mtv#installing-mtv-operator_web).


The MTV Operator settings are configured by modifying the `ForkliftController` custom resource (CR) or by using the **Settings section of the Overview page** in the web console, unless specified otherwise.

The Settings tab in the MTV Overview page allows adjustment of the following key parameters:
- **Maximum concurrent VM migrations**, default is 20.
- **Controller main container CPU limit**
- **Controller main container memory limit**
- **Controller inventory container memory limit**
- **Precopy internal (minutes)**
- **Snapshot polling interval**

For more information, see [Configuring the MTV Operator](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/installing-the-operator_mtv#configuring-mtv-operator_mtv).


## Migrating Virtual Machines from VMware vSphere (UI Workflow)
{: #migrating-vms-ui}
{: step}

### Add VMware vSphere Source Provider
{: #add-vmware-source}

- Access the **Create provider** interface via **Migration for Virtualization > Providers**.
- Selecting VMware requires specifying the Provider resource name and Endpoint type (vCenter or ESXi).
- Provide the URL of the SDK endpoint (e.g., `https://vCenter-host-example.com/sdk`).
- **Highly Recommended:** Create a **VMware Virtual Disk Development Kit (VDDK) image** and specify its path, as it accelerates migrations and is necessary if VMs are backed by VMware vSAN.
- Enter vCenter or ESXi credentials.
- Choose CA certificate validation options: **Use a custom CA certificate**, **Use the system CA certificate**, or **Skip certificate validation**.
- Click **Create provider**.

### Creating a Migration Plan
{: #creating-migration-plan}

1. General: Define plan name, project, source provider, and target provider/project.

- Start the wizard from **Migration for Virtualization > Migration plans** by clicking **Create plan**.
- Specify the **Plan name**, **Plan project**, **Source provider**, **Target provider**, and **Target project**.

2. Virtual Machines

- Select the virtual machines to be migrated. **Important:** A single plan cannot contain more than 500 VMs or 500 disks.

3. Define the Network Map

- Choose to use an existing, **ownerless network map** (which creates a copy attached to the plan), or **use a new network map** (owned by the plan).
- If creating a new map, define the mappings between the **Source network** and **Target network**.

4. Define the Storage Map

- Choose to use an existing, **ownerless storage map** (which creates a copy), or **use new storage map** (owned by the plan).
- If creating a new map, define the mappings between the **Source storage** and **Target storage**.

5. Select the Migration Type

- Select **Cold migration (default)** (VM is shut down during data copy) or **Warm migration** (VM runs during precopy stage, minimizing downtime).

6. Other Settings (Optional)

- **Disk decryption passphrases:** Enter passphrases for Linux Unified Key Setup (LUKS) encrypted devices.
- **Transfer Network:** Optionally override the provider's default transfer network. Note: If the OpenShift transfer network MTU is changed, the VMware migration network MTU must also be adjusted.
- **Preserve static IPs:** Select this checkbox to attempt to preserve static IP addresses, mitigating loss due to vNIC changes during migration.
- **Root device:** For multi-boot VMs, manually specify the disk location for the root device (e.g., `/dev/sdb2`).
- **Shared disks:** This is enabled by default for cold migrations. Shared disks use the multi-writer option and can slow down the migration process.
- Select the **Enable hook** checkbox for **pre-migration** (operations on source VM before migration) or **post-migration** (operations on migrated VM after migration).
- You must specify the **Hook runner image** (default is `quay.io/kubev2v/hook-runner`) and provide the Ansible playbook. Only one pre-migration and one post-migration hook are allowed per plan.

7. Review and Create: Create the plan, which triggers validation.

- Review all plan details. Edits can be made using the **Edit step link**.
- Click **Create plan**; MTV validates the plan. If validation is successful, the Plan details page opens.

### Post-Creation Configuration
{: #post-creation-configuration}
{: step}

1. Review the Plan Details page, especially for settings not in the wizard.

- The **Plan details page** contains important settings not visible in the wizard.
- Review the **Plan settings** section for optional configurations. All settings can be edited via the Options menu.
- Check the **Conditions** section; all listed conditions must be resolved before the plan can run successfully.

Running and Monitoring
{: #running-monitoring}
{: step}

1. Pre-Migration Activities 
- Before starting a warm migration, check for VM snapshots and commit/delete snapshots to ensure the VMs have enough available CBT snapshots.
- Grant access for 'default' service account of Target Namespace to pull VDDK image from Openshift MTV Namespace.
- Validate migration plan and resolve any warnings/errors.
- Take a full backup of the source VMs if needed.

2. Start Migration

- Navigate to **Migration > Plans for virtualization**.
- Click **Start** next to the migration plan and confirm.
- **Restriction:** vMotion, svMotion, and relocation must be disabled for the VMs being imported to prevent data corruption.

3. Warm Migration Cutover

- For warm migrations, the initial data transfer (**precopy stage**) begins immediately.
- The cutover stage, which involves shutting down the source VM and transferring the final delta, must be initiated manually by clicking **Cutover**.
- The Cutover window allows you to **Set cutover** (schedule time/date) or **Remove cutover** (cancel a scheduled time).
- Before cutover, stop applications, middleware, and database services on source VMs. This step is not required by MTV but is recommended to minimize data changes during cutover.


4. Monitoring

- The migration's **Status** link displays overall progress, success/failure counts, and ongoing status.
- The **Virtual Machines tab** provides specific VM status, start/end times, data copied, and a progress pipeline.
- Logs for running or completed migrations can be accessed by selecting a VM on the Virtual Machines tab and clicking the **Logs link** in the Pods section.

5. Important Restriction

- Do not take a snapshot of a VM after the migration has started, as this may cause the migration to fail.

6. Post-Migration Activities

- Configure network settings and IP changes if required.
- Configure RBAC and service accounts for VM management.
- Update DNS records if required.
- Validate DNS resolution and network policies.
- Start applications and databases on target VMs if they were stopped before cutover.
- Verify application functionality on the migrated VMs.

## V. Performance, and Troubleshooting
{: #performance-troubleshooting}
{: step}

 1. Networking, Storage, and Host Tuning for Migration Throughput

- **Ensure Fast Networking and Storage:** Both VMware and Red Hat OpenShift (OCP) environments require fast storage and network speeds.
- **High Throughput:** VMware network connectivity should offer high throughput (at least 10 Gigabit Ethernet or 10 GiB network connection) to ensure reception rates align with ESXi datastore read rates.
- **Observed Speeds:** Average network transfer rates observed were 200 to 325 MiB/s from the vmnic for each ESXi host.
- **Host Settings:** Set ESXi Hosts BIOS profiles and Host Power Management settings to High Performance where possible. This showed an increase of 15 MiB in the average datastore read rate when transferring more than 10 VMs.

2. Concurrency

- **Maximum Concurrent Transfers (`MAX_VM_INFLIGHT`):** Use the `MAX_VM_INFLIGHT` MTV variable to control the maximum number of simultaneous VM transfers permitted per ESXi host; the default value is 20.
- For VMware cold migrations to a local OpenShift environment, this controls the number of VMs per ESXi host that migrate concurrently.
- For VMware warm migrations, this controls the number of disks per ESXi host that migrate simultaneously.
- **Benefit of Parallelism:** Starting concurrent migrations for multiple VMs from a single ESXi host significantly reduces total migration time compared to migrating them sequentially. Migrating 10 VMs concurrently was three times faster than migrating them sequentially.
- **Multiple Hosts:** Using multiple ESXi hosts with VMs distributed evenly among them results in faster migration times. For example, testing showed migrating 80 VMs using 8 ESXi hosts concurrently was four times faster than using a single host.

3. Large Migrations

- **Plan Limit:** A single migration plan cannot exceed 500 VMs or 500 disks.
- **Multiple Plans:** When migrating many VMs, breaking a single large plan into multiple moderately sized plans (e.g., 100 VMs per plan) can reduce the total migration time by allowing concurrent starts.

4. Warm Migration Tuning

- **Disk Concurrency Limit:** While testing involved up to 400 parallel disk transfers (200 VMs with 2 disks each), it is recommended to limit parallel disk migrations to 200 or fewer disks for the fastest migration rate, as speeds decreased by about 25% beyond this threshold.
- **Immediate Cutover:** To minimize overall warm migration time and ensure only one precopy runs per VM, set the cutover to occur immediately after the migration plan starts.
- **Adjusting Precopy Interval:** If you have adequate time between the migration start and cutover, increase the `controller_precopy_interval` parameter (default 60 minutes) to between 120 and 240 minutes to reduce the total number of snapshots and disk transfers before cutover.

5. Large Disks (1 TB+)

- **Prioritization:** Treat migrations involving large disks as a special case; prioritize MTV activities and ensure no other heavy network or storage activities run concurrently.
- **High Churn VMs:** For large VMs with a high churn rate (100 GB+ data change between snapshots), consider reducing the default 60-minute warm migration `controller_precopy_interval`. This process must start at least 24 hours before the scheduled cutover.
- **Cold vs. Warm:** If some downtime is possible, choose cold migration over warm migration for particularly large single-disk VMs, especially due to potentially large VM snapshots.
- **Databases:** For large database disks with continuous writes where snapshots are impossible, consider using database vendor-specific replication options outside of MTV.

6. AIO Buffering (Cold Migration Only)

- **Function:** Asynchronous Input/Output (AIO) buffering changes Network Block Device (NBD) transport network file copy (NFC) parameters to potentially increase cold migration performance.
- **Cold Migration Requirement:** AIO buffering is suitable only for cold migration use cases and must be disabled before initiating warm migrations.

### B. Troubleshooting
{: #troubleshooting}
{: step}

 1. Logs and custom resources

You can download logs and custom resource (CR) information for troubleshooting. For more information, see [Collected logs and custom resource information](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/logs-and-crs_mtv#collected-logs-cr-info_mtv).

2. Common Errors

    a. **Warm import retry limit reached**

    - Cause: Warm migration created more than the maximum 28 CBT snapshots.
    - Resolution:
        - Delete older CBT snapshots on the source VM.
        - Ensure snapshot churn is reduced (increase precopy interval if appropriate).
        - Restart the migration plan.

    b. **Unable to resize disk image to required size**

    - Cause: Destination VM persistent volumes (EXT4 on block storage) exceed the default 10% filesystem overhead assumed by CDI, leaving insufficient space for the root partition.
    - Resolution:
        - Edit the ForkliftController CR.
        - Increase controller_filesystem_overhead (set to a value > 0.10, for example 0.15).
        - Apply the change and re-run the migration.

    c. **Migration plan fails after creation**

    - Cause: VDDK image pull denied; validator pod cannot authenticate to internal registry.
    - Resolution:
        - Grant pull access: `oc adm policy add-cluster-role-to-user registry-viewer system:serviceaccount:<target-namespace>:default`
        - Run via Web Terminal or local oc CLI.

    d. **Migration plan fails during initialize phase**

    - Cause: Importer pod cannot resolve ESXi hostnames; DNS lookup fails for port 902 host connections.
    - Resolution:
        - Configure DNS forwarding/zone for the vCenter/ESXi domain.
        - Add forwarding zone pointing to domain controllers.
        - Example:
            ```yaml
            servers:
            - forwardPlugin:
                    policy: Random
                    upstreams:
                    - <Domain Controller IP1>
                    - <Domain Controller IP2>
                name: vcs-resolver
                zones:
                - vcs.example.com
            ```
        - Apply DNS change.

    e. **virt-v2v: filesystem mounted read-only**

    - Cause: Source Windows VM was not cleanly shut down (Fast Startup or hibernation left filesystem dirty) before exporting the OVA.
    - Resolution:
        - Disable Fast Startup: Control Panel > Power Options > Choose what the power buttons do > Uncheck "Turn on fast startup".
        - Disable hibernation (elevated cmd): powercfg /h off
        - Perform a clean shutdown: shutdown /s /t 0
        - Re-export the OVA after the clean shutdown.
        - Upload the new OVA to the NFS server and retry OVA migration.

## VI. Additional Resources
{: #additional-resources}

The following resources provide detailed guidance and practical examples:
- MTV documentation (v2.9): https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/index
- Preparing virtual machine networking for migration from vSphere to Red Hat OpenShift Virtualization: https://www.redhat.com/en/blog/openshift-virtualization-networking-for-vsphere-migration
