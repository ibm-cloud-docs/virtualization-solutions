---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-09"

keywords: migration, warm migration, cold migration, mtv, red hat openShift migration toolkit for virtualization, migration toolkit for virtualization

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Red Hat OpenShift Migration Toolkit for Virtualization (MTV)
{: #virt-sol-openshift-migration-design-mtv}

**The Migration Toolkit for Virtualization (MTV)** on Red Hat OpenShift Container Platform helps you migrate virtual servers from traditional hypervisors such as VMware vSphere, Red Hat Virtualization, or OpenStack into Red Hat OpenShift Virtualization, by consolidating virtual servers and containers into a single Kubernetes-native platform.
{: shortdesc}

MTV provides a web UI and API for discovery, planning, and execution of migrations that uses persistent volume cloning or streaming for disk data while it maintains virtual server configuration and networking. MTV integrates with Red Hat OpenShift Virtualization to run migrated virtual servers alongside containers by using Kubernetes-native storage and networking constructs.

For VMware migrations, the Migration Toolkit for Virtualization (MTV) integrates with vCenter to discover virtual machines and inventory data, map virtual machine resources (clusters, networks, data stores) to Red Hat OpenShift equivalents, and automate bulk or selective migrations. It supports disk data transfer with warm or cold migration, preserves virtual machine configuration (CPU, memory, NICs), and converts VMware constructs into Kubernetes-native resources for seamless execution in Red Hat OpenShift Virtualization.

## Migration Types Supported
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

## Red Hat OpenShift migrations
{: #virt-sol-openshift-migration-design-migration-type}

Migration Toolkit for Virtualization (MTV) supports two types of migration.

   - **Cold migration** is the default migration. The source virtual servers are stopped while the data is copied.
   - **Warm migration** most of the data during the precopy stage is copied. Then, the virtual servers are stopped and the remaining data is copied during the cutover stage.

When you compare the migration speeds of cold and warm migrations, you can observe that the single disk transfer and disk conversion are approximately the same for each option. The benefit of warm migration is that the transfer of the snapshot happens in the background while the virtual server runs. The default snapshot time is every 60 minutes. If virtual servers change substantially, more data needs to be transferred than in cold migration when the virtual server is stopped. The cutover time, meaning the shutdown of the virtual machine and last snapshot transfer, depends on how much the virtual server was changed after the last snapshot.

## Red Hat OpenShift cold migration
{: #virt-sol-openshift-cold-migration}

Cold migration is the default migration type. The source virtual machines are shut down while the data is copied.

To enable MTV to automatically install `qemu-guest-agent` on the migrated virtual servers, make sure that your package manager can install the daemon during the first start of the virtual server after the migration. If that isn't possible, use your preferred automated or manual procedure to install `qemu-guest-agent`.
{: note}

## Red Hat OpenShift warm migration
{: #virt-sol-openshift-warm-migration}

During a warm migration, the virtual server isn't stopped during the precopy stage. A warm migration is a two-step process. Most of the data is copied during the precopy stage while the source virtual servers run. Then, the virtual servers are stopped and the remaining data is copied during the cutover stage.

### Warm migration precopy stage
{: #virt-sol-openshift-warm-migration-precopy}

The following information is an overview of the precopy stage process.

- An initial snapshot of running virtual server disks is created.
- The initial snapshot to target (full disk transfer, largest amount of data copied - takes more time) is copied.
- The virtual server disks are incrementally copied by using the changed block tracking (CBT) snapshots.
- Copy deltas - changed data (copies only the data that changed after the last snapshot - takes less time).
   - A new snapshot is created.
   - The delta between the previous snapshot and the new snapshot is copied.
   - The next snapshot (configurable, by default 1 hour after the last snapshot finished) is scheduled.

Virtual servers can support up to 28 CBT snapshots. If you exceed that limit, a warm import retry limit-reached error message appears. If the virtual server has preexisting CBT snapshots, it reaches this limit sooner.
{: note}

### Warm migration cutover stage
{: #virt-sol-openshift-warm-migration-cutover}

The virtual servers are shut down during the cutover stage and the remaining data is migrated. Data that is stored in RAM isn't migrated.

The following information is an overview of the cutover stage process.

- Time to finalize warm migration is scheduled.
- You can start the cutover stage manually in the MTV console.
- The migration process continues in the same way as cold migration
   - Guest conversion.
   - Optionally, start the target virtual server.
