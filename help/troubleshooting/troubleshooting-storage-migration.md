---

copyright:
  years: 2026
lastupdated: "2026-07-21"

keywords: storage migration troubleshooting OpenShift, Calico network policy rollback error, live migration size mismatch, MTC migration toolkit errors, KubeVirt live migration failures, PVC size mismatch migration, calico-tier-getter permissions, bandwidthPerMigration configuration, OpenShift VM storage migration, migration rollback RBAC issues


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Troubleshooting storage migration in IBM Cloud VPC
{: #troubleshooting-storage-migration}

Troubleshoot storage migration in IBM Cloud VPC, including Calico network policy forbidden errors and live migration size mismatch failures.
{: shortdesc}

If you encounter issues during storage migration, review the following common problems and their solutions.

## Error: Calico network policy forbidden (during rollback)
{: #error-calico-policy}

### What is happening?
{: #tsSymptoms-calico-policy}

**Error message**: "networkpolicies.projectcalico.org is forbidden: Operation on Calico tiered policy is forbidden."

### Why is it happening?
{: #tsCauses-calico-policy}

This error occurs during rollback operations when the Migration Toolkit attempts to restore Calico tiered network policies. The `calico-tier-getter` ClusterRole lacks the necessary permissions to perform operations on Calico tiered policies. Calico tiered policies require specific role-based access control (RBAC) permissions that are not included in the default ClusterRole configuration, which causes the rollback operation to fail when it tries to restore these network policy resources.

### How do you fix it?
{: #tsResolve-calico-policy}

Set the resources in the `calico-tier-getter` ClusterRole on your cluster.

For more information, refer to the [GitHub Issue #10110](https://github.com/projectcalico/calico/issues/10110#issuecomment-2791344397){: external}

## Error: Live migration failed with size mismatch
{: #error-live-migration-size}

### What is happening?
{: #tsSymptoms-live-migration-size}

**Error message**: "Failed Live migration failed error that is encountered during the MigrateToURI3 libvirt api call: virError(Code=1, Domain=10, message='internal error: process exited while connecting to monitor: ... The sum of offset (0) and size (0) must be smaller or equal to the actual size of the containing file ...')."

### Why is it happening?
{: #tsCauses-live-migration-size}

- The destination PVC is smaller than the source disk.
- Migrating between different storage operators.
- Using the `spec.liveMigrationConfig.bandwidthPerMigration: 0Mi` configuration.

### How do you fix it?
{: #tsResolve-live-migration-size}

1. Verify the VM storage class in {{site.data.keyword.redhat_openshift_notm}} Virtualization.
2. Check whether the migration succeeded despite the error.
3. Verify that the destination storage has adequate capacity.
4. Consider adjusting the `bandwidthPerMigration` setting.

For more information, refer to the following resources:

- [KubeVirt Issue #15348](https://github.com/kubevirt/kubevirt/issues/15348){: external}
- [KubeVirt Issue #12875](https://github.com/kubevirt/kubevirt/issues/12875#issuecomment-2487978345){: external}

## Additional troubleshooting resources
{: #additional-troubleshooting-resources}

For comprehensive troubleshooting guidance, see the [official troubleshooting documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.12/html/migration_toolkit_for_containers/troubleshooting-mtc){: external}.
