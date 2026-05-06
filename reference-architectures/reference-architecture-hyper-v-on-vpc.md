---

copyright:
  years: 2026, 2026
lastupdated: "2026-05-06"

keywords: Hyper-V, VPC, VSI, cloud infrastructure, Virtual Machines

subcollection: virtualization-solutions

authors:
- name: Xinpeng Liu, Jim Robbins

production: false

---

{{site.data.keyword.attribute-definition-list}}

> ⚠️ **Attention:** Currently the Hyper-V on IBM VPC is a BYOL solution, i.e., customers need to bring their own Windows Server 2025 Datacenter license for bare-metral server OS activation.

# Hyper-V on IBM VPC
{: #virt-sol-hyperv-on-vpc-architecture}

Hyper-V is Microsoft's enterprise-grade hypervisor technology built into Windows Server and Windows. It provides hardware virtualization capabilities that enable organizations to create, manage, and run virtual machines at scale.
As a type-1 hypervisor, Hyper-V can easily run on bare-metal servers hosted by {{site.data.keyword.vsi_is_full}} to deliver near-native performance and robust isolation for virtualized workloads leveraging IBM VPC's infrastructure capabilities.


## Hyper-V on IBM VPC architecture overview
{: #virt-sol-hyperv-on-vpc-architecture-diagram}

The following diagram shows the high-level reference architecture to set up a typical Hyper-V cluster on IBM VPC.

![Hyper-V on IBM VPC Reference Architecture](../images/hyper-v/hyper-v-on-vpc-arch.svg "Hyper-V on IBM VPC Reference Architecture"){: caption="Hyper-V on IBM VPC Reference Architecture" caption-side="bottom"}

### Components
{: #virt-sol-hyperv-on-vpc-components}

The following table outlines the products or services that are used in the architecture for each component.

| Component | Architecture components | How the component is used |
| -------------- | -------------- | -------------- |
| Jump Server | A Virtual Server Instance on VPC | Provide a virtual machine that is accessible from public Internet to access other VSIs or hosts within the architecture that are only accessible with the private network. Typically, users could use Microsoft Remote Desktop client to remote log on any target machines for installation or management purpose. |
| Active Directory | 2 Virtual Server Instances on VPC | Provide a virtual machine acting as Microsoft directory service to help Bare Metal Hosts identify domain names within the Hyrer-V cluster on top of VPC. Because IBM Cloud VPC is primarily a Layer 3 (network layer) software-defined network, Subnets do not act as a shared Layer 2 broadcast domain, so AD servers are needed here. The 2nd virtual machine is used as the backup of the first AD server to provide redundancy based fail-over. |
| Hyper-V Host Node | Bare metal servers on VPC | Provide bare metal servers with Windows server installed as native OS, which have built-in Hyper-V infrastructure capabilities. The network of these hosts usually only have isolated subnets detached from external Internet, and in most of the cases there are at least one subnet serving for management purpose, and one subnet used for actual workload communication. All these hosts' domain names are registered to above AD servers. |
| Hyper-V Cluster | Fail-over cluster on Hyper-V host nodes | Installed as a fail-over cluster across the Hyper-V host nodes above. A Cluster Shared Volumes may be added to the cluster to provide distributed cluster-wide storage and availability management. |

{: caption="Hyper-V on VPC components" caption-side="bottom"}

Within above reference architecture, there 3 types of virtual network interface(VNI) used:

- Undifferentiated VNI attached to a Virtual Server Instance
- VNI attached to a bare-metal server as a physical interface - PCI VNI
- VNI attached to a bare metal server as either a floating or a virtual interface - VLAN VNI

## Next steps
{: #virt-sol-hyperv-on-vpc-next-steps}

Now that you understand the Hyper-V on IBM VPC reference architecture, explore the following resources:

- **Tutorial to deploy a Hyper-V cluster within an IBM VPC environment**: Follow the [step-by-step deployment tutorial](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-hyperv-on-vpc-deployment-tutorial-overview) to set up your first Hyper-V cluster with IBM VPC.
