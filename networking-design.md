---
copyright:
  years: 2025
lastupdated: "2025-11-25"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Network Design Components
{: #virt-sol-network-design-overview}

Networking is the backbone of any cloud architecture and plays a critical role in enabling secure, reliable, and high-performance connectivity for workloads. In IBM Cloud, networking services provide the foundation for virtualization, container orchestration, and hybrid cloud integration, ensuring that applications and data can move seamlessly across environments.

For workload migration, robust networking capabilities are essential to maintain application availability, security, and compliance during transition. IBM Cloud offers advanced networking features such as Virtual Private Cloud (VPC), subnets, security groups, load balancing, and Direct Link for private connectivity to on-premises environments. These services enable organizations to design architectures that support scalable deployments, multi-zone resilience, and secure interconnectivity across hybrid and multicloud landscapes.

By leveraging IBM Cloud networking, businesses can confidently migrate workloads while maintaining performance and governance, paving the way for modernization and innovation.

The key network architecture elements are shown in the following diagram.

![Red Hat Virtualization on IBM Cloud Network](images/openshift-virtualization-high-level-network.svg "Red Hat Virtualization on IBM Cloud Network"){: caption="Red Hat Virtualization on IBM Cloud Network" caption-side="bottom"}

## IBM Cloud VPC
{: #virt-sol-network-design-vpc-networking}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

IBM Cloud VPC is a secure, isolated, and highly configurable networking environment that enables organizations to deploy and manage cloud resources with fine-grained control. It provides the foundation for modern workloads, including virtual servers, containers, and bare metal deployments, while ensuring network segmentation, security, and scalability.

### Default private networking with subnets
{: #virt-sol-network-design-vpc-networking-subnets}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

VPCs span a region and are divided into subnets spanning individual zones within that region. These subnets use a range of private IP addresses, with the option to bring your own public IP range. Subnets within a VPC are private by default and are able to talk to each other without setting up any routes. As a result, all resources within a VPC are able to communicate to one another. VSIs are attached to one or more subnets. See the architecture diagram in [About networking](https://cloud.ibm.com/docs/vpc?topic=vpc-about-networking-for-vpc) for a visual representation of the VPC networking concepts. For additional information and design considerations, see [Setting IP ranges](https://cloud.ibm.com/docs/vpc?group=ip-ranges).

### External connectivity
{: #virt-sol-network-design-vpc-networking-external-connectivity}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

External connectivity can be achieved with public gateways, floating IPs, and VPNs. Public gateways span an entire subnet and all VSIs attached to it and support initiating connections to the internet. Floating IPs span a single VSI and support initiating connections to and receiving connections from the internet. The IBM Cloud VPN for VPC service supports secure connectivity from a VPC to another private network. See [About site-to-site VPN gateways](https://cloud.ibm.com/docs/vpc?topic=vpc-using-vpn) for additional details on working with VPNs.

### Security
{: #virt-sol-network-design-vpc-networking-security}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

Networking security can be controlled with security groups and access control lists. Access control lists manage traffic at the subnet level, whereas security groups manage traffic at the VSI level. Access control lists therefore can gate external connectivity established via a public gateway on a subnet, while security groups can be used to gate external connectivity established via a floating IP on a VSI. See [Security in your VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-security-in-your-vpc) for additional details.

### Interconnectivity
{: #virt-sol-network-design-vpc-networking-interconnectivity}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

Interconnecting a VPC with an on-premises network can be done with IBM Cloud Direct Link or VPNaaS, while interconnecting VPCs to each other and various other resources can be done with IBM Cloud Transit Gateway. Connecting a VPC with IBM Cloud classic infrastructure can also be done with IBM Cloud Transit Gateway. [Getting started with IBM CLoud Transit Gateway](https://cloud.ibm.com/docs/transit-gateway?topic=transit-gateway-getting-started)

To determine the appropriate IBM Cloud Direct Link offering for your workload and learn more about interconnectivity with IBM Cloud VPC, see [Interconnecting your VPC using IBM Cloud offerings](https://cloud.ibm.com/docs/vpc?topic=vpc-interconnectivity).

IBM Cloud has two VPN services. VPN for VPC offers site-to-site gateways, which connect your on-premises network to the IBM Cloud VPC network. Client VPN for VPC offers client-to-site servers, which allow clients on the internet to connect to VPN servers, while still maintaining secure connectivity. see [VPNs for VPC overview]{https://cloud.ibm.com/docs/vpc?topic=vpc-vpn-overview}


## Network Components for Red Hat OpenShift Virtualization
{: #virt-sol-network-design-rove}

[OpenShift Virtualization]{: tag-red}

## Open Virtual Networking (OVN)
{: #virt-sol-network-design-ovn}

[OpenShift Virtualization]{: tag-red}

OpenShift Virtualization uses **Open Virtual Networking (OVN)** as the built-in SDN solution for the hosted workloads.

OpenShift has the OVN-Kubernetes CNI plugin. It is the recommended option for OpenShift Virtualization to support VM networking use cases. OVN-Kubernetes is based on OVN and it uses Open vSwitch (OVS) on every worker node. OVN supports multi-tenancy, NetworkPolicies, and hybrid VM/pod networking. Red Hat OpenShift on IBM Cloud VPC will support OVN, and OVN will be provided as an option in addition to the current Calico SDN. 

OVN-Kubernetes provides an L3-routed overlay network using Geneve tunneling between the nodes. This allows the pods, hosted on the nodes, to communicate with each other even when on different worker nodes or even worker nodes between different zones.

In OpenShift Container Platform with OVN, the following three networking topologies represent the approaches to providing secondary network connectivity to pods and VMs: 

* layer 2
* layer 3
* localnet

In IBM Cloud VPC, **OVN layer 2** and **OVN localnet** are the two key topologies with user-defined network (UDN) concept. **OVN layer 2** provides a similar overlay concept as NSX's overlay segments. It also uses Geneve encapsulation like in NSX. **Localnet** provides VLAN access to the underlying network. In IBM Cloud VPC, this means providing access to VPC subnets using a VNI VLAN attachment for each VM or POD using a **Localnet** topology. 

## OVN User Defined Networks
{: #virt-sol-network-design-udn}

[OpenShift Virtualization]{: tag-red}

A **User Defined Network (UDN)** supports seamless integration between OpenShift's OVN-Kubernetes cluster network and existing external networks, and features targeted networking solutions that cross over that boundary. UDN improves the flexibility and segmentation capability of the default layer 3 Kubernetes pod network by enabling custom **layer 2**, **layer 3**, and **localnet** network segments that act as either primary or secondary networks for container pods and virtual machines. Using the default OpenShift OVN-Kubernetes networking, UDN provides a set of network semantics that network administrators and applications are already familiar with, but it also augments OpenShift’s existing Multus-enabled secondary CNI capability by providing a comparable experience and feature set to all network segments.

A **User Defined Network (UDN)** in OpenShift is a custom, secondary network that you define on top of the default cluster network provided by OVN-Kubernetes. UDNs let you create isolated layer 2 or Layer 3 networks with their own IP subnets, gateways, and routing domains, independent of the primary pod network. They’re commonly used when workloads need:

* **Network isolation** from other applications in the cluster.
* **Custom IP address ranges** or overlapping subnets.
* **Direct control over east–west traffic** between selected namespaces or workloads.
* **Integration with virtual machines** (KubeVirt) or appliances that need multiple interfaces.

Unlike the default pod network, which all pods share, UDNs are **explicitly attached to namespaces**. Each UDN results in an additional logical switch in OVN, and when labeled as the namespace’s **primary user-defined network**, all pods in that namespace use it as their main network instead of the cluster default.

**Cluster User Defined Network (CUDN)** is an expansion to the UDN concept, which expands the usage of UDNs' capabilities. CUDN is a **cluster-scoped** resource which means it **does not belong to any namespace**. It is created once and is visible to all namespaces. A `NetworkAttachmentDefinition` (NAD) is **namespace-scoped**, so you need one per namespace that wants to use the network. A NAD is created automatically to the namespace, when the specific namespace is added to the CUDN definition.

In IBM Cloud, **layer 2** and **localnet** are the UDN network topologies that you can use for VM networking with OVN.

An **OVN layer 2** network is essentially a software-defined layer 2 broadcast domain, similar to an NSX overlay segment or legacy VLAN, but implemented entirely in OVN on top of the cluster’s existing network using Geneve encapsulation. A layer 2 network allows pods or VMs to communicate as if they were on the same Ethernet segment. pods/VMs in the network can discover each other via ARP, and you can have things like broadcast, multicast, and direct MAC-to-MAC communication. Your OpenShift cluster already has a primary cluster network (pods get IPs from the default cluster CIDR, routed via OVN). A secondary layer 2 network is one you define yourself (via a ClusterUserDefinedNetwork, CUDN). A **secondary layer 2 network** is any **extra network** you create beyond the default pod network. There is no built-in DNS resolution for pod names on secondary networks. These networks are just L2 broadcast domains created by OVN which provide you IPAM, MACs, and connectivity, but not service discovery. Currently, the traffic from the **primary layer 2 network** is source NATted when the traffic egresses the VM. **Secondary layer 2 networks** are isolated by default. 

An **OVN Localnet** provides VMs and pods VLAN access to the underlying network. In IBM Cloud VPC, this means providing access to VPC subnets using VNI VLAN attachments. With VLAN attachments, your VMs running on OpenShift Virtualization can be attached to a VPC subnet, similar to VPC Virtual Server Instances or VPC Bare Metal Servers. With this approach, you can leverage the VPC subnets for your new or migrated VMs. Each virtual machines NIC attached to the VPC subnet, needs to reserve and IP address from the subnet, a virtual network interface (VNI) and a VNI VLAN attachment. A VNI and a VNI VLAN attachments have settings and security groups that define their characteristics, for example are they allowed to float between worker nodes and what VLAN ID should they use inside the OVN and OVS. When designing security group rules, note that when using localnet some switching happens inside the OVS and never leaves the worker node, and security rule rules can only be applied if the traffic reaches IBM Cloud VPC.
