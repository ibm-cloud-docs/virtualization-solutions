---

copyright:
  years: 2025
lastupdated: "2026-02-06"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migration Design Overview
{: #virt-sol-vpc-migration-design-design-overview}

This guide focuses specifically on the design considerations for migrating your VMware workloads to IBM Cloud VPC virtual server instance. It assumes you have other design considerations documents covering compute, networking, storage, security, resiliency, and observability in your target VPC environment. Our focus here is on the migration journey itself, the decisions you'll need to make, the patterns you can follow, and the issues you'll want to avoid.
{: shortdesc}

## Understanding the Destination: VPC virtual server instance for VMware Administrators
{: #virt-sol-vpc-migration-design-design-overview-vpc}

Before diving into migration design, let's establish what's fundamentally different about VPC virtual server instance compared to your VMware environment.

### The Managed Hypervisor Model
{: #virt-sol-vpc-migration-design-design-overview-vpc-hypervisor}

In your VMware environment, you manage everything from the hypervisor up. You patch ESXi hosts, manage vCenter, configure vSAN polices, design and maintain NSX overlays, and handle capacity planning for compute, storage, and network resources.

With VPC virtual server instance, IBM manages the hypervisor layer, so you don't need to patch hosts. Instead, you work with:

- Instance profiles that define your virtual machine's CPU, memory, and network characteristics
- Storage profiles that define your disk performance characteristics
- Software-defined networking that's fundamentally Layer 3, not Layer 2
- APIs and Infrastructure-as-Code as the primary management interface

### From Layer 2 to Layer 3: The Networking Paradigm Shift
{: #virt-sol-vpc-migration-design-design-overview-network}

This is perhaps the most significant conceptual change. Your VMware environment uses Layer 2 networking extensively—VLANs, virtual switches, distributed port groups. You can move a virtual machine between hosts and maintain the same IP address because you're working at Layer 2.

VPC is fundamentally a Layer 3 software-defined network. While your virtual server instances may appear to have Layer 2 connectivity, under the hood they're connected via a highly scalable Layer 3 fabric. This has several implications:

1. You cannot stretch a subnet between VMware and VPC during migration. You'll need to migrate at least a subnet at a time.
1. Every IP address needs a Virtual Network Interface (VNI). In VMware terms, think of a VNI as a combination of a vNIC, its MAC address, its IP configuration, and its security group membership, all defined as a discrete object.
1. VPC reserves specific addresses in every subnet: the .0 address, .1 (gateway), .2 and .3 (reserved), and the broadcast address. Even though you can use your own private IP ranges, you may need to re-IP some virtual machines if they currently use these reserved addresses.
1. Shared VIPs are not straightforward. You can't simply assign the same virtual IP to multiple virtual server instances. For high availability scenarios, you'll typically use VPC's load balancers.

### Storage: Block Volumes and IOPS Tiers
{: #virt-sol-vpc-migration-design-design-overview-vpc-storage}

Coming from VMware, you're accustomed to either local VMDK files on vSAN or NFS storage accessed by your ESXi hosts. In VPC:

- All storage is network-attached block volumes. There are no local disks from the virtual machine's perspective.
- Storage performance is defined by IOPS tiers, not by the underlying array. You select a profile (like `5iops-tier` or `10iops-tier`) that guarantees performance per GB.
- Network bandwidth is shared between virtual machine networking and storage I/O in a 3:1 ratio by default. An instance profile that provides 32 Gbps of network bandwidth gives you ~24 Gbps for network traffic and ~8 Gbps for storage traffic. This matters when you're sizing your instances.
- Boot volumes are special. They have constraints (10-250 GB, must use `general-purpose` profile) and exist in a linked-clone relationship with their source image.
- No shared datastores - In VMware, you provision an NFS datastore and multiple virtual machines share that datastore's total IOPS capacity. If your datastore can deliver 10,000 IOPS, those IOPS are shared across all virtual machines on that datastore—potentially creating "noisy neighbor" issues where one virtual machine's heavy I/O impacts others. In VPC, each volume has its own dedicated performance allocation. A volume configured with 10iops-tier at 100GB gets 1,000 IOPS that are guaranteed to that volume alone, regardless of what other virtual server instances are doing. This eliminates the need for Storage DRS-style balancing and makes capacity planning more predictable. You size each volume individually rather than sizing shared capacity pools.

### Shared block storage is not available
{: #virt-sol-vpc-migration-design-design-overview-vpc-block}

VPC virtual server instance does not support shared block volumes
{: attention}

If you have Windows Failover Clusters using shared VMDK files, or Linux clusters with shared LVM, you'll need to redesign these. Your options, include:

- Use VPC file storage (NFS-based) for shared file access
- Deploy your own iSCSI target on a virtual server instance to expose shared block storage
- Redesign the application to use cloud-native HA patterns
- Use database replication instead of shared storage clustering

## Conclusion
{: #virt-sol-vpc-migration-design-design-overview-vpc-conclusion}

Migrating from VMware to IBM Cloud VPC virtual server instance is a significant undertaking that requires careful planning, the right method selection, and structured execution. The fundamental shift from managing hypervisors to consuming managed compute services changes not just your infrastructure but also your operational model.

Key Takeaways:

1. Understand the differences: VPC's Layer 3 networking, managed hypervisor model, and storage profiles represent a paradigm shift from VMware. Invest time in understanding these differences before you migrate your first virtual machine.
1. Choose the right method for your context:
   - Review the comparision table of the different migration methods available in [Migration methods for virtual servers](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-methods).
   - [Image Import (Template-Based Migration)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method1) for simple, single-disk virtual machines and template scenarios
   - [Copying direct volume (multi-disk method)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method2) for multi-disk virtual machines and precise control
   - [Live network transfer (Recommended for Scale)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method3) for scale, efficiency, and minimizing downtime
   - [VDDK Direct Extraction (vCenter Only)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method4) for vCenter automation-first approaches
1. Windows requires special handling: Whether sysprep or virt-v2v, budget time for driver injection and testing. Don't underestimate the RHEL/Ubuntu tooling gaps.
1. Network architecture is critical: Transit Gateway connectivity between your VMware environment and VPC is foundational. Test it early, test it thoroughly.
1. Start with a pilot**: Your first wave teaches you more than any amount of lab testing. Make it representative but low-risk.
1. Automate where it matters: For large migrations, invest in automation for the repetitive parts (volume provisioning, virtual server instance creation). Accept manual steps for the variable parts (application validation).
1. Plan for rollback: Things will go wrong. Having clear rollback criteria, procedures, and preserved source virtual machines gives you confidence to proceed.
1. Optimize post-migration: Don't stop at "it works." Right-size instances, leverage cloud-native services, and improve upon your VMware architecture.

The path from VMware to VPC virtual server instance is well-traveled, but every migration is unique. Use this guide as a framework, adapt it to your specific requirements, and document your own lessons learned for the next project. Your experience migrating infrastructure positions you well for the broader cloud transformation journey ahead.

## Additional Resources
{: #virt-sol-vpc-migration-design-design-overview-vpc-resources}

**IBM Cloud Documentation**:
- [VPC Getting Started](/docs/vpc?topic=vpc-getting-started)
- [Migrating Virtual Server Instances to VPC](/docs/vpc?topic=vpc-migrate-vsi-to-vpc)
- [Instance Profiles](/docs/vpc?topic=vpc-profiles)
- [Block Storage Profiles](/docs/vpc?topic=vpc-block-storage-profiles)
- [Transit Gateway](/docs/transit-gateway)
- [VPC Solution Tutorials](/docs?tab=solutions)

**Third-Party Tools and Resources**:
- [libguestfs and virt-v2v](https://libguestfs.org/){: external}
- [Red Hat Migration Toolkit for Virtualization](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.10){: external}
- [VMware VDDK Documentation](https://developer.broadcom.com/sdks/vmware-virtual-disk-development-kit-vddk/7.0){: external}
