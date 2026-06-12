---

copyright:
  years: 2026
lastupdated: "2026-06-11"

keywords: Red Hat OpenShift Virtualization, virtual servers, Red Hat OpenShift Kubernetes Service, VSI, ODF, RBD, OpenShift Data Foundation ODF issues, NVMe drive mounting problems, ROKS cluster provisioning NVMe, bare metal NVMe disk visibility, ODF storage node troubleshooting, NVMe disk recovery OpenShift, cluster provisioning storage issues, OpenShift NVMe configuration, ODF bare metal known issues, NVMe drive verification ROKS


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Known issues for Red Hat OpenShift Data Foundation workloads
{: #known-issues-for-odf-workloads}

NVMe drives may fail to mount after ROKS cluster provisioning. Verify disk visibility on each bare metal host and recover missing NVMe disks as needed.
{: shortdesc}

Known issues and limitations might change as capabilities are added, so check back periodically.

## NVMe drives not mounting correctly after Red Hat OpenShift Kubernetes Service cluster provisioning
{: #nvme-drives-not-mounting-after-cluster-provisioning}

**Issue**:  Some NVMe drives might not mount correctly after the {{site.data.keyword.redhat_openshift_full}} Kubernetes Service cluster is provisioned.

**Workaround** Verify that all NVMe disks are visible on each host after the {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster is provisioned. For more information, see the [Verification and Recovery of Missing NVMe Disks](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-odf-workloads#verification-and-recovery-of-missing-nvme-disks) in the troubleshooting page.
