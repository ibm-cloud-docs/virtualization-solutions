---

copyright:
  years: 2025
lastupdated: "2026-07-21"

keywords: OpenShift virtualization migration, migrate VMware to OpenShift, Migration Toolkit for Virtualization, MTV OpenShift, VMware vSphere to OpenShift, Red Hat migration consulting, OVA file migration, RHOSP to OpenShift, virtual machine migration OpenShift

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}



# Designing the migration infrastructure for Red Hat OpenShift Virtualization on IBM Cloud
{: #virt-sol-openshift-migration-design-infrastructure}

Design the network and Domain Name Service (DNS) infrastructure needed to connect VMware source environments to Red Hat OpenShift Virtualization using the Migration Toolkit for Virtualization (MTV).
{: shortdesc}

When you migrate workloads with the Migration Toolkit for Virtualization (MTV), you need a reliable, high-bandwidth connectivity between the source environment (for example, vCenter/ESXi or Red Hat Virtualization (RHV)) and Red Hat OpenShift worker nodes on the target cluster. It helps ensure uninterrupted data transfer. If firewalls are in place, the necessary ports for vCenter and ESXi communication must be opened. In addition to the basic network connectivity, DNS resolution is required between the environments.

## Infrastructure design for migrating from Classic infrastructure
{: #virt-sol-openshift-migration-design-classic}

When you migrate from {{site.data.keyword.vmwaresolutions_full}} to {{site.data.keyword.openshiftlong_notm}}, customers can use {{site.data.keyword.tg_full}} to establish private connectivity between the VMware (source) platform and a new target {{site.data.keyword.openshiftlong_notm}} Virtualization environment. The following example presents a high-level overview of the infrastructure design for a migration. In the VPC hosting {{site.data.keyword.openshiftlong_notm}}, {{site.data.keyword.dns_full_notm}} and DNS Custom Resolvers can be used for this purpose.

![Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC](../../../images/openshift/openshift-virtualization-mtv-design.svg "Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC"){: caption="Migration Infrastructure Design from Classic to {{site.data.keyword.openshiftlong_notm}} on VPC" caption-side="bottom"}
