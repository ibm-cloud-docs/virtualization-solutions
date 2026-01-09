---

copyright:
  years: 2025
lastupdated: "2026-01-09"

keywords:

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Red Hat OpenShift Migration Infrastucture Design
{: #virt-sol-openshift-migration-design-infrastructure}

When migrating workloads with the Migration Toolkit for Virtualization, you need a reliable, high-bandwidth connectivity between the source environment (e.g., vCenter/ESXi or RHV) and OpenShift workers nodes on the target cluster to ensure uninterrupted data transfer. If firewalls and in place, the necessary ports for vCenter and ESXi communication must be opened. In addition to the basic network connectivity, Domain Name Service (DNS) resolution is required between the environments.

## Infrastucture Design for migrating from Classic Infrastucture
{: #virt-sol-openshift-migration-design-classic}

When migrating from {{site.data.keyword.vmwaresolutions_full}} to {{site.data.keyword.openshiftlong_notm}}, customers can use {{site.data.keyword.tg_full}} to establish private connectivity between the VMware (source) platform and a new target {{site.data.keyword.openshiftlong_notm}} Virtualization environment. The following example presents an high level overview of the infrastructure design for a migration.  In the VPC hosting {{site.data.keyword.openshiftlong_notm}}, {{site.data.keyword.dns_full_notm}} and DNS Custom Resolvers can be used for this purpose.

![Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC](../../../images/openshift/openshift-virtualization-mtv-design.svg "Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC"){: caption="Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC" caption-side="bottom"}
