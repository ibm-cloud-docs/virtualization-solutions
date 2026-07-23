---

copyright:
  years: 2026
lastupdated: "2026-07-21"

keywords: OpenShift Data Foundation ODF issues, NVMe drive mounting problems, ROKS cluster provisioning NVMe, bare metal NVMe disk visibility, ODF storage node troubleshooting, NVMe disk recovery OpenShift, cluster provisioning storage issues, OpenShift NVMe configuration, ODF bare metal known issues, NVMe drive verification ROKS


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Known issues and limitations for Red Hat OpenShift Data Foundation workloads
{: #known-issues-for-odf-workloads}

Review known issues for Red Hat OpenShift Data Foundation workloads on IBM Cloud, including NVMe drive mounting failures and workarounds.
{: shortdesc}

This list reflects known issues and limitations at the time of publication. Review this page periodically for updates as new capabilities are released.

## NVMe drives not mounting correctly after Red Hat OpenShift Kubernetes Service cluster provisioning
{: #nvme-drives-not-mounting-after-cluster-provisioning}

**Issue**: In some cases, NVMe drives do not mount correctly after the {{site.data.keyword.redhat_openshift_full}} Kubernetes Service cluster is provisioned.

**Workaround** Verify that all NVMe disks are visible on each host after the {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster is provisioned. For more information, see the [Verification and Recovery of Missing NVMe Disks](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-odf-workloads#verification-and-recovery-of-missing-nvme-disks) in the troubleshooting page.
