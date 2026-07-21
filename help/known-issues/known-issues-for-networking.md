---

copyright:
  years: 2026
lastupdated: "2026-07-21"

keywords: OpenShift Virtualization networking known issues, VNI limitations, floating attachment restrictions, VNI modification known issues, cluster-scoped dynamic attachments, VNI security group changes, Infrastructure NAT VNI settings, OpenShift networking limitations, VNI detach reattach workflow, floating IP VNI restrictions, OpenShift VNI troubleshooting


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Known issues for networking in Red Hat OpenShift Virtualization on IBM Cloud
{: #known-issues-for-networking}

Review known networking issues for Red Hat OpenShift Virtualization on IBM Cloud, including VNI modification restrictions and workarounds.
{: shortdesc}

This list reflects known issues and limitations at the time of publication. Review this page periodically for updates as new capabilities are released.

## VNI modification restrictions for floating attachments
{: #vni-modification-restrictions-floating-attachments}

You cannot modify the properties of a virtual network interface (VNI) that is attached as a floating (cluster-scoped) dynamic attachment. Restricted properties include the VNI name, floating IP addresses, infrastructure network address translation (NAT) settings, and security group assignments. To change any of these settings, detach the VNI from the cluster, make the required updates, and then reattach the VNI. This limitation is temporary.

For more information, see [Limitations and considerations](/docs/openshift?topic=openshift-vni-virtualization#vni-limitations).
