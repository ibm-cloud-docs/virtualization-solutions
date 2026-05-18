---

copyright:
  years: 2026
lastupdated: "2026-05-18"

keywords: Red Hat OpenShift Virtualization, virtual servers, Red Hat OpenShift Kubernetes Service, VSI, ODF, RBD

subcollection: virtualization-solutions

content-type: known-issues
services: OpenShift Virtualization

---

{{site.data.keyword.attribute-definition-list}}

# Known issues for storage migration
{: #known-issues-storage-migration}

Known issues and limitations might change as capabilities are added, so check back periodically.

## Common issues and solutions
{: #common-issues-and-solutions-storage-migration}

The following issues are commonly encountered during storage migration operations.

### Issue: Existing MigPlan found
{: #issue-existing-migplan}

**Error message**: "The system finds an existing MigPlan for this namespace. Click Storage Migrations to review and delete existing MigPlans."

**Explanation**: This error occurs when you attempt to create a new migration plan for a namespace that already has an existing MigPlan. The Migration Toolkit for Virtualization (MTV) does not allow multiple migration plans for the same namespace to prevent conflicts and ensure data consistency during the migration process. You must delete the existing plan before you can create a new one.

**Workaround**:

1. In the {{site.data.keyword.openshiftshort}} web console, click **Migration** > **Storage migrations**.
2. Select your namespace in the **Project** list.
3. Click the ellipsis **(⋮)** for the existing plan.
4. Click **Delete MigPlan**.

### Warning: Non-default NodeSelector
{: #warning-nodeselector}

**Warning message**: "The system finds pods with non-default `Spec.NodeSelector` set in namespace: []. The system clears this field on pods that it restores into the target cluster."

**Explanation**: Source pods contain hardcoded `nodeSelector` fields that target specific nodes that might not exist on the target cluster. The operator automatically removes these constraints to prevent migration failure, which allows the system to schedule pods anywhere on the target cluster.

**Workaround**: No action is required. This message is informational.

### Warning: Volume capacity mismatch
{: #warning-volume-capacity}

**Warning message**: "Migrating data of the following volumes might result in a failure either due to mismatch in their requested and actual capacities or disk usage being close to 100%: [volume]."

**Explanation**: The storage class API does not report which access modes it supports or whether a capacity mismatch exists.

**Workaround**: Verify that the destination storage class supports the required access modes and has sufficient capacity.

For more information, refer to the [Red Hat Solution Article](https://access.redhat.com/solutions/6497941){: external}.
