---

copyright:
  years: 2026
lastupdated: "2026-09-04"

keywords: storage migration OpenShift, storage class conversion, OpenShift Virtualization storage, live migration VMs, PVC migration OpenShift, persistent volume migration, VirtualMachineStorageMigrationPlan, cross-namespace VM migration, storage class migration, retentionPolicy deleteSource

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization
account-plan: paid
completion-time: 45m

---

{{site.data.keyword.attribute-definition-list}}

# Migrating virtual machine storage in Red Hat OpenShift Virtualization on IBM Cloud
{: #storage-migration-vms}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="45m"}

## Overview of the guide
{: #overview-of-the-guide}

Migrate VM disk storage between storage classes in Red Hat {{site.data.keyword.redhat_openshift_notm}} Virtualization on IBM Cloud without stopping your virtual machines.
{: shortdesc}

Storage migration moves a VM's persistent volume claims (PVCs) from one storage class to another within the same cluster. The VM continues running throughout the entire process via live storage migration. Expect a brief connection loss of approximately 1 second at the end of the migration during the final cutover phase.

This guide applies to any storage class migration scenario, including but not limited to:

- OpenShift Data Foundation (ODF) to IBM Cloud VPC File Storage (NFS), or vice versa
- ODF to ODF (different storage tiers or pools)
- NFS to NFS (different performance tiers)
- Any other supported storage class combination available in your cluster

Common reasons to migrate storage:

- Moving VMs to higher-performance or lower-cost storage tiers
- Consolidating workloads on specific storage backends
- Performing storage infrastructure changes without scheduling a maintenance window

## About native VM storage migration
{: #about-mtc}

Storage migration is **built into the {{site.data.keyword.redhat_openshift_notm}} Virtualization operator** — no additional operator installation is required. The migration controller uses `VirtualMachineStorageMigrationPlan` custom resources to move VM disk data between storage classes within the same cluster, using live storage migration so the VM stays running throughout.

In {{site.data.keyword.redhat_openshift_notm}} Virtualization 4.21 and later, storage migration is fully native. Earlier versions of {{site.data.keyword.redhat_openshift_notm}} Container Platform used the Migration Toolkit for Containers (MTC) operator for storage migrations. MTC is no longer required and is not used in this guide. For more information, see [Migration Toolkit for Containers — End of Life announcement](https://access.redhat.com/articles/7137482){: external}.
{: note}

### Key capabilities for VM storage migration
{: #key-capabilities}

- Migrate running VMs with **minimal downtime** (approximately 1-second connection loss at cutover) in {{site.data.keyword.redhat_openshift_notm}} Virtualization 4.21 or later
- Storage migration is **native to the OpenShift Virtualization operator** — no additional operator is needed
- Migrate a **single VM** from the VM detail view, **all VMs in a project** from the project context menu, or **selected VMs** using the selective volumes option — all from the UI
- Migration works between any two supported storage classes in the cluster

### Important limitations for VM storage migration
{: #important-limitations}

The following limitations apply to {{site.data.keyword.redhat_openshift_notm}} Virtualization virtual machine (VM) storage migrations:
{: important}

Same cluster only
:   VM storage migrations are supported only within the same {{site.data.keyword.redhat_openshift_notm}} cluster.

Source PVCs deleted by default
:   By default, source PVCs are automatically deleted after a successful migration (`deleteSource` retention policy). To enable rollback, explicitly retain source PVCs **before** starting migration. See [Source PVC retention and rollback](#source-pvc-behavior).

## Prerequisites
{: #prerequisites}

Before you perform storage migrations, verify that the following requirements are met:

### Required components
{: #required-components}

{{site.data.keyword.redhat_openshift_notm}} Virtualization operator
:   The operator must be installed and running. Storage migration is built into the operator — no additional installation is needed. For installation guidance, see [Installing the OpenShift Virtualization operator](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/virtualization/#installing-virt-operator_installing-virt){: external}.

{{site.data.keyword.redhat_openshift_notm}} Virtualization version
:   Version 4.21 or later is required. This guide has been validated on version 4.21. Earlier versions such as 4.18 used the Migration Toolkit for Containers (MTC) operator for storage migrations, which is now sunset.

### VM requirements
{: #vm-requirements}

Before you migrate, confirm the following for each VM:

- The VM is **powered on**. Live storage migration requires a running VM.
- The `StorageLiveMigratable` status condition is `True`. Run the following command to check:

   ```sh
   oc get vm <vm-name> -n <namespace> \
     -o jsonpath='{.status.conditions[?(@.type=="StorageLiveMigratable")].status}'
   ```
   {: pre}

   If the value is not `True`, the VM cannot be live-migrated. Resolve the condition before continuing.

- The cluster has at least two worker nodes. Live storage migration moves the VM to a different node during the process.

### Verify the target StorageClass StorageProfile
{: #verify-storageprofile}

The migration controller uses the StorageProfile of the target storage class to determine which access modes and volume modes to use for the destination PVC. If the StorageProfile has empty `claimPropertySets`, migration will stall silently. Ensure the StorageProfile of your target storage class is configured before starting migration. For more information, see [Configuring a StorageProfile](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/virtualization/storage#virt-configuring-storage-profile){: external}.

### Verify cluster health
{: #verify-cluster-health}

Confirm that all worker nodes are in `Ready` state and, if ODF is involved, that the ODF cluster reports `HEALTH_OK` before initiating migration:

```sh
oc get nodes --no-headers | awk '{print $1, $2}'
oc get cephcluster -n openshift-storage -o jsonpath='{.items[0].status.ceph.health}'
```
{: pre}

## Migrating VM storage
{: #perform-storage-migration}

All three migration scopes — single VM, all VMs in a project, or selected VMs — use the same underlying storage migration wizard and produce a `VirtualMachineStorageMigrationPlan` custom resource. The steps are identical except for how you open the wizard and which VMs you select.

### Step 1: Open the storage migration wizard
{: #open-wizard}

How you open the wizard depends on your migration scope:

Single VM
:   In the {{site.data.keyword.openshiftshort}} web console, click **Virtualization** > **Virtual Machines**. Select the VM, then click **Actions** > **Migration** > **Storage**.

All VMs in a project
:   In the {{site.data.keyword.openshiftshort}} web console, click **Virtualization** > **Virtual Machines**. In the left navigation tree, **right-click the project name** (namespace) and select **Migration** > **Storage**. The wizard header shows the total number of VMs and combined storage size.

Selected VMs in a project
:   Open the project-level wizard as described above. On the first wizard step, select **Selected volumes** instead of **Entire project**. A per-VM, per-disk checklist appears — check only the VMs and disks you want to include. VMs with no volumes checked are excluded from the plan entirely.

### Step 2: Select the destination storage class
{: #select-storageclass}

1. Select the target storage class from the dropdown list.

   The access mode and volume mode are set automatically based on the StorageProfile of the target storage class.

2. To retain the source PVC after migration (required for rollback), select **Keep original volumes at source after successful migration**. By default this option is **not selected** and the source PVC is deleted after a successful migration.

   If you plan to roll back the migration, select this option before clicking **Next**. Once migration completes with the default setting, the source PVC is deleted and rollback is not possible.
   {: important}

3. Click **Next**.

### Step 3: Review and start migration
{: #review-start}

1. Review the migration summary:
   - **Source storage class** — the current storage class
   - **Destination storage class** — the selected target storage class
   - **Post-migration cleanup** — either **Decommission source volumes** (default, source deleted) or **Retain source volumes**
2. Click **Migrate VirtualMachine storage**.

The wizard closes. The VM status changes to **Migrating** in the Virtual Machines list. The VM remains operational and accessible during migration. VMs in the same project migrate concurrently.

### Step 4: Monitor migration progress
{: #monitor-progress}

```sh
# Watch overall plan progress
oc get virtualmachinestoragemigrationplans -n <namespace> \
  -o custom-columns="NAME:.metadata.name,READY:.status.conditions[?(@.type==\"Ready\")].status,COMPLETED:.status.completedOutOf"

# Monitor individual live migration objects
oc get virtualmachineinstancemigration -n <namespace> \
  -o custom-columns="NAME:.metadata.name,PHASE:.status.phase,VM:.spec.vmiName" -w

# Watch all VM statuses
oc get vm -n <namespace> -w
```
{: pre}

Migration is complete when `completedOutOf` shows `1/1` (single VM) or `<n>/<n>` (multiple VMs), and all `VirtualMachineInstanceMigration` objects reach the `Succeeded` phase.

The live migration percentage progress bar in the UI is not always accurate. Use the CLI commands above for reliable status.
{: note}

## Post-migration validation
{: #post-migration-validation}

After any migration completes — whether single VM, entire project, or selected VMs — verify the following:

VM status
:   Confirm all migrated VMs are in **Running** state:

    ```sh
    oc get vmi -n <namespace> -o wide
    ```
    {: pre}

PVC storage class
:   Confirm the PVCs are bound and on the target storage class:

    ```sh
    oc get pvc -n <namespace> \
      -o custom-columns="NAME:.metadata.name,SC:.spec.storageClassName,STATUS:.status.phase"
    ```
    {: pre}

Source PVC cleanup
:   If the default `deleteSource` policy was used, confirm the old PVCs are no longer present. If `keepSource` was used, old PVCs remain and must be cleaned up manually when rollback is no longer needed:

    ```sh
    oc get pvc -n <namespace>
    oc get datavolume -n <namespace>
    ```
    {: pre}

Node placement
:   Confirm the VM has moved to a different worker node, which is expected behavior during live storage migration:

    ```sh
    oc get vmi <vm-name> -n <namespace> \
      -o jsonpath='{.status.nodeName}{"\n"}'
    ```
    {: pre}

## Source PVC retention and rollback
{: #source-pvc-behavior}

### Retention policy
{: #retention-policy}

Migration behavior is controlled by the `retentionPolicy` field in the migration plan:

| Policy | Default | Behavior |
|---|---|---|
| `deleteSource` | **Yes (UI default)** | Source PVCs and DataVolumes are deleted after migration completes. Rollback is not possible after this point. |
| `keepSource` | No | Source PVCs are retained. You can roll back by updating the VM to reference the original PVC. |
{: caption="Table 1. Migration retention policy options" caption-side="bottom"}

In the UI wizard, this is controlled by the **Keep original volumes at source after successful migration** checkbox. The checkbox is **unchecked by default** (`deleteSource`). The `retentionPolicy` field is **immutable** — it cannot be changed after the plan is created.
{: note}

### Manual rollback (keepSource plans only)
{: #manual-rollback}

If you retained source PVCs (`retentionPolicy: keepSource` or the **Keep original volumes** checkbox was selected), you can roll back by updating the VM spec to reference the original DataVolume:

```sh
# Get the current VM volume reference
oc get vm <vm-name> -n <namespace> \
  -o jsonpath='{.spec.template.spec.volumes[?(@.name=="rootdisk")].dataVolume.name}'

# Patch the VM to reference the original DataVolume
oc patch vm <vm-name> -n <namespace> --type=json -p '[
  {"op":"replace","path":"/spec/dataVolumeTemplates/0","value":{
    "apiVersion":"cdi.kubevirt.io/v1beta1",
    "kind":"DataVolume",
    "metadata":{"name":"<original-dv-name>"},
    "spec":{
      "source":{"blank":{}},
      "storage":{
        "resources":{"requests":{"storage":"<size>"}},
        "storageClassName":"<original-sc>"
      }
    }
  }},
  {"op":"replace","path":"/spec/template/spec/volumes/0","value":{
    "dataVolume":{"name":"<original-dv-name>"},
    "name":"rootdisk"
  }}
]'
```
{: pre}

After rollback, delete the unused target PVC and DataVolume:

```sh
oc delete datavolume <target-dv-name> -n <namespace>
```
{: pre}

## Deleting a completed migration plan
{: #deleting-migration-plan}

After migration completes, delete the migration plan and its associated trigger object. It is best practice to clean up completed plans before starting new migrations in the same namespace.

```sh
# List plans
oc get virtualmachinestoragemigrationplans -n <namespace>

# Delete the plan and its trigger
oc delete virtualmachinestoragemigrationplan <plan-name> -n <namespace>
oc delete virtualmachinestoragemigration <migration-name> -n <namespace>
```
{: pre}

Deleting a plan does not delete source or target PVCs. Clean up residual PVCs and DataVolumes manually if they are no longer needed.
{: note}

## Important considerations
{: #important-considerations}

Before you perform storage migrations, be aware of the following constraints and conditions:

### Migration constraints
{: #migration-constraints}

Brief connection loss at cutover
:   Storage migration uses live storage migration (`VirtualMachineInstanceMigration`). The VM remains running and accessible throughout the entire migration process. Expect a brief connection loss of approximately 1 second at the end of the migration during the final cutover phase.

Node migration required
:   The VM moves to a different worker node during live storage migration. This is expected behavior. The VM retains its IP address and network connectivity.

Delete completed plans before creating new ones
:   While the system can support multiple plans in a namespace, it is best practice to delete completed or stale plans before starting a new migration to keep the namespace clean and avoid confusion.

Use Selected volumes to limit scope
:   When migrating from the project context menu, the default scope is all VMs in the namespace. Use **Selected volumes** on the first wizard step to get a per-VM checklist and exclude VMs you do not need to migrate.

### Post-migration behavior
{: #post-migration-behavior}

Cleanup
:   When `keepSource` retention is used, old DataVolumes and PVCs are not automatically deleted. Manually clean up these resources after confirming successful migration and that rollback is no longer needed.

### Version compatibility
{: #version-compatibility}

{{site.data.keyword.redhat_openshift_notm}} Virtualization 4.21 and later
:   Supports native live storage migration with minimal downtime (approximately 1-second connection loss at cutover).

Version 4.17
:   Does not support live storage migration natively. The Migration Toolkit for Containers (MTC) operator was used in version 4.17 and earlier for storage migrations. MTC is sunset and is no longer the recommended approach. For more information, see [Migration Toolkit for Containers — End of Life announcement](https://access.redhat.com/articles/7137482){: external}.

## Additional resources
{: #additional-resources}

For more information about storage migration and related topics, see the following resources.

- [OpenShift Virtualization live migration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/virtualization/index#virt-configuring-live-migration){: external}
- [OpenShift Virtualization storage overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/virtualization/index#virt-storage-configuration-overview){: external}
- [Known issues for storage migration](/docs/virtualization-solutions?topic=virtualization-solutions-known-issues-storage-migration)
- [Troubleshooting storage migration](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-storage-migration)
