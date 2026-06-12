---

copyright:
  years: 2025, 2026
lastupdated: "2026-06-12"

keywords: VSI, File Storage, Block Storage, Encryption, Migration, VMware to VPC migration, migrate VMware to IBM Cloud, VPC migration design, VMware workload migration, hypervisor differences VPC, VPC networking Layer 3, VPC storage migration, migration methods comparison, virt-v2v migration, VDDK migration VPC


subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migration design overview
{: #virt-sol-vpc-migration-design-design-overview}

Plan VMware to IBM Cloud VPC migrations by understanding hypervisor, networking, and storage differences, then choosing the right migration method for your workloads.
{: shortdesc}

## Understanding the destination: VPC virtual server instances for VMware administrators
{: #virt-sol-vpc-migration-design-design-overview-vpc}

Use the following sections to help you understand the differences between the IBM Cloud VPC environment compared to a VMware environment.

### The managed hypervisor approach
{: #virt-sol-vpc-migration-design-design-overview-vpc-hypervisor}

In your VMware environment, you manage most everything at the hypervisor level.

With VPC virtual servers, IBM Cloud manages the following actions at the hypervisor layer.

- Instance profiles that define virtual server CPU, memory, and network characteristics
- Storage profiles that define your disk performance characteristics
- Software-defined Layer 3 networking
- APIs and Infrastructure-as-Code as the primary management interface

### Networking differences
{: #virt-sol-vpc-migration-design-design-overview-network}

A VMware environment uses Layer 2 networking VLANs and virtual switches that are distributed port groups. You can move a virtual server between hosts and maintain the same IP address within a Layer 2 network.

IBM Cloud VPC is a Layer 3 software-defined network. While your VPC virtual server instances might appear to have Layer 2 connectivity, they're connected through a Layer 3 environment. Keep the following networking differences in mind when you plan your migration to VPC.

* You can't stretch a subnet between VMware and VPC during migration. You need to migrate at least one subnet at a time.
* Every IP address needs a Virtual Network Interface (VNI). In a VMware environment, think of a VNI as a combination of a vNIC, MAC address, IP configuration, and security group membership that are all defined as one object.
* VPC reserves a specific address in every subnet: the `.0` address, `.1` (gateway), `.2` and `.3` (reserved), and the broadcast address. Even though you can use your own private IP ranges, you might need to re-IP some virtual server if they currently use these reserved addresses.
* Shared virtual IPs are not straightforward. You can't assign the same virtual IP to multiple virtual server instances. For high availability scenarios, you typically use load balancers.

### Storage differences
{: #virt-sol-vpc-migration-design-design-overview-vpc-storage}

A VMware environment relies on either local VMDK files on vSAN or NFS storage that is accessed by ESXi hosts, while IBM Cloud VPC uses network-attached storage. The following items describe IBM Cloud VPC storage solutions for IBM Cloud VPC.

- All storage is network-attached block volumes. Virtual servers don't have local disks.
- Storage performance is defined by volume profiles. You can choose between traditional tiered profiles (3iops-tier, 5iops-tier, 10iops-tier) or the newer SDP (Defined Performance) profile that offers custom capacity (1-32,000 GB), IOPS (up to 64,000), and throughput (125-1024 MBps).
- Network bandwidth is shared between virtual server networking and storage input and output in a 3:1 ratio by default. Keep this ratio in mind when you size your instances.
- VPC doesn't use shared data stores. In a VMware environment, IOPS is shared across all virtual servers. This sharing creates "noisy neighbor" issues when one virtual server inputs and outputs affect other virtual servers. In the IBM Cloud VPC environment, each volume has its own dedicated performance allocation that helps prevent noisy neighbors.
- Boot volumes are automatically created during instance provisioning with capacity ranging from 10 GB to 32,000 GB `sdp` profile or 10-250 GB `general-purpose` profile. Boot volumes larger than 250 GB cannot be used to create custom images.

#### Choosing SDP over general-purpose block storage profiles
{: #virt-sol-vpc-migration-design-design-overview-vpc-storage-sdp}

When migrating to VPC, it's important to understand why sdp (Defined Performance) block storage is the recommended choice over general-purpose profiles:

**Modern VPC Architecture (Key Reason)**

- **SDP block storage** is native to IBM Cloud VPC and designed specifically for modern cloud-native architectures
- **General-purpose block storage** belongs to the classic infrastructure and is not integrated with VPC

**Predictable Performance with IOPS Profiles**

- **SDP**: Offers predefined performance tiers that are tied to volume size, providing predictable and consistent performance for your workloads
- **General-purpose**: Performance scales linearly with size, which may require larger volumes to achieve desired performance levels

**Better Security & Isolation**

SDP block storage provides enhanced security features through native VPC integration:

- **SDP** runs inside VPC and integrates seamlessly with:
  - Security groups for fine-grained access control
  - Private networking for isolated communication
  - IAM-based access for centralized identity and access management
- **General-purpose** relies on classic VLAN-based segmentation, which offers less granular security controls

For new migrations to VPC virtual servers, **SDP block storage is the recommended choice** due to its modern architecture, better security integration, and predictable performance characteristics.

#### Shared block storage not supported
{: #virt-sol-vpc-migration-design-design-overview-vpc-block}

VPC virtual server instance does not support shared block volumes
{: attention}

If you have Windows failover clusters that use shared VMDK files, or Linux clusters with shared LVM, then you need to redesign your storage solution. The following alternative options are available.

- Use VPC file storage (NFS-based) for shared file access
- Deploy your own iSCSI target on a virtual server to show shared block storage
- Redesign the application to use cloud-native high availability patterns
- Use database replication instead of shared storage clustering

## Tips for successful migrations
{: #virt-sol-vpc-migration-design-design-overview-vpc-next-steps}

Now that you understand the main differences between a VMware environment and an IBM Cloud environment, use the following information to learn more.

* Choose the right method that is best for your scenario.

   - Review the comparison table of the different [migration methods](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-methods).
   - [Image import (template-based migration)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method1) for simple, single-disk virtual servers and template scenarios.
   - [Copying direct volume (multi-disk method)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method2) for multi-disk virtual servers and more control over the migration process.
   - [Live network transfer](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method3) for scale, efficiency, and minimized downtime.
   - [VDDK Direct Extraction (vCenter only)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method4) for vCenter automated migrations.

* Windows requires special handling. Whether sysprep or virt-v2v, driver set up and testing takes time.
* Network architecture is important. Which means that Transit Gateway connectivity between your VMware environment and VPC is foundational. Test this connectivity thoroughly.
* Start with a simple test migration. Make it representative but low-risk.
* Automate where it matters. For large migrations, invest in automation for the repetitive parts (volume provisioning, virtual server instance creation). Accept manual steps for the variable parts (application validation).
* Plan for rollback. Things can go wrong. Have clear rollback criteria, procedures, and preserved source virtual servers.
* Optimize post-migration. Don't stop at "it works." Properly sized instances and adopting cloud-native services improves upon your previous architecture.

## Extra resources
{: #virt-sol-vpc-migration-design-design-overview-vpc-resources}

For extra information, see the following links.

- [VPC Getting started](/docs/vpc?topic=vpc-getting-started)
- [Migrating virtual server to VPC](/docs/vpc?topic=vpc-migrate-vsi-to-vpc)
- [Instance profiles](/docs/vpc?topic=vpc-profiles)
- [Block Storage Profiles](/docs/vpc?topic=vpc-block-storage-profiles)
- [Transit gateway](/docs/transit-gateway)
- [VPC solution tutorials](/docs?tab=solutions)

- [Libguestfs and virt-v2v](https://libguestfs.org/){: external}
- [Red Hat Migration Toolkit for virtualization](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.11){: external}
- [VMware VDDK documentation](https://developer.broadcom.com/sdks/vmware-virtual-disk-development-kit-vddk/7.0){: external}
