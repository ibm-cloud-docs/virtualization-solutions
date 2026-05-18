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

# Known issues for Red Hat OpenShift Data Foundation workloads
{: #known-issues-for-odf-workloads}
{: toc-content-type="known-issues"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}

Known issues and limitations might change as capabilities are added, so check back periodically.

## NVMe drives not mounting correctly after Red Hat OpenShift Kubernetes Service cluster provisioning
{: #nvme-drives-not-mounting-after-cluster-provisioning}

**Issue**:  Some NVMe drives might not mount correctly after the {{site.data.keyword.redhat_openshift_full}} Kubernetes Service cluster is provisioned.

**Workaround** Verify that all NVMe disks are visible on each host after the {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster is provisioned. For more information, see the [Verification and Recovery of Missing NVMe Disks](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-odf-workloads#verification-and-recovery-of-missing-nvme-disks) in the troubleshooting page.
