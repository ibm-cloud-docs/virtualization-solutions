---

copyright:
  years: 2025, 2026
lastupdated: "2026-07-21"


keywords: OpenShift migration workflow, Migration Toolkit for Virtualization workflow, MTV custom resources, OpenShift Operator migration, NetworkMapping MTV, StorageMapping MTV, migration plan OpenShift, vCenter migration workflow, network attachment definitions OpenShift

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}



# Migrating VMware workloads to Red Hat OpenShift Virtualization: Step-by-step workflow
{: #virt-sol-openshift-migration-design-migration-workflow}

Understand how the Migration Toolkit for Virtualization (MTV) Operator manages custom resources — Provider, Plan, Migration — to execute VMware virtual machine (VM) migrations in Red Hat OpenShift.
{: shortdesc}

The Migration Toolkit for Virtualization (MTV) is provided as a Red Hat OpenShift Operator.

![Red Hat OpenShift Virtualization Migration Workflow](../../../images/openshift/openshift-virtualization-migration-flow.svg "Red Hat OpenShift Virtualization Migration Workflow"){: caption="Red Hat OpenShift Virtualization Migration Workflow" caption-side="bottom"}

MTV creates and manages the following custom resources (CRs) and services. The following table lists each MTV option with description.

| Option | Description |
| -------------- | -------------- |
| `NetworkMapping` | Maps the networks of the source and target providers. |
| `StorageMapping` | Maps the storage of the source and target providers. |
| `Provider` | Stores attributes that enable MTV to connect to and interact with the source and target providers. (VMware or Red Hat) |
| `Plan` | Contains a list of virtual machines with the same migration parameters and associated network and storage mappings. |
| `Migration` | Executes a migration plan. |
{: caption="MTV options and descriptions" caption-side="bottom"}


MTV can migrate workloads between mapped source and destination networks, network architecture and connectivity planning needs to be done before the migration workflow begins. For guidance on mapping vSphere network concepts to OpenShift equivalents, see [OVN networking in OpenShift for vSphere administrators](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-options-overview).
{: important}
