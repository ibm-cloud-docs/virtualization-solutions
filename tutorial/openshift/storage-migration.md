---

copyright:
  years: 2026
lastupdated: "2026-05-22"

keywords: Red Hat OpenShift Virtualization, Red Hat OpenShift Kubernetes Service, VSI, storage migration, IBM Cloud, Storage vMotion, MTC

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 60m

---

{{site.data.keyword.attribute-definition-list}}

# Storage migration for virtual machines in OpenShift Virtualization
{: #storage-migration-vms}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}

## Overview of the guide
{: #overview-of-the-guide}

This solution guide explains how to perform storage migrations for virtual machines (VMs) that run in {{site.data.keyword.redhat_openshift_notm}} Virtualization using the Migration Toolkit for Containers (MTC) operator. For users who are familiar with VMware&reg;, this feature is equivalent to Storage vMotion.
{: shortdesc}

Storage migration helps you move VM disk data between storage classes within the same {{site.data.keyword.redhat_openshift_notm}} cluster and namespace, enabling you to:

- Migrate VMs to higher-performance storage
- Optimize storage costs by moving to different storage tiers
- Consolidate workloads on specific storage backends
- Perform storage infrastructure upgrades with minimal downtime

## About Migration Toolkit for Containers (MTC)
{: #about-mtc}

MTC is an operator that enables the migration of stateful application workloads between {{site.data.keyword.redhat_openshift_notm}} Container Platform clusters at the namespace level. MTC provides both a web console and an API that uses Kubernetes&reg; custom resources to help you control migrations and minimize application downtime.

### Key capabilities for VM storage migration
{: #key-capabilities}

When you use MTC for {{site.data.keyword.redhat_openshift_notm}} Virtualization VM storage migrations, you can perform the following tasks:

- Move VM storage between different storage classes within the same namespace.
- Migrate running VMs with minimal downtime in ({{site.data.keyword.redhat_openshift_notm}} Virtualization 4.18 or later).
- Use stage, cutover, and rollback capabilities for flexible migration control.
- Migrate between different storage providers (for example, OpenShift Data Foundation (ODF) to Network File System (NFS) or vice versa).

### Important limitations for VM storage migration
{: #important-limitations}

The following limitations apply to {{site.data.keyword.redhat_openshift_notm}} Virtualization VM storage migrations. While MTC can migrate general persistent volume claims (PVCs) across namespaces and clusters, VM-attached PVCs require stricter handling.
{: important}

Same cluster only
:   The system supports VM storage migrations only within the same {{site.data.keyword.redhat_openshift_notm}} cluster.

Same namespace only
:   VM storage migrations stay in the same namespace. The system does not support cross-namespace migration for VMs.

Single migration plan
:   The system supports only one migration plan per namespace at a time.

For non-VM workloads, MTC supports migration across different clusters and namespaces. However, for {{site.data.keyword.redhat_openshift_notm}} Virtualization VMs, the same-cluster and same-namespace restrictions always apply.

For more information, refer to the [official MTC documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#about-mtc){: external}.

## Prerequisites
{: #prerequisites}

Before you perform storage migrations, verify that the following requirements are met:

### Required components
{: #required-components}

{{site.data.keyword.redhat_openshift_notm}} Virtualization operator
:   You must install and configure the operator. For more information, see the [installation guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/virtualization/#installing-virt-operator_installing-virt){: external}.

{{site.data.keyword.redhat_openshift_notm}} Virtualization version
:   You need version 4.18 or later.

- Version 4.17 does not support live migration.
- Versions earlier than 4.17 require enabling a feature gate, which disables cluster-level support.

VM requirements
:   The following apply to VMs:

- You must power on VMs for live migration.
- VMs can migrate to a different node.
- Check the `StorageLiveMigratable` status condition of the VM object to confirm migration capability.

### Configuring live migration in OpenShift Virtualization
{: #live-migration-config}

Configure the live migration settings in your cluster to ensure that migration processes do not overwhelm the cluster. For more information about live migration configuration, see [Configuring live migration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/virtualization/index#virt-configuring-live-migration){: external}.

## Installation
{: #installation}

Complete the following steps to install the Migration Toolkit for Containers operator:

### Step 1: Install the MTC operator
{: #install-mtc-operator}

1. In the {{site.data.keyword.openshiftshort}} web console, click **Ecosystem** > **Software catalog**.
2. In the **Filter by keyword** field, search for **Migration Toolkit for Containers operator**.
3. Select the **Migration Toolkit for Containers operator** and click **Install**.
4. Click **Install** to confirm.
5. Verify the installation by completing the following steps:
   1. Click **Operators** > **Installed operators**.
   2. Confirm that the **Migration Toolkit for Containers operator** appears in the `openshift-migration` project with the **Succeeded** status.

### Step 2: Create migration controller instance
{: #create-migration-controller}

1. Click **Migration Toolkit for Containers operator**.
2. Under **Provided APIs**, locate the **Migration controller** tile.
3. Click **Create Instance**.
4. Click **Create**.
5. Verify that the MTC pods are running by completing the following steps:
   1. Click **Workloads** > **Pods**.
   2. Confirm that all MTC pods are in the **Running** state.

### Step 3: Create MigCluster resource
{: #create-migcluster}

1. Click **Migration Toolkit for Containers operator**.
2. Under **Provided APIs**, locate the **MigCluster** tile.
3. Click **Create MigCluster** (if the system does not create it automatically).
4. Click **Create**.

For detailed installation instructions, refer to the [official installation guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/migration_toolkit_for_containers/installing-mtc){: external}.

## How to perform storage migration
{: #perform-storage-migration}

You can perform storage migrations by using the following interfaces:

{{site.data.keyword.redhat_openshift_notm}} Virtualization UI
:   Best for migrating a single VM.

Migration Toolkit for Containers UI
:   Recommended for migrating multiple VMs.

### Method 1: Using OpenShift Virtualization UI (single VM)
{: #method-virtualization-ui}

This method is ideal for quick, single-VM migrations:

1. In the {{site.data.keyword.openshiftshort}} web console, click **Virtualization** > **Virtual Machines**.
2. Select the virtual machine that you want to migrate.
3. Click **Actions** > **Migration** > **Storage**.
4. Select the disks to migrate. Choose to migrate the entire VM or specific disks, and then click **Next**.
5. Select the destination storage class.
6. Review the migration plan and click **Migrate VirtualMachine storage**.

The VM status changes from **Ready** to **Migrating**. The VM remains operational and accessible during migration.

If you see the message "An existing MigPlan exists for this namespace," refer to the troubleshooting section.
{: note}

### Method 2: Using Migration Toolkit for Containers UI (multiple VMs)
{: #method-mtc-ui}

This method provides better control and is recommended for migrating multiple VMs:

#### Accessing the MTC UI
{: #accessing-mtc-ui}

1. In the {{site.data.keyword.openshiftshort}} web console, click **Networking** > **Routes**.
2. Filter by the `openshift-migration` project.
3. Locate the **migration** route.
4. Click the URL in the **Location** column.

#### MTC UI overview
{: #mtc-ui-overview}

The MTC UI includes the following sections:

Clusters
:   Add multiple {{site.data.keyword.redhat_openshift_notm}} clusters. VM storage migrations do not use this section.

Replication repositories
:   Configure S3 buckets for cross-cluster and cross-namespace migrations. VM storage migrations do not use this section.

Migration plans
:   Create and manage migration plans.

Hooks
:   Add custom scripts to run before or after migrations.

#### Creating a migration plan
{: #creating-migration-plan}

1. Click **Migration Plans** > **Add migration plan**.
2. Configure the general settings by completing the following steps:
   1. Enter a plan name.
   2. Select the **Storage class conversion** as the migration type.
   3. Select your cluster in the source cluster selection.
   4. Click **Next**.
3. Select the namespace by completing the following steps:
   1. Choose your namespace.
   2. Click **Next**.

   The system does not display the `default` namespace. Use the {{site.data.keyword.redhat_openshift_notm}} Virtualization UI for VMs in the default namespace.
   {: note}

4. Select the persistent volumes by completing the following steps:
   1. Select the persistent volumes (PVs) to migrate.
   2. Choose multiple VMs or PVs as needed.
   3. Configure the following settings for each PV:
      - Target storage class
      - Target volume mode
      - Target access mode
   4. Click **Next**.

   This step might take longer if the namespace contains many resources.
   {: note}

5. Enable live migration by completing the following steps:
   1. Select the checkbox to enable live migration.
   2. Click **Finish**.

#### Running the migration
{: #executing-migration}

After your migration plan is created, it is listed under **Migration plans**. Select your plan and choose one of the following actions:

Stage
:   Copies data to the target storage without stopping the VM. You can run this action multiple times to reduce cutover duration.

Cutover
:   Completes the migration by switching the VM to the new storage. The VM loses connection for approximately 1 second during this phase.

Rollback
:   Reverts a completed migration if needed.

Even with live migration enabled, expect a brief (approximately 1-second) connection loss during the cutover phase.
{: important}

For more information on the migration workflow, see the [official workflow documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#migration-mtc-workflow_about-mtc){: external}.

## Migration hooks
{: #migration-hooks}

Migration hooks help you customize the migration process by running scripts at different phases. You can add up to four hooks per migration plan.

### Hook execution phases
{: #hook-execution-phases}

| Phase | Description |
|-------|-------------|
| PreBackup | Runs before the system backs up resources on the source cluster. |
| PostBackup | Runs after the system backs up resources on the source cluster. |
| PreRestore | Runs before the system restores resources on the target cluster. |
| PostRestore | Runs after the system restores resources on the target cluster. |
{: caption="Table 1. Migration hook execution phases" caption-side="bottom"}

### Common use cases
{: #hook-use-cases}

- Customizing application quiescence
- Manually migrating unsupported data types
- Updating applications after migration
- Running validation scripts

For detailed information on how to create and use hooks, see the [Migration hooks documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#migration-hooks_advanced-migration-options-mtc){: external}.

## Important considerations
{: #important-considerations}

Before you perform storage migrations, be aware of the following constraints/conditions:

### Migration constraints
{: #migration-constraints}

Node migration required
:   The system migrates VMs to a different node.

Namespace limitation
:   The system does not support cross-namespace VM migration.

Brief connection loss
:   The VM loses connection for approximately 1 second at the end of the migration process.

VM state
:   You must power on VMs for live migration.

Single plan limitation
:   The system supports only one migration plan per namespace.
    - To perform additional migrations in the same namespace, delete the current plan first.
    - The system does not support modification of existing plans.
    - Historical tracking of previous migrations does not exist.

### Post-migration behavior
{: #post-migration-behavior}

Cleanup
:   The system does not automatically delete old data volumes (DVs) and PVCs.
    - The system retains these resources for rollback purposes.
    - You must manually clean up if you do not need rollback.

### UI limitations
{: #ui-limitations}

Progress indicators
:   Live migration percentage progress bar and elapsed time are not always reliable.

Default namespace
:   The MTC UI does not list VMs under the `default` namespace. Use the {{site.data.keyword.redhat_openshift_notm}} Virtualization UI for VMs in the default namespace.

### Version compatibility
{: #version-compatibility}

{{site.data.keyword.redhat_openshift_notm}} Virtualization 4.17
:   Version 4.17 does not support live migration.

Earlier versions
:   Versions earlier than 4.17 require enabling a feature gate, which removes cluster-level support.

For a complete list of known issues, see the [Known issues documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#mtc-migrating-vms-known-issues_mtc-migrating-vms){: external}.

## Additional resources
{: #additional-resources}

For more information about storage migration and related topics, see the following resources.

### Official documentation
{: #official-documentation}

For comprehensive information about MTC and {{site.data.keyword.redhat_openshift_notm}} Virtualization migration, refer to the following resources.

- [Migration Toolkit for Containers overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#about-mtc){: external}
- [Installing MTC](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/migration_toolkit_for_containers/installing-mtc){: external}
- [OpenShift Virtualization live migration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/virtualization/index#virt-configuring-live-migration){: external}
- [Migration controller options](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migrating_from_version_3_to_4/#migration-controller-options_advanced-migration-options-3-4){: external}
- [Migration workflow](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#migration-mtc-workflow_about-mtc){: external}
- [Known issues](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html-single/migration_toolkit_for_containers/index#mtc-migrating-vms-known-issues_mtc-migrating-vms){: external}
- [Known issues for storage migration](/docs/virtualization-solutions?topic=virtualization-solutions-known-issues-storage-migration)
- [Troubleshooting storage migration](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-storage-migration)

### Support
{: #support}

For additional assistance or to report issues, contact Red Hat Support or consult the OpenShift Virtualization community forums.
