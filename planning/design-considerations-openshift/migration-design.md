---

copyright:
  years: 2025
lastupdated: "2025-12-02"

keywords:

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Migration Workloads to Red Hat OpenShift Virtualiztaion 
{: #virt-sol-openshift-migration-design}

This is a short description that introduces the content in this topic.
{: shortdesc}

For migrations, customers can self-serve or they can use offerings and services from either IBM Consulting, Red Hat Consulting or other IBM Cloud’s migration services partners. To migrate virtual machines from an external provider such as VMware vSphere, Red Hat OpenStack Platform (RHOSP), Red Hat Virtualization, or another OpenShift Container Platform cluster, you can use the Migration Toolkit for Virtualization (MTV). Open Virtual Appliance (OVA) files created by VMware vSphere can also be migrated using the same tool. For self-serve customers supporting documentation and assets will be provided.


The key compute architecture elements are shown in the following diagram.

![Red Hat OpenShift Virtualization on IBM Cloud Migration](../../images/openshift/openshift-virtualization-high-level-migration.svg "Red Hat OpenShift Virtualization on IBM Cloud Migration"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Migration" caption-side="bottom"}


## Red Hat OpenShift Migration Toolkit for Virtualization (MTV)
{: #virt-sol-openshift-migration-design-mtv}

[OpenShift Virtualization]{: tag-red}

**The Migration Toolkit for Virtualization (MTV)** on OpenShift Container Platform helps migrate virtual machines from traditional hypervisors like VMware vSphere, Red Hat Virtualization, or OpenStack into OpenShift Virtualization, enabling consolidation of VMs and containers on a single Kubernetes-native platform.

MTV provides a web UI and API for discovery, planning, and execution of migrations, leveraging persistent volume cloning or streaming for disk data, while maintaining VM configuration and networking. It integrates with OpenShift Virtualization to run migrated VMs alongside containers, using Kubernetes-native storage and networking constructs.

For VMware migrations, the Migration Toolkit for Virtualization (MTV) integrates with vCenter to discover VMs and inventory data, map VMware resources (clusters, networks, datastores) to OpenShift equivalents, and automate bulk or selective migrations. It supports disk data transfer via warm or cold migration, preserves VM configuration (CPU, memory, NICs), and converts VMware constructs into Kubernetes-native resources for seamless execution in OpenShift Virtualization.

### Red Hat OpenShift Migration Types
{: #virt-sol-openshift-migration-design-migration-type}

[OpenShift Virtualization]{: tag-red}

Migration Toolkit for Virtualization (MTV) supports two types of migration, cold and warm. 

1. **Cold migration** is the default migration type where the source’s virtual machines are shutdown while the data is copied.
2. **Warm migration** copies most of the data during the precopy stage. Then the VMs are shut down and the remaining data is copied during the cutover stage.

Warm Migrations - **Precopy**
- The VMs are not shut down during the precopy stage.
- Create an initial snapshot of running VM disks.
- Copy first snapshot to target (full disk transfer, largest amount of data copied- Takes More Time)
- The VM disks are copied incrementally using changed block tracking (CBT) snapshots
- Copy deltas - Changed data ( copying only data which has changed since last snapshot- Takes Less Time):
    - Create a new snapshot
    - Copy the delta between previous snapshot and the new snapshot
    - Schedule the next snapshot (configurable, by default 1 hour after last snapshot finished)
    - A VM can support up to 28 CBT snapshots. If that limit is exceeded, a warm import retry limit reached error message is displayed. If the VM has preexisting CBT snapshots, it will reach this limit sooner.

Warm Migrations - **Cutover**
- The VMs are shut down during the cutover stage and the remaining data is migrated. Data stored in RAM is not migrated.
- Scheduled time to finalize warm migration
- You can start the cutover stage manually in the MTV console.
- Continue in the same way as cold migration
    - Guest conversion
    - Optionally starting target VM

Comparing the migration speeds of cold and warm migrations, you can observe single disk transfer and disk conversion are approximately the same for the warm and cold migrations. The benefit of warm migration is that the transfer of the snapshot happens in the background while the VM is powered on. The default snapshot time is taken every 60 minutes. If VMs change substantially, more data needs to be transferred than in cold migration when the VM is powered off. The cutover time, meaning the shutdown of the VM and last snapshot transfer, depends on how much the VM has changed since the last snapshot.

### Red Hat OpenShift Migration Workflow
{: #virt-sol-openshift-migration-design-migration-workflow}

[OpenShift Virtualization]{: tag-red}

The Migration Toolkit for Virtualization (MTV) is provided as an Red Hat OpenShift Operator. 

![Red Hat OpenShift Virtualization Migration Workflow](../../images/openshift/openshift-virtualization-migration-flow.svg "Red Hat OpenShift Virtualization Migration Workflow"){: caption="Red Hat OpenShift Virtualization Migration Workflow" caption-side="bottom"}


It creates and manages the following custom resources (CRs) and services.

* `NetworkMapping` Maps the networks of the source and target providers.
* `StorageMapping` Maps the storage of the source and target providers.
* `Provider`  Stores attributes that enable MTV to connect to and interact with the source and target providers. (VMware or Red Hat)
* `Plan` Contains a list of VMs with the same migration parameters and associated network and storage mappings.
* `Migration` Executes a migration plan.


It is important to understand, that MTV does not do network migration. It must be planned and designed separately. MTV assumes that networks and network attachment definitions (NAD) exist when the `migration` happens and when you configure the Network and Storage maps. 

### VMware Migration requirements
{: #virt-sol-openshift-migration-design-migration-vmware}

[OpenShift Virtualization]{: tag-red}

#### VMware environment requirements
{: #virt-sol-openshift-migration-design-migration-vmware-req}

- VMware vSphere must be version 6.5 or later.
- If you are migrating more than 10 VMs from an ESXi host in the same migration plan, you must increase the NFC service memory of the host.

#### Virtual machines
{: #virt-sol-openshift-migration-design-migration-vmware-vm}

- VMware Tools is installed.
- ISO/CDROM disks are unmounted.
- Each NIC must contain no more than one IPv4 and/or one IPv6 address.
- VM name contains only:
    - lowercase letters (a-z), numbers (0-9), or hyphens (-), up to a maximum of 253 characters. 
    - The first and last characters must be alphanumeric. 
    - The name must not contain uppercase letters, spaces, periods (.), or special characters.
- VM name does not duplicate the name of a VM in the OpenShift Virtualization environment.
- Operating system is certified and supported.

## Migration partners
{: #virt-sol-openshift-migration-design-partners}

IBM Cloud is working with several migration partners. Customers can do migration with self-serve and extensive documentation will be available to support customers on their efforts.

IBM Consulting, RedHat Consulting and IBM business partners can help, if customers require either resources or skills during their migration efforts.
