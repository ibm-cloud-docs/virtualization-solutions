---

copyright:
  years: 2026
lastupdated: "2026-06-29"

keywords: Red Hat OpenShift Virtualization, networking, VNI, OpenShift Virtualization VNI limitations, floating attachment restrictions, VNI modification known issues, cluster-scoped dynamic attachments, VNI security group changes, Infrastructure NAT VNI settings, OpenShift networking limitations, VNI detach reattach workflow, floating IP VNI restrictions, OpenShift VNI troubleshooting


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Known issues for networking
{: #known-issues-for-networking}

VNI properties for floating attachments cannot be modified directly. Detach the VNI, update settings like security groups or NAT, then reattach to cluster.
{: shortdesc}

Known issues and limitations might change as capabilities are added, so check back periodically.

## VNI modification restrictions for floating attachments
{: #vni-modification-restrictions-floating-attachments}

You cannot modify the properties of a virtual network interface (VNI) attached as a floating (cluster-scoped) dynamic attachment. Restricted properties include the VNI name, floating IP addresses, infrastructure NAT settings, and security group assignments. To change any of these settings, detach the VNI from the cluster, make the required updates, and then reattach the VNI. This limitation is temporary.

For more information, see [Limitations and considerations](/docs/openshift?topic=openshift-vni-virtualization#vni-limitations).
