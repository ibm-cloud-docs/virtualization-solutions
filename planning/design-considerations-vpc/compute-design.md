---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-09"

keywords: VSI, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Compute Design for VPC virtual servers
{: #virt-sol-vpc-compute-design-overview}

IBM Cloud VPC provides a comprehensive portfolio of compute options designed to support diverse workloads, from traditional applications to modern cloud-native solutions. These offerings deliver flexibility, scalability, and enterprise-grade security, enabling organizations to deploy workloads across virtualized, containerized, and bare metal environments.

The following are the key IBM Cloud VPC compute options:

- Virtual Servers for VPC
- Bare Metal Servers for VPC

IBM Cloud VPC compute solutions empower businesses to choose the right infrastructure for their needs—whether optimizing cost, achieving high performance, or enabling hybrid and multicloud strategies.

The key compute architecture elements are shown in the following diagram.

![IBM Cloud VPC VSI Compute](../../images/vpc-vsi/vpc-vsi-high-level-compute.svg "IBM Cloud VPC VSI Compute"){: caption="IBM Cloud VPC VSI Compute" caption-side="bottom"}

## IBM Cloud Virtual Servers for VPC
{: #virt-sol-vpc-compute-design-virtual-servers}

IBM Cloud Virtual Servers for VPC provide secure, isolated virtual machines deployed within a Virtual Private Cloud environment. These instances deliver enterprise-grade compute for production workloads, and development and test environments requiring flexible resource allocation and comprehensive infrastructure control.

The following tables lists the key features for IBM Cloud Virtual Servers for VPC.

| Feature | Description |
| -------------- | -------------- |
| Customizable profiles | Balanced, compute-optimized, memory-optimized, GPU, and very high memory configurations |
| Flexible tenancy | Shared tenancy infrastructure with optional dedicated host placement for compliance requirements |
| Advanced networking | Integration with VPC Security Groups, Network ACLs, Load Balancers, and VPN connectivity |
| Persistent storage | IBM Cloud Block Storage and File Storage for VPC with configurable IOPS and encryption |
| Operating system flexibility | IBM-provided stock images or bring-your-own custom images |
| Scalability | Vertical scaling through profile changes and horizontal scaling through instance groups with auto-scaling |
{: caption="IBM Cloud Virtual Servers for VPC key features" caption-side="bottom"}

For more information on virtual server instance profiles, see [IBM Cloud Docs - VPC Instance Profiles](/docs/vpc?topic=vpc-profiles).

## Next steps
{: #virt-sol-vpc-compute-design-next-steps}

Now that you understand the compute design options for VPC virtual servers, explore these related topics:

- **Networking**: Review [networking design considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-design) for VPC connectivity and security
- **Storage**: Explore [storage design options](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-storage-design-overview) for persistent data
- **Security**: Learn about [security design patterns](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-security-design-overview) for VPC workloads
- **Migration**: Review [migration strategies](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-migration) from VMware to VPC
