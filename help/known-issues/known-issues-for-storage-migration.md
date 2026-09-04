---

copyright:
  years: 2026
lastupdated: "2026-09-04"

keywords: storage migration known issues OpenShift, VirtualMachineStorageMigrationPlan existing plan error, storage class conversion problems, nodeSelector migration warnings, volume capacity mismatch migration, OpenShift storage class migration, namespace migration plan conflicts, PVC migration issues OpenShift, migration plan deletion workflow, SubnetFindFailed VPC file storage, deleteSource retentionPolicy, source PVC deleted migration, ibm-cloud-provider-data subnet vpc file csi driver, SubnetFindFailed RC:404, VPC subnet resource group mismatch

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Storage migration known issues and limitations
{: #known-issues-storage-migration}

Review known issues for VM storage migration on IBM Cloud OpenShift Virtualization.
{: shortdesc}

This list reflects known issues and limitations at the time of publication. Review this page periodically for updates as new capabilities are released.

## Common issues and solutions
{: #common-issues-and-solutions-storage-migration}

The following issues are commonly encountered during storage migration operations.

### Issue: Source PVCs deleted after migration
{: #issue-source-pvc-deleted}

**Symptom**: After migration completes, source PVCs and DataVolumes are no longer present. Rollback is not possible.

**Explanation**: The default retention policy is `deleteSource`. Source PVCs and DataVolumes are automatically deleted after a successful migration unless you explicitly retain them. In the UI wizard, this corresponds to the **Keep original volumes at source after successful migration** checkbox being **unchecked** (the default state).

**Workaround**: To retain source PVCs for rollback purposes, select **Keep original volumes at source after successful migration** in the migration wizard before clicking **Next**, or set `retentionPolicy: keepSource` in the CLI `VirtualMachineStorageMigrationPlan` spec.

The `retentionPolicy` field is immutable once the plan is created. You cannot change retention behavior after the plan exists.
{: important}

### Issue: Plan reports "No virtual machines are ready for storage migration"
{: #issue-vms-not-ready}

**Symptom**: The `VirtualMachineStorageMigrationPlan` status shows `Ready: False` with the message `No virtual machines are ready for storage migration`.

**Explanation**: The `volumeName` field in `targetMigrationPVCs` does not match the actual VM volume name. It must match the name under `spec.template.spec.volumes[].name` in the VM spec (commonly `rootdisk`), not the PVC or DataVolume name.

**Workaround**: Check the VM volume name and correct the plan:

```sh
oc get vm <vm-name> -n <namespace> \
  -o jsonpath='{.spec.template.spec.volumes[*].name}'
```
{: pre}

### Warning: Nondefault NodeSelector
{: #warning-nodeselector}

**Warning message**: "The system finds pods with nondefault `Spec.NodeSelector` set."

**Explanation**: A VM uses a node selector or node affinity rule that may restrict which nodes the live migration can target.

**Workaround**: Review the VM's scheduling constraints and ensure that at least one eligible destination node matches the selector. No action is required if sufficient eligible nodes exist.

### Warning: Volume capacity mismatch
{: #warning-volume-capacity}

**Warning message**: "Migrating data of the following volumes can result in a failure either due to mismatch in their requested and actual capacities or disk usage close to 100%."

**Explanation**: The storage class API does not report which access modes it supports or whether a capacity mismatch exists between the source and target PVC.

**Workaround**: Verify that the destination storage class supports the required access modes and has sufficient capacity before migration.

For more information, refer to the [Red Hat Solution Article](https://access.redhat.com/solutions/6497941){: external}.

### Issue: VM cannot start after failed migration — DataVolume recreated with wrong volumeMode
{: #issue-dv-volumemode-corrupt}

**Symptom**: After a stalled or cancelled migration targeting an NFS storage class, the VM's DataVolume is recreated with `volumeMode: Block`, but the NFS storage class requires `volumeMode: Filesystem`. The PVC stays `Pending` indefinitely.

**Explanation**: The migration controller may update the VM's `dataVolumeTemplates` with an incorrect `volumeMode` for the target storage class during a stalled migration. Because the DataVolume spec is immutable, the PVC cannot be corrected without recreating the DataVolume. The VM's `dataVolumeTemplates` must also be patched before the DataVolume can be deleted — otherwise the VM controller immediately recreates it.

**Workaround**: Follow the recovery procedure in the [VM stuck after failed migration](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-storage-migration#error-vm-spec-updated) troubleshooting section.

### Issue: NFS target migration fails with SubnetFindFailed — VPC subnet and resource group mismatch
{: #issue-subnet-find-failed}

**Symptom**: After starting a migration to an IBM Cloud VPC File Storage (NFS) storage class, all target PVCs remain in `Pending` state. Cluster events show:

```
SubnetFindFailed RC:404
```
{: screen}

**Explanation**: The IBM Cloud VPC File CSI driver queries VPC subnets filtered by the resource group configured in its `storage-secret-store` secret. If the `vpc_subnet_ids` value in the `ibm-cloud-provider-data` ConfigMap in `kube-system` points to a subnet that exists in a different resource group — for example, an orphaned resource group — the CSI driver cannot find the subnet and all NFS PVC provisioning fails.

**Workaround**: For resolution steps, see [VPC File Storage PVC fails due to subnet or resource group mismatch](https://cloud.ibm.com/docs/containers?topic=containers-ts-storage-vpc-file-eit-pvc-fails){: external}.
