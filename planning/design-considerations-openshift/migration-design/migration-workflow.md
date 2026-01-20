---

copyright:
  years: 2025
lastupdated: "2026-01-09"

keywords:

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Red Hat OpenShift Migration Workflow
{: #virt-sol-openshift-migration-design-migration-workflow}

The Migration Toolkit for Virtualization (MTV) is provided as an Red Hat OpenShift Operator.

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

MTV does not do network migration. It must be planned and designed separately. MTV assumes that networks and network attachment definitions (NAD) exist when the `migration` happens and when you configure the Network and Storage maps.
{: important}
