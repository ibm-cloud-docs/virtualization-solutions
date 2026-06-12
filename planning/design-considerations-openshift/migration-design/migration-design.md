---

copyright:
  years: 2025
lastupdated: "2026-06-11"

keywords: openShift virtualization migration, migrate VMware to OpenShift, Migration Toolkit for Virtualization, MTV OpenShift, VMware vSphere to OpenShift, IBM Cloud migration services, Red Hat migration consulting, OVA file migration, RHOSP to OpenShift, virtual machine migration OpenShift

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}



# Red Hat OpenShift Migration Infrastructure Design
{: #virt-sol-openshift-migration-design-infrastructure}

Migrate virtual machines from VMware vSphere, Red Hat platforms, or OpenShift clusters to IBM Cloud OpenShift Virtualization using Migration Toolkit for Virtualization.
{: shortdesc}

When you migrate workloads with the Migration Toolkit for Virtualization, you need a reliable, high-bandwidth connectivity between the source environment (for example, vCenter/ESXi or RHV) and Red Hat OpenShift workers nodes on the target cluster. It help ensure uninterrupted data transfer. If firewalls are in place, the necessary ports for vCenter and ESXi communication must be opened. In addition to the basic network connectivity, Domain Name Service (DNS) resolution is required between the environments.

## Infrastructure Design for migrating from Classic Infrastructure
{: #virt-sol-openshift-migration-design-classic}

When you migrate from {{site.data.keyword.vmwaresolutions_full}} to {{site.data.keyword.openshiftlong_notm}}, customers can use {{site.data.keyword.tg_full}} to establish private connectivity between the VMware (source) platform and a new target {{site.data.keyword.openshiftlong_notm}} Virtualization environment. The following example presents a high-level overview of the infrastructure design for a migration. In the VPC hosting {{site.data.keyword.openshiftlong_notm}}, {{site.data.keyword.dns_full_notm}} and DNS Custom Resolvers can be used for this purpose.

![Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC](../../../images/openshift/openshift-virtualization-mtv-design.svg "Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC"){: caption="Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC" caption-side="bottom"}
