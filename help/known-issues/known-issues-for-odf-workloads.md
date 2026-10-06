---

copyright:
  years: 2026
lastupdated: "2026-10-06"

keywords: ODF known issues IBM Cloud, NVMe drive mounting problems IBM Cloud, ROKS bare metal NVMe issues, ODF storage node troubleshooting IBM Cloud, NVMe disk recovery OpenShift IBM Cloud, bare metal NVMe disk visibility IBM Cloud, ODF bare metal known issues IBM Cloud, OpenShift Data Foundation issues IBM Cloud, NVMe drive verification ROKS, ROKS cluster provisioning NVMe IBM Cloud


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# {{site.data.keyword.redhat_openshift_notm}} Virtualization ODF workloads: Known issues
{: #known-issues-for-odf-workloads}

Review known ODF storage workload issues on {{site.data.keyword.redhat_openshift_notm}} Virtualization on {{site.data.keyword.cloud_notm}}, including NVMe drive mounting failures and recommended workarounds.
{: shortdesc}

This list reflects known issues and limitations at the time of publication. Review this page periodically for updates as new capabilities are released.

## NVMe drives not mounting correctly after {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster provisioning
{: #nvme-drives-not-mounting-after-cluster-provisioning}

**Issue**: In some cases, NVMe drives do not mount correctly after the {{site.data.keyword.redhat_openshift_full}} Kubernetes Service cluster is provisioned.

**Workaround** Verify that all NVMe disks are visible on each host after the {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster is provisioned. For more information, see the [Verification and Recovery of Missing NVMe Disks](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-odf-workloads#verification-and-recovery-of-missing-nvme-disks) in the troubleshooting page.
