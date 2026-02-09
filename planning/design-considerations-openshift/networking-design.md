---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-09"

keywords: ROKS, network, layer2, localnet

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Network design for Red Hat OpenShift Virtualization
{: #virt-sol-openshift-network-design}

IBM Cloud® Virtual Private Cloud (VPC) is a virtual network that is linked to your account. It gives you cloud security, with the ability to scale dynamically, by providing fine-grained control over your virtual infrastructure and your network traffic segmentation.
{: shortdesc}

The network design in Red Hat OpenShift Virtualization on IBM Cloud VPC has the following distinct layers.

* VPC networking
* Red Hat OpenShift networking
* OVN networking

The key network architecture elements are shown in the following diagram.

![Red Hat OpenShift Virtualization on IBM Cloud Network](../../images/openshift/openshift-virtualization-high-level-network.svg "Red Hat OpenShift Virtualization on IBM Cloud Network"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Network" caption-side="bottom"}

## IBM Cloud VPC networking
{: #virt-sol-openshift-network-design-vpc}

You use IBM Cloud VPC networking to deploy and manage cloud resources. It provides the foundation for your workloads, including virtual servers, containers, and bare metal deployments, that can help ensure network segmentation, security, and scalability.

You need to create a VPC to provision a ROKS cluster.

### Default private networking with subnets
{: #virt-sol-openshift-network-design-vpc-subnets}

You need to create a VPC subnet in at least one availability zone to provision a ROKS cluster. For more information, see [Default private networking with subnets](/docs/openshift?topic=openshift-vpc-subnets).

### Load balancers
{: #virt-sol-openshift-network-design-vpc-lb}

A Red Hat OpenShift ingress controller is deployed to your Red Hat OpenShift Kubernetes Service (ROKS) cluster that functions as the ingress endpoint for external network traffic. In a ROKS cluster, a VPC application load balancer is automatically created per cluster to expose the ingress controller. For more information, see [Load-balancers](/docs/openshift?topic=openshift-setup_vpc_alb).

ROKS does the following functions.

* DNS service resolves the route subdomain to the VPC load balancer hostname.
* The VPC load balancer resolves the VPC hostname to an available external IP address of an ingress controller service that was reported as operating properly.
* The VPC load balancer sends the request to an ingress controller service.
* The ingress controller forwards the request to the private IP address of the app pod over the private network.

### Virtual private endpoints
{: #virt-sol-openshift-network-design-vpc-vpe}

Virtual Private Endpoints (VPE) in ROKS environments are primarily used to enable private connectivity between the Red Hat OpenShift cluster and IBM Cloud platform services without network traffic that traverses the public internet.

The following table lists all the virtual private endpoints that are automatically provisioned by IBM Cloud for essential cluster operations.

| Virtual private endpoint | Managed by | Description
| -------------- | -------------- | -------------- |
| iks-api | Kubernetes Service API | * Private access to the IBM Cloud Kubernetes Service API  \n * Cluster management operations (kubectl, oc commands)  \n * Worker node to control plane communication  \n * IBM Cloud CLI operations (`ibmcloud ks` commands)  \n * Enables private-only cluster configurations |
| iks-riaas | VPC Infrastructure services | * Private access to VPC Infrastructure APIs  \n * Worker node provisioning and lifecycle management  \n * Storage volume attachment and management  \n * VPC networking operations (load balancers, security groups)  \n * Infrastructure resource management  \n * Used by IBM Cloud cluster autoscaler, storage CSI drivers for volume operations, load balancer provisioning services, worker node lifecycle controllers |
| iks-registry | Container registry | * Private access to IBM Cloud Container Registry  \n * Pull container images without public internet  \n * Access to public and private registry namespaces  \n * Eliminates public egress charges for image pulls |
| iks-<cluster_id> | Specific cluster instance | * Private endpoint specific to your cluster instance  \n * Direct cluster API access  \n * Used for private-only cluster configurations  \n * Alternative to regional API endpoint  \n * Used by tools that require direct cluster access, service-to-service communication within the VPC, private cluster access patterns |
| iks-cos-config | Cloud Object Storage (Configuration) | * Private access to IBM Cloud Object Storage configuration API  \n * Bucket management and configuration operations  \n * IAM policy and access control management  \n * Service credential operations |
| iks-cos | Cloud Object Storage (Data) | * Private access to IBM Cloud Object Storage S3 API  \n * Object storage data plane operations (PUT/GET/DELETE)  \n * Backup and restore data transfer  \n * Application data storage access |
{: caption="Virtual private endpoints that are provisioned for cluster operations." caption-side="bottom"}

## Red Hat OpenShift Virtualization Networking
{: #virt-sol-openshift-network-design-openshift}

Red Hat OpenShift Virtualization uses the Red Hat OpenShift networking capabilities to provide flexible, software-defined networking for virtual servers that run alongside containerized workloads. It is important to understand the difference for virtual server networking and pod networking. Each virtual server runs within a `virt-launcher` pod that is always connected to the default pod network.

```bash
┌────────────────────────────────┐
│          Worker Node           │
│  ┌──────────────────────────┐  │
│  │      virt-launcher       │  │  ← Kubernetes Pod Security Context
│  │          pod             │  │
│  │  ┌────────────────────┐  │  │
│  │  │   virtual server   │  │  │  ← KVM/QEMU Hypervisor Isolation
│  │  │       (QEMU)       │  │  │
│  │  └────────────────────┘  │  │
│  └──────────────────────────┘  │
└────────────────────────────────┘
```

Depending how you provision and set up your virtual server, it shares a pod network (or it can connect to different networks by using `multus`).

The following example describes the default pod networking in Red Hat OpenShift that you can modify with OVN-Kubernetes networking.

Pod networks (Cluster network)

* Each pod receives a private IP address from the cluster network CIDR
* Provides pod-to-pod communication across nodes
* Pods communicate directly by using their private IPs within the cluster
* Network policies control pod-to-pod traffic at layer 3/4
* Flat network model - all pods can communicate by default
* No NAT between pods (direct pod-to-pod communication)
* Network policies provide segmentation and security
* DNS-based service discovery within cluster
* When a virtual server runs inside the virt-launcher pod the IP is NATed with the virt-launcher pod's IP

IP masquerading (SNAT)

* When pods initiate outbound connections to external networks, the source IP is masqueraded
* The source IP address of the request packet is changed to the IP address of the worker node where the pod runs
* IP masquerading is necessary because pod IPs are not routable outside the cluster
* Return traffic is demasqueraded back to the original pod IP
* External services see requests that come from worker node IPs, not pod IPs

### ClusterIP service
{: #virt-sol-openshift-network-design-openshift-clusterip}

Services provide stable endpoints and load balancing for pods. They abstract pod IPs and provide consistent access points for applications. The ClusterIP Service provides the following functions.

* Creates a virtual IP (ClusterIP) accessible within only the cluster
* ClusterIP is the default service type if not specified
* Provides internal load balancing across backend pods
* Uses kube-proxy or OVN-Kubernetes for traffic distribution

The following use cases are an example of what a ClusterIP is used for.

* Internal microservices communication
* Backend services that don't require external access
* Database services that are accessed by only cluster workloads
* Inter-pod service discovery

### NodePort service
{: #virt-sol-openshift-network-design-openshift-nodeport}

Services provide stable endpoints and load balancing for pods. They abstract pod IPs and provide consistent access points for applications. The NodePort service provides the following functions.

* Exposes service on a static port (30000-32767 range) on each worker node
* Makes service accessible through `<NodeIP>:<NodePort>`
* Automatically creates ClusterIP service
* Traffic to any NodePort forwards to the service

The following example shows the NodePort traffic flow.

* External client connects to `<WorkerNodeIP>:<NodePort>`
* Nodes forward traffic to the ClusterIP service
* The service balances the load to backend pods
* Response follows the reverse path with SNAT

The following use cases are an example of what NodePorts are used for.

* Development and testing environments
* Quick external access without load balancer
* Integration with external load balancers
* Custom load balancing solutions

### Load balancer service
{: #virt-sol-openshift-network-design-openshift-loadbalancer}

On IBM Cloud ROKS, the load balancer service automatically provisions a VPC network load balancer or application load balancer. The load balancer service provides the following functions.

* Automatically provisions an external load balancer
* Assigns external IP or hostname to the service
* Creates NodePort and ClusterIP services automatically
* Provides layer 4 load balancing to service backends

The following example shows the traffic flow in a VPC.

* External client connects to VPC load balancer IP or hostname
* VPC load balancer distributes to worker node NodePorts
* Node forwards to service ClusterIP
* The service balances the load to backend pods

The following use cases are an example of what load balancers are used for.

* Production applications that require dedicated external access
* Non-HTTP protocols (TCP or UDP services)
* Applications that need stable external IPs
* Services that bypass the ingress or route layer

### Red Hat OpenShift routes
{: #virt-sol-openshift-network-design-openshift-routes}

Red Hat OpenShift Routes expose services to external network traffic by mapping FQDNs to backend services, which makes applications accessible outside of the cluster. The following list shows key features of Red Hat OpenShift Routes.

* Layer 7 routing - HTTP/HTTPS traffic with host name-based routing
* Automatic DNS - routes use cluster subdomain: `<route-name>-<namespace>`.apps.`<cluster-domain>`
* Unsecured routes (HTTP)
* TLS termination
   * Edge-terminated routes (TLS at the router)
   * Pass-through routes (TLS at Pod)
   * Reencrypt routes (TLS at the router and Pod)
* HAProxy-based is implemented by the Red Hat OpenShift ingress controller (router)
* Traffic management - Path-based routing, traffic splitting, and session affinity

## Open Virtual Networking (OVN)
{: #virt-sol-openshift-network-design-ovn}

The **OVN-Kubernetes** CNI (Container Network Interface) plug-in is the recommended networking option for Red Hat OpenShift Virtualization that supports virtual server networking use cases that run alongside traditional pod networking. OVN-Kubernetes is based on OVN and uses Open vSwitch (OVS) on every worker node. It supports multi-tenancy, NetworkPolicies, and hybrid virtual server and pod networking. Red Hat OpenShift on IBM Cloud VPC supports OVN-Kubernetes as the default networking plug-in.

In Red Hat OpenShift with OVN, the following three networking topologies provide secondary network connectivity to pods and virtual servers.

* Layer 2 (L2) - Software-defined L2 broadcast domains by using Geneve encapsulation
* Layer 3 (L3) - Routed network segments with custom IP subnets. An L3 network has a separate CIDR per node.
* Localnet - Direct access to underlying physical network VLANs

In Red Hat OpenShift Virtualization on IBM Cloud, **OVN layer 2** and **OVN localnet** are the two primary topologies that are used with User-Defined Networks (UDN).

* OVN Layer 2 provides overlay networking that is similar to NSX overlay segments by using Geneve encapsulation to create software-defined L2 broadcast domains across the cluster. These networks are isolated from the VPC subnets. They require a gateway pod or virtual server that is connected to an OVN Localnet to provide ingress and egress to the VPC subnet and a VPC route.
* OVN Localnet provides VLAN access to the underlying VPC network and is similar to NSX VLAN-backed segments. In IBM Cloud VPC, this direct connectivity enables virtual servers and pods to connect directly to VPC subnets by using a Virtual Network Interface (VNI) and VLAN attachments.

The following diagram presents an overview of the virtual server networking with OVN and `multus`. By default, Kubernetes (and Red Hat OpenShift) assigns a single network interface to each pod by using a primary CNI plug-in (such as OVN-Kubernetes). Multus in Red Hat OpenShift is a CNI plug-in that enables multiple network interfaces for pods and virtual servers.

![OVN Networking with multus](../../images/openshift/openshift-virtualization-ovn-multus.svg "OVN Networking with multus"){: caption="OVN Networking with multus" caption-side="bottom"}

Initially, only OVN Layer 2 networking is available.
{: important}

## OVN User-Defined Networks
{: #virt-sol-openshift-network-design-udn}

[Red Hat OpenShift Virtualization]{: tag-red}

A User-Defined Network (UDN) in Red Hat OpenShift is a custom network that is provided by OVN-Kubernetes. A UDN replaces the default cluster network (also known as the default pod network) UDNs that you use to create networks with their own IP subnets, gateways, and routing domains. UDNs are independent of the primary pod network and are commonly used when workloads require the following functions.

* Network isolation from other applications in the cluster
* Custom IP address ranges or overlapping subnets
* Direct control over east-west traffic between selected namespaces or workloads
* Integration with virtual servers (Red Hat OpenShift virtualization) that require multiple network interfaces
* Dedicated network segments for security or compliance requirements

Unlike the default pod network, UDNs are explicitly attached to namespaces. Each UDN creates an extra logical switch in OVN. When a UDN is labeled as the namespace's primary user-defined network, all pods and virtual servers in that namespace use it as their main network instead of the cluster default.

Cluster User-Defined Network (CUDN) expands the UDN concept by providing a cluster-scoped resource that does not belong to any specific namespace. A CUDN is created and is associated with one or more namespaces. Unlike namespace-scoped `NetworkAttachmentDefinition` (NAD) resources that require one per namespace, a CUDN automatically creates NADs in namespaces when those namespaces are added to the CUDN definition.

UDNs provide flexible networking options based on scope, attachment method, and topology:

Network scope

   * Namespace-scoped UDN - Network definition that is limited to a single namespace, requiring separate NetworkAttachmentDefinitions per namespace
   * Cluster-scoped CUDN - Network definition available cluster-wide, automatically creating NetworkAttachmentDefinitions in selected namespaces

Attachment method

   * Primary network - Acts as the default network for all pods/virtual servers in the namespace, replacing the cluster default network
   * Secondary network - Attached through Multus CNI to provide extra network interfaces to pods/virtual servers alongside the primary network

Network topology

   * Layer 2 - Software-defined L2 broadcast domain by using Geneve encapsulation that enables ARP-based discovery and MAC-to-MAC communication
   * Layer 3 - Routed network segments with custom IP subnets and gateways
   * Localnet - Direct VLAN access to underlying VPC subnets by using Virtual Network Interface (VNI) attachments

You can combine these characteristics to create customized networking solutions. For example, a cluster-scoped CUDN that uses localnet topology can provide multiple namespaces with direct VPC subnet access as either a primary or secondary network.

### OVN Layer 2 networks
{: #virt-sol-openshift-network-design-udn-layer2}

An OVN layer 2 network is a software-defined Layer 2 broadcast domain that is similar to an NSX overlay segment or traditional VLAN. Layer 2 is implemented entirely within OVN by using Geneve encapsulation over the cluster's existing network infrastructure. A Layer 2 network allows pods and virtual servers to communicate as if they were on the same Ethernet segment, with support for ARP discovery, broadcast, multicast, and direct MAC-to-MAC communication.

A Red Hat OpenShift cluster has a primary cluster network where pods and virtual servers receive IPs from the default cluster CIDR that is routed through OVN. You define a secondary Layer 2 network through a `ClusterUserDefinedNetwork` (CUDN) or namespace-scoped UDN. A secondary Layer 2 network is any extra network that you create beyond the default pod network.

The following items are key characteristics of Layer 2 networks.

* Provide Layer 2 broadcast domains that are created by OVN with IPAM, MAC assignment, and connectivity
* No built-in DNS resolution for pod names on secondary networks
* Traffic from the primary Layer 2 network is source NATed when you egress the virtual server and is also routed access to the Layer 2 network that can be configured by using `FRR-K8s` and VPC routes
* Secondary Layer 2 networks are isolated by default with no direct internet access unless explicitly configured
* Suitable for virtual server-to-virtual server communication within the cluster and multicast-dependent applications

### OVN Localnet networks
{: #virt-sol-openshift-network-design-udn-localnet}

An OVN Localnet network provides virtual servers and pods with direct VLAN access to the underlying VPC network infrastructure. OVN Localnet enables virtual servers and pods to connect to VPC subnets by using a virtual network interface (VNI) and VLAN attachments.

With VLAN attachments, you can directly attach virtual servers that run on Red Hat OpenShift Virtualization to VPC subnets. You can use this approach to use your existing VPC subnet design for new or migrated virtual servers by providing consistent networking across your workloads.

Localnet networking requires that each virtual server NIC that is attached to a VPC subnet needs the following requirements:

* A Virtual Network Interface (VNI) resource that defines an IP address that is reserved from the VPC subnet and one or more security groups that control inbound and outbound traffic to the VNI.
* A bare metal server VLAN attachment. This VLAN attachment defines floating capability whether the attachment can move between worker nodes needs to be enabled for the virtual server to Live Motion to another worker node.
* A VLAN ID. The VLAN tag associates the PCI interface on the worker nodes. Typically this association is a one-to-one mapping between VLAN ID and VPC subnet.

When you design security group rules for localnet networks, consider that some network switching occurs within OVS on the worker node and never reaches the VPC infrastructure. Security group rules are applied only to traffic that traverses the VPC network fabric. Traffic between virtual servers on the same worker node can bypass VPC security controls.
{: note}

The following examples are use cases for Localnet.

* Migrated virtual servers that require existing VPC subnet IP addresses
* Integration with existing VPC security groups and network policies
* Direct connectivity to other VPC resources
* Compliance requirements for network segmentation by using VPC subnets
* Hybrid architectures that require consistent IP addressing across VPC and Red Hat OpenShift Virtualization

## Next steps
{: #virt-sol-openshift-network-design-next-steps}

Now that you understand the networking design for Red Hat OpenShift Virtualization, explore these related topics:

- **Security**: Review [security design considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-security-design-overview) including network policies and SCCs
- **Compute**: Explore [compute design options](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-compute-design) for worker nodes
- **Storage**: Learn about [storage design patterns](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-storage-design-overview) for persistent volumes
- **Observability**: Understand [observability solutions](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-openshift-observability-design-overview) for network monitoring
