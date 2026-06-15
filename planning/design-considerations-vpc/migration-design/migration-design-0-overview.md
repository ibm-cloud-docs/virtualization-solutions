---

copyright:
  years: 2026
lastupdated: "2026-06-15"

keywords: VMware migration, IBM Cloud VPC, VPC virtual server instances, migration design, block storage, network migration, workload migration, VMware to VPC, encryption, VSI, File Storage, Block Storage, Encryption, Migration, VMware to VPC migration, migrate VMware to IBM Cloud, VPC migration design, VMware workload migration, hypervisor differences VPC, VPC networking Layer 3, VPC storage migration, migration methods comparison, virt-v2v migration, VDDK migration VPC

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migration design overview
{: #virt-sol-vpc-migration-design-design-overview}

Review the main design considerations for migrating VMware workloads to {{site.data.keyword.vpc_short}}, including compute, networking, storage, and migration method planning.
{: shortdesc}

## Understanding the destination: VPC virtual server instances for VMware administrators
{: #virt-sol-vpc-migration-design-design-overview-vpc}

Use the following sections to understand the differences between the {{site.data.keyword.vpc_short}} environment and a VMware environment.

### The managed hypervisor approach
{: #virt-sol-vpc-migration-design-design-overview-vpc-hypervisor}

In your VMware environment, you manage most resources at the hypervisor level.

With VPC virtual servers, {{site.data.keyword.cloud_notm}} manages the following functions at the hypervisor layer:

- Instance profiles that define virtual server CPU, memory, and network characteristics
- Storage profiles that define disk performance characteristics
- Software-defined Layer 3 networking
- APIs and Infrastructure as Code that serve as the primary management interfaces

### Networking differences
{: #virt-sol-vpc-migration-design-design-overview-network}

A VMware environment uses Layer 2 VLANs and distributed virtual switch port groups. You can move a virtual server between hosts while maintaining the same IP address within a Layer 2 network.

{{site.data.keyword.vpc_short}} is a Layer 3 software-defined network. While your VPC virtual server instances might appear to have Layer 2 connectivity, they are connected through a Layer 3 environment. Keep the following networking differences in mind when you plan your migration to {{site.data.keyword.vpc_short}}:

- You cannot stretch a subnet between VMware and {{site.data.keyword.vpc_short}} during migration. You need to migrate at least one subnet at a time.
- Every IP address needs a Virtual Network Interface (VNI). In VMware, a VNI combines a virtual network interface card (vNIC), media access control (MAC) address, IP configuration, and security group membership into one object.
- VPC reserves-specific addresses in every subnet: the `.0` address, `.1` (gateway), `.2` and `.3` (reserved), and the broadcast address. Even though you can use your own private IP ranges, you might need to assign new IP addresses to some virtual servers if they currently use these reserved addresses.
- Shared virtual IPs are not straightforward. You cannot assign the same virtual IP to multiple virtual server instances. For high availability scenarios, you typically use load balancers.

### Storage differences
{: #virt-sol-vpc-migration-design-design-overview-vpc-storage}

A VMware environment uses local virtual machine disk (VMDK) files on VMware vSphere Virtual SAN (vSAN) or Network File System (NFS) storage that ESXi hosts access, while {{site.data.keyword.vpc_short}} uses network-attached storage. The following points describe {{site.data.keyword.cloud_notm}} VPC storage solutions for {{site.data.keyword.vpc_short}}:

- All storage uses network-attached block volumes. Virtual servers do not have local disks.
- Storage performance is defined by volume profiles. You can choose from traditional tiered profiles (`3iops-tier`, `5iops-tier`, and `10iops-tier`) or the newer SDP (defined performance) profile, which offers custom capacity (1 - 32,000 GB), input/output operations per second (IOPS) (up to 64,000), and throughput (125 - 1,024 MBps).
- Network bandwidth is shared between virtual server networking and storage input/output (I/O) in a 3:1 ratio by default. Keep this ratio in mind when you size your instances.
- {{site.data.keyword.vpc_short}} does not use shared data stores. In a VMware environment, IOPS is shared across all virtual servers. This model can create "noisy neighbor" issues when one virtual server's workload affects others. In the {{site.data.keyword.vpc_short}} environment, each volume has its own dedicated performance allocation, which helps prevent noisy neighbor issues.
- Boot volumes are created automatically during instance provisioning. Capacity ranges from 10 GB to 32,000 GB for the `sdp` profile and from 10 GB to 250 GB for the `general-purpose` profile. Boot volumes larger than 250 GB cannot be used to create custom images.

#### Choosing SDP over general-purpose block storage profiles
{: #virt-sol-vpc-migration-design-design-overview-vpc-storage-sdp}

When you migrate to {{site.data.keyword.vpc_short}}, SDP (defined performance) block storage is recommended over general-purpose profiles for the following reasons:

##### Modern VPC architecture

- SDP block storage is native to {{site.data.keyword.vpc_short}} and is designed for modern cloud-native architectures.
- General-purpose block storage belongs to the classic infrastructure and is not integrated with {{site.data.keyword.vpc_short}}.

##### Predictable performance with IOPS profiles

- SDP: Uses predefined performance tiers that are tied to volume size, which provide predictable and consistent performance for your workloads.
- General-purpose: Uses performance that scales linearly with size, which might require larger volumes to achieve the required performance levels.

##### Better security and isolation

SDP block storage provides enhanced security features through native {{site.data.keyword.vpc_short}} integration:

- SDP integrates with security groups for fine-grained access control.
- SDP integrates with private networking for isolated communication.
- SDP integrates with Identity and Access Management (IAM)-based access for centralized identity and access management.
- General-purpose relies on classic VLAN-based segmentation, which offers less granular security controls.

For new migrations to {{site.data.keyword.vpc_short}} virtual servers, SDP block storage is recommended because it provides modern architecture, security integration, and predictable performance characteristics.

#### Shared block storage not supported
{: #virt-sol-vpc-migration-design-design-overview-vpc-block}

VPC virtual server instances do not support shared block volumes.
{: attention}

If you have Windows failover clusters that use shared VMDK files or Linux clusters with shared logical volume manager (LVM), you need to redesign your storage solution. The following alternatives are available:

- Use VPC file storage (NFS-based) for shared file access
- Deploy your own Internet Small Computer Systems Interface (iSCSI) target on a virtual server to provide shared block storage
- Redesign the application to use cloud-native high availability patterns
- Use database replication instead of shared storage clustering

## Tips for successful migrations
{: #virt-sol-vpc-migration-design-design-overview-vpc-next-steps}

Now that you understand the main differences between the VMware environment and the {{site.data.keyword.cloud_notm}} environment, use the following information to learn more.

Choose the migration method that best fits your scenario.

- Review the comparison table of [migration methods](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-methods).
- [Image import (template-based migration)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method1) for simple, single-disk virtual servers and template scenarios.
- [Direct volume copy (multi-disk method)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method2) for multi-disk virtual servers and more control over the migration process.
- [Live network transfer](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method3) for scale, efficiency, and minimized downtime.
- [VMware Virtual Disk Development Kit (VDDK) direct extraction (vCenter only)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method4) for automated migrations from vCenter.

- Account for Windows-specific requirements. Whether you use sysprep or `virt-v2v`, driver setup and testing take time.
- Validate your network architecture. {{site.data.keyword.tg_full}} connectivity between your VMware environment and VPC is foundational. Test this connectivity thoroughly.
- Start with a simple test migration. Make it representative but low risk.
- Automate where it matters. For large migrations, invest in automation for repetitive tasks such as volume provisioning and virtual server instance creation. Use manual steps for variable tasks such as application validation.
- Plan for rollback. Things can go wrong. Have clear rollback criteria, procedures, and preserved source virtual servers.
- Optimize after migration. Do not stop at "it works." Properly sized instances and cloud-native services can improve your architecture.

## Additional resources
{: #virt-sol-vpc-migration-design-design-overview-vpc-resources}

Refer to the following resources:

- [VPC getting started](/docs/vpc?topic=vpc-getting-started)
- [Migrating virtual server instances to VPC](/docs/vpc?topic=vpc-migrate-vsi-to-vpc)
- [Instance profiles](/docs/vpc?topic=vpc-profiles)
- [Block storage profiles](/docs/vpc?topic=vpc-block-storage-profiles)
- [Transit gateway](/docs/transit-gateway)
- [VPC solution tutorials](/docs?tab=solutions)
- [Libguestfs and virt-v2v](https://libguestfs.org/){: external}
- [{{site.data.keyword.redhat_full}} Migration Toolkit for Virtualization](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.11){: external}
- [VMware VDDK documentation](https://developer.broadcom.com/sdks/vmware-virtual-disk-development-kit-vddk/7.0){: external}
