---

copyright:
  years: 2026, 2026
lastupdated: "2026-03-27"

keywords: Red Hat OpenShift Virtualization, virtual servers, ROKS, VSI, ODF, RBD

subcollection: virtualization-solutions

content-type: known-issues
services: OpenShift Virtualization, VMware

---

{{site.data.keyword.attribute-definition-list}}

# Known issues for ODF workloads
{: #known-issues-for-odf-workloads}

Known issues and limitations might change as capabilities are added, so check back periodically.

## NVMe drives not mounting correctly after ROKS cluster provisioning.

**Issue**: Due to a known issue, some NVMe drives might not mount correctly. Verify that all NVMe disks are visible on each host after the ROKS cluster is provisioned. For more information, see the [Verification and Recovery of Missing NVMe Disks](docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-for-odf-workloads#verification-and-recovery-of-missing-nvme-disks) in the troubleshooting page.
