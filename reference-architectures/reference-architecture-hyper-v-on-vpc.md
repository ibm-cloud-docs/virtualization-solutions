---

copyright:
  years: 2026
lastupdated: "2026-07-21"

keywords: Hyper-V on VPC, VPC bare metal servers, Hyper-V cluster, failover cluster VPC, Cluster Shared Volumes, Active Directory VPC, Windows Server VPC, type-1 hypervisor, Hyper-V reference architecture, BYOL virtualization

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Deploying Hyper-V on IBM Cloud VPC: Reference architecture
{: #virt-sol-hyperv-on-vpc-architecture}

Explore the reference architecture for deploying Microsoft Hyper-V on IBM Cloud Virtual Private Cloud (VPC) bare metal servers as a Bring Your Own License (BYOL) virtualization solution.
{: shortdesc}

Currently, Hyper-V on {{site.data.keyword.vpc_short}} is a BYOL solution. You must bring your own Windows&reg; Server 2025 Datacenter license to activate the bare metal server operating system (OS).
{: attention}

Hyper-V is Microsoft's enterprise-grade hypervisor technology that is built into Windows Server and Windows. It provides hardware virtualization capabilities that enable organizations to create, manage, and run virtual machines at scale. As a type-1 hypervisor, Hyper-V can run on bare metal servers hosted on {{site.data.keyword.vpc_short}} to deliver near-native performance and isolation for virtualized workloads by using its infrastructure capabilities.

## Hyper-V on IBM Cloud VPC architecture overview
{: #virt-sol-hyperv-on-vpc-architecture-diagram}

The following diagram shows the high-level reference architecture to set up a typical Hyper-V cluster on {{site.data.keyword.vpc_short}}.

![Hyper-V on IBM Cloud VPC Reference Architecture](../images/hyper-v/hyper-v-on-vpc-arch.svg "Hyper-V on IBM Cloud VPC Reference Architecture"){: caption="Hyper-V on IBM Cloud VPC Reference Architecture" caption-side="bottom"}

### Components
{: #virt-sol-hyperv-on-vpc-components}

The following table outlines the products or services that are used in the architecture for each component.

| Component | Architecture components | How the component is used |
| -------------- | -------------- | -------------- |
| Jump server | A virtual server instance on {{site.data.keyword.vpc_short}} | Provides a virtual machine that is accessible from the public internet to access other virtual servers or hosts within the architecture that are accessible only through the private network. Typically, users can use Microsoft Remote Desktop client to remotely access target systems for installation or management purposes. |
| Active Directory (AD) | 2 virtual server instances on {{site.data.keyword.vpc_short}} | Provides virtual machines that act as Microsoft directory services to help bare metal hosts identify domain names within the Hyper-V cluster on {{site.data.keyword.vpc_short}}. Because {{site.data.keyword.vpc_short}} is primarily a Layer 3 (network layer) software-defined network, subnets do not act as a shared Layer 2 broadcast domain, so AD servers are needed. The second virtual machine is used as a backup of the first AD server to provide redundancy-based failover. |
| Hyper-V host node | Bare metal servers on {{site.data.keyword.vpc_short}} | Provides bare metal servers with Windows Server installed as the native OS, which has built-in Hyper-V infrastructure capabilities. The network of these hosts usually has only isolated subnets that are detached from the external internet. In most cases, there is at least one subnet for management purposes and one subnet for actual workload communication. All host domain names are registered with the AD servers. |
| Hyper-V cluster | Failover cluster on Hyper-V host nodes | Installed as a failover cluster across the Hyper-V host nodes. Cluster Shared Volumes can be added to the cluster to provide distributed cluster-wide storage and availability management. |
{: caption="Hyper-V on VPC components" caption-side="bottom"}

Within the reference architecture, the following 3 types of virtual network interface (VNI) are used:

- Undifferentiated VNI attached to a virtual server instance
- VNI attached to a bare metal server as a physical interface (PCI VNI)
- VNI attached to a bare metal server as either a floating or a virtual interface (VLAN VNI)

For high-availability networking in Hyper-V on VPC, networking interfaces and uplinks must be configured without using any Hyper-V vSwitch Switch Embedded Teaming (SET) bonding or any other network interface controller (NIC) bonding or link aggregation group (LAG) high availability strategy. From the VPC perspective, the SmartNICs in the host have dual physical ports and provide built-in high availability. The SmartNIC handles all of the redundancy and fail-over automatically and does not expose it to the host OS. As a result, the Windows platform is unaware and Hyper-V remain unaware of the failover process. This configuration maintains high availability but does not support configuring SET team switch setups or other bonding configurations that are provided by the Windows OS and Hyper-V infrastructure. As a Hyper-V on VPC user, you can configure up to 8 PCI interfaces for a VPC bare metal server, and rely on each PCI interface for full redundancy and high availability to the switch fabric without requiring any configuration in Windows.
{: note}

The Hyper-V on {{site.data.keyword.vpc_short}} feature currently supports only Storage Spaces Direct (S2D) technology. Block storage and Network File System (NFS)-based file storage that is shared by customers are not recognized or supported by bare metal servers or by Hyper-V that is installed on those servers.
{: note}

## Next steps
{: #virt-sol-hyperv-on-vpc-next-steps}

Now that you understand the Hyper-V on {{site.data.keyword.vpc_short}} reference architecture, explore the following resource:

Tutorial to deploy a Hyper-V cluster within the {{site.data.keyword.vpc_short}} environment: Follow the [step-by-step deployment tutorial](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-hyperv-on-vpc-tutorial) to set up your first Hyper-V cluster with {{site.data.keyword.vpc_short}}.
