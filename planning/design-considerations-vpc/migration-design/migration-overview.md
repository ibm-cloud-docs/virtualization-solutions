---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# VMware to VPC VSI migration overview
{: #virt-sol-vpc-migration-design-migration}

There are the following options for migration:

- **Service Provider** - Using a Service Provider such as WanClouds or PrimaryIO to manage the migration for you. This guide does not focus on this option.
- **RackWare RMM** - Using the RackWare RMM tooling to migrate your VMs. This documentation includes a guide and a tutorial.
- **DIY migration** - Migration methods that don't use a service provider or RackWare RMM, but use techniques that you can adopt and automate to suit your needs.

The key compute architecture elements are shown in the following diagram.

![VPC VSI Virtualization on IBM Cloud Migration](../../../images/vpc-vsi/vpc-vsi-high-level-migration.svg "VPC VSI Virtualization on IBM Cloud Migration"){: caption="VPC VSI Virtualization on IBM Cloud Migration" caption-side="bottom"}


This guide focuses on **General Principles**, **DIY migration**, and **RackWare RMM**, see:

## General Principles
{: #virt-sol-vpc-migration-design-migration-principles}

* [Pre-Migration Design Decisions](migration-design-1-premigration.md)
* [Wave Planning and Execution Design](migration-design-2-wave.md)
* [Post-Migration Validation and Optimization](migration-design-3-postmigration.md)
* [Risk Mitigation and Rollback Strategies](migration-design-4-risk.md)
* [Governance, Documentation, and Lessons Learned](migration-design-5-governance.md)

## DIY migration
{: #virt-sol-vpc-migration-design-migration-diy}

* [Method 1: Image Import (Template-Based Migration)](migration-method-1.md)
* [Method 2: Direct Volume Copy (Multi-Disk Migration)](migration-method-2.md)
* [Method 3: Live Network Transfer (Recommended for Scale)](migration-method-3.md)
* [Method 4: VDDK Direct Extraction (vCenter Only)](migration-method-4.md)
* [Linux Migration Considerations](migration-considerations-linux.md)
* [Windows Migration Considerations](migration-considerations-windows.md)

## RackWare RMM
{: #virt-sol-vpc-migration-design-migration-rackware}

* [Migrating from IBM Cloud VMware VCF-Automated to VPC VSI with RackWare RMM Technical Guide](rackware-vcf-classic-2-vpc-vsi-guide.md)
* [Migrating from IBM Cloud VMware VCF-Automated to VPC VSI with RackWare RMM Tutorial](rackware-vcf-classic-2-vpc-vsi-tutorial.md)


Here are short descriptions of each migration method for an overview/summary:

## DIY Migration Methods Overview
{: #virt-sol-vpc-migration-design-migration-diy}

### Method 1: Image Import
{: #virt-sol-vpc-migration-design-migration-diy-method1}

Export your VM from VMware as a VMDK, convert it to QCOW2 format, upload to IBM Cloud Object Storage, and create a custom VPC image. Boot new VSIs from this image, similar to deploying from a VMware template. Best suited for single-disk VMs and scenarios where you want to reuse a base image across multiple deployments. Simple and well-documented, but creates proliferation of custom images and only handles one disk per VM. See [Method 1: Image Import (Template-Based Migration)](migration-method-1.md).

### Method 2: Direct Volume Copy
{: #virt-sol-vpc-migration-design-migration-diy-method2}

t specifications you need, then directly write your VM's disk contents to them using a worker VSI. Export VMDKs from VMware, transfer them to the worker VSI, convert with `qemu-img`, and optionally transform with `virt-v2v` for driver injection. Attach the populated volumes to your new VSI. Handles multi-disk VMs, avoids image proliferation, and provides precise control over the migration process at the cost of increased orchestration complexity. See [Method 2: Direct Volume Copy (Multi-Disk Migration)](migration-method-2.md).

### Method 3: Live Network Transfer
{: #virt-sol-vpc-migration-design-migration-diy-method3}

Boot your source VM from a live ISO (like Ubuntu installer), establish network connectivity to a worker VSI in VPC via Transit Gateway, and stream disk contents directly over the network using tools like `dd`, `gzip`, and `netcat`. Eliminates the export step entirely, making it highly efficient for large-scale migrations. Supports parallel migrations and provides maximum flexibility, though it requires upfront investment in Transit Gateway setup and live ISO preparation. See [Method 3: Live Network Transfer (Recommended for Scale)](migration-method-3.md).

### Method 4: VDDK Direct Extraction
{: #virt-sol-vpc-migration-design-migration-diy-method3}

Use Red Hat's `virt-v2v` tool with VMware's VDDK library to connect directly to vCenter, extract VM disks, perform format conversion and driver injection in a single operation, and write directly to VPC volumes. Highly automated and ideal for vCenter environments, but requires complex tool setup (dealing with RHEL/Ubuntu capability gaps), only works with vCenter (not VCFaaS), and has significant prerequisites including VDDK licensing and vCenter API access. See [Method 4: VDDK Direct Extraction (vCenter Only)](migration-method-4.md).

### Migration Considerations
{: #virt-sol-vpc-migration-design-migration-diy-considerations}

See the following for specific considerations for Windows and Linux VMs that apply to the methods described above:

* [Linux Migration Considerations](migration-considerations-linux.md)
* [Windows Migration Considerations](migration-considerations-windows.md)

## RackWare RMM migration overview
{: #virt-sol-vpc-migration-design-migration-rmm}

RackWare Migration Manager (RMM) is a commercial migration platform available directly from the IBM Cloud Catalog that provides an automated, enterprise-grade approach to migrating VMware workloads to IBM Cloud VPC VSI. Unlike the manual methods described earlier, RackWare offers a migration experience with a graphical interface, and centralized orchestration.

### How RackWare RMM Works
{: #virt-sol-vpc-migration-design-migration-rmm-how}

RackWare RMM operates using a lightweight agentless architecture. You deploy the RackWare Management Server as a VSI in your target VPC environment, which serves as the orchestration hub for all migration activities. The platform discovers your source virtual machines, and provides a web-based interface for selecting and configuring migrations.

The platform performs block-level replication of your VM disks, transferring data directly to volumes in VPC. During this process, RackWare automatically handles format conversions (VMDK to raw), driver injection (installing VirtIO drivers for Windows and Linux), and OS-level modifications needed for the target cloud environment.

RackWare RMM can auto-provision the target VSIs and their boot and data volumes based on the source VMs. RackWare supports delta sync, allowing you to minimize cutover windows by performing an initial full sync while the source VM remains running, followed by incremental syncs of changed blocks, and finally a brief cutover window for the final sync and switchover.

RackWare RMM, supports the use of a bridge server, that enables NAT for the source VMs, enabling the retention of IP address ranges in both source and target.

### Availability in IBM Cloud Catalog
{: #virt-sol-vpc-migration-design-migration-rmm-availability}

RackWare RMM is available as a BYOL (Bring Your Own License) offering in the IBM Cloud Catalog. You provision the RackWare Management Server as a VSI in your VPC, and licensing is handled directly with RackWare based on the number of workloads you plan to migrate. IBM has a partnership with RackWare, and the solution is validated and supported for VMware to VPC migrations. You can find RackWare in the catalog under Migration tools or by searching for "RackWare".

The deployment process involves provisioning the RackWare appliance in your VPC, configuring network connectivity (typically via Transit Gateway) between your VMware environment and VPC, and then using the RackWare web interface to configure and execute migrations. RackWare provides documentation and support specific to IBM Cloud VPC target environments.

### Suitability for VMware to VPC Migrations
{: #virt-sol-vpc-migration-design-migration-rmm-suitability}

RackWare RMM is particularly well-suited for organizations that:

**Need to migrate large numbers of VMs (50+)**: The centralized orchestration, batching capabilities, and automation significantly reduce per-VM effort compared to manual methods. What might take 2-3 hours per VM manually can be reduced to 30-60 minutes of hands-on time with RackWare handling the bulk of the work automatically.

**Require minimal downtime**: RackWare's migration with delta sync allows you to perform most of the data transfer while applications remain running, reducing cutover windows from hours to minutes. This is especially valuable for production workloads with strict availability requirements.

**Have limited cloud expertise**: RackWare abstracts many of the complexities of driver injection, OS preparation, and cloud-specific configurations. Teams that are strong in VMware but new to VPC can execute migrations successfully with less of a learning curve.

**Need consistent, repeatable processes**: For enterprises with compliance requirements or standardized change management, RackWare provides workflow automation, audit logging, and repeatability that can be difficult to achieve with manual methods.

**Value commercial support**: Unlike the open-source tools (libguestfs, virt-v2v), RackWare provides enterprise support, regular updates, and validated configurations for IBM Cloud VPC.

However, RackWare may not be the best fit for:

**Small migrations (<10 VMs)**: The licensing cost and setup time may not justify the automation benefits for very small migrations where manual methods are sufficient.

**Highly customized or non-standard VMs**: While RackWare handles most OS configurations, VMs with very specialized configurations, custom kernels, or unusual storage layouts may require manual intervention or may be better suited to manual migration methods where you have complete control.

**Budget-constrained projects**: RackWare is a commercial product with licensing costs. If budget is tight and you have the technical expertise, the manual methods described earlier can achieve similar results at lower cost (though with higher labor investment).

**Organizations wanting to learn VPC deeply**: Using RackWare abstracts away many VPC concepts that your team might benefit from learning hands-on. For teams building long-term VPC expertise, starting with manual migrations (even for a pilot wave) can provide valuable learning experiences before automating with RackWare for scale.

### Integration with Manual Methods
{: #virt-sol-vpc-migration-design-migration-rmm-integration}

RackWare and manual methods are not mutually exclusive. A common pattern is to:

1. Execute a pilot wave manually (Methods 2 or 3) to learn VPC concepts and validate your target architecture
2. Document issues, timing, and lessons learned
3. Deploy RackWare for subsequent waves, leveraging your pilot experience to configure RackWare optimally
4. Reserve manual methods for edge cases or problematic VMs that RackWare struggles with

This hybrid approach balances learning, cost, and automation benefits across your migration project.
