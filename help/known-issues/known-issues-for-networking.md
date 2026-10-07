---

copyright:
  years: 2026
lastupdated: "2026-10-06"

keywords: OpenShift Virtualization networking known issues IBM Cloud, VNI limitations IBM Cloud, floating IP VNI restrictions IBM Cloud, VNI modification known issues ROKS, cluster-scoped dynamic attachments OpenShift, VNI security group changes IBM Cloud, Infrastructure NAT VNI settings, OpenShift networking limitations IBM Cloud, VNI detach reattach workflow, OpenShift Virtualization VNI IBM Cloud


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# {{site.data.keyword.redhat_openshift_notm}} Virtualization networking: Known issues
{: #known-issues-for-networking}

Review known {{site.data.keyword.redhat_openshift_notm}} Virtualization networking issues on {{site.data.keyword.cloud_notm}}, including VNI attachment modification restrictions and available workarounds for each issue.
{: shortdesc}

This list reflects known issues and limitations at the time of publication. Review this page periodically for updates as new capabilities are released.

## VNI modification restrictions for floating attachments
{: #vni-modification-restrictions-floating-attachments}

You cannot modify the properties of a virtual network interface (VNI) that is attached as a floating (cluster-scoped) dynamic attachment. Restricted properties include the VNI name, floating IP addresses, infrastructure network address translation (NAT) settings, and security group assignments. To change any of these settings, detach the VNI from the cluster, make the required updates, and then reattach the VNI. This limitation is temporary.

For more information, see [Limitations and considerations](/docs/openshift?topic=openshift-vni-virtualization#vni-limitations).
