---

copyright:
  years: 2025
lastupdated: "2026-01-13"

keywords:

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Red Hat OpenShift Migration Toolkit for Virtualization (MTV)
{: #virt-sol-openshift-migration-design-mtv}

The Migration Toolkit for Virtualization (MTV) on OpenShift Container Platform helps migrate virtual machines from traditional hypervisors like VMware vSphere, Red Hat Virtualization, or OpenStack into OpenShift Virtualization, enabling consolidation of virtual machines and containers on a single Kubernetes-native platform.
{: shortdesc}

MTV provides a web UI and API for discovery, planning, and execution of migrations, leveraging persistent volume cloning or streaming for disk data, while maintaining virtual machine configuration and networking. It integrates with OpenShift Virtualization to run migrated virtual machines alongside containers, using Kubernetes-native storage and networking constructs.

For VMware migrations, the Migration Toolkit for Virtualization (MTV) integrates with vCenter to discover virtual machines and inventory data, map virtual machine resources (clusters, networks, datastores) to OpenShift equivalents, and automate bulk or selective migrations. It supports disk data transfer via warm or cold migration, preserves virtual machine configuration (CPU, memory, NICs), and converts VMware constructs into Kubernetes-native resources for seamless execution in OpenShift Virtualization.

## Red Hat OpenShift Migration Types
{: #virt-sol-openshift-migration-design-migration-type}

Migration Toolkit for Virtualization (MTV) supports two types of migration:

   - Cold migration is the default migration type where the source’s virtual machines are shutdown while the data is copied.
   - Warm migration copies most of the data during the precopy stage. Then the virtual machines are shut down and the remaining data is copied during the cutover stage.

Comparing the migration speeds of cold and warm migrations, you can observe single disk transfer and disk conversion are approximately the same for the warm and cold migrations. The benefit of warm migration is that the transfer of the snapshot happens in the background while the virtual machine is powered on. The default snapshot time is taken every 60 minutes. If virtual machines change substantially, more data needs to be transferred than in cold migration when the virtual machine is powered off. The cutover time, meaning the shutdown of the virtual machine and last snapshot transfer, depends on how much the virtual machine has changed since the last snapshot.

## Red Hat OpenShift cold migration
{: #virt-sol-openshift-cold-migration}

Cold migration is the default migration type. The source virtual machines are shut down while the data is copied.

To enable MTV to automatically install `qemu-guest-agent` on the migrated VMs, ensure that your package manager can install the daemon during the first boot of the VM after migration. If that is not possible, use your preferred automated or manual procedure to install `qemu-guest-agent` manually.
{: note}

## Red Hat OpenShift warm migration
{: #virt-sol-openshift-warm-migration}

In warm migration, the virtual machine is not shutdown during the precopy stage. This is a two step process. Most of the data is copied during the precopy stage while the source virtual machines (VMs) are running.Then the VMs are shut down and the remaining data is copied during the cutover stage.

### Warm migration precopy stage
{: #virt-sol-openshift-warm-migration-precopy}

The following processes is an overview of the precopy stage process.

1. Creates an initial snapshot of running virtual machine disks.
1. Copies the initial snapshot to target (full disk transfer, largest amount of data copied - takes more time).
1. The virtual machine disks are copied incrementally using changed block tracking (CBT) snapshots.
1. Copy deltas - Changed data (copies only the data, which has changed since last snapshot - takes less time).
      1. Creates a new snapshot.
      1. Copies the delta between previous snapshot and the new snapshot.
      1. Schedules the next snapshot (configurable, by default 1 hour after last snapshot finished).

A virtual machine can support up to 28 CBT snapshots. If that limit is exceeded, a warm import retry limit reached error message is displayed. If the virtual machine has preexisting CBT snapshots, it will reach this limit sooner.
{: note}

### Warm migration cutover stage
{: #virt-sol-openshift-warm-migration-cutover}

The virtual machines are shut down during the cutover stage and the remaining data is migrated. Data stored in RAM is not migrated.

The following process is an overview of the cutover stage process.

1. Schedules time to finalize warm migration.
1. You can start the cutover stage manually in the MTV console.
1. Continue in the same way as cold migration
   1. Guest conversion.
   1. Optionally starting target virtual machine.
