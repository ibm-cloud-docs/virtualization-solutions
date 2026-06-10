---

copyright:
  years: 2026, 2026
lastupdated: "2026-06-10"

keywords: ROKS, OpenShift Virtualization, OVN, UDN, CUDN, Localnet, Masquerade

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization
account-plan: paid
completion-time: 60m
---

{{site.data.keyword.attribute-definition-list}}

# OVN networking in OpenShift for vSphere administrators
{: #virt-sol-network-options-overview}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}

If you have used NSX-T, you already understand the core idea. OVN (Open Virtual Network) is the software-defined networking layer built into OpenShift. It replaces the older Calico networking stack in the same way NSX-T replaced standard vSwitches, by moving network intelligence into software rather than relying on the underlying physical network. OVN handles all internal cluster networking: how VMs talk to each other, how they reach external networks, and how traffic is isolated between tenants or workloads.
{: shortdesc}

When deploying a new ROKS cluster, you choose either OVN or Calico at deployment time. There is no migration path between them, just as you cannot switch between a standard vSwitch and a dvSwitch on a running VM without a planned transition.
{: note}

## Network types
{: #virt-sol-network-options-types}

Before going further, the following table shows how OVN concepts map to things you already know in vSphere:

| OVN concept | vSphere equivalent |
| ----------- | ------------------ |
| UDN or CUDN | NSX logical segment or dvPortGroup |
| Cluster infrastructure network (Primary) | Background plumbing - health checks, DNS, Kubernetes services |
| VM workload network (Secondary) | Your actual VM network - the IP your application uses |
| Masquerade | NAT network - VMs go out, nothing comes directly in |
| Layer 2 Secondary | Isolated NSX overlay segment - static IPs, full control |
| Layer 2 Primary | Routed NSX segment with a BGP uplink to the physical network |
| Localnet | VLAN-backed dvPortGroup - direct passthrough to the physical network |
{: caption="OVN concepts mapped to familiar vSphere networking constructs" caption-side="bottom"}

## UDN and CUDN
{: #virt-sol-network-options-udn-cudn}

OpenShift groups workloads into **namespaces**. Think of these as resource pools or folders that isolate one application or team from another. Network definitions follow the same pattern:

UDN (User Defined Network)
:   Scoped to a single namespace - the equivalent of a port group that only one cluster or folder can use.

CUDN (Cluster User Defined Network)
:   Can span multiple namespaces - the equivalent of a shared dvPortGroup accessible from multiple clusters.

In practice, you almost always configure CUDNs. They are more flexible and avoid duplicating network definitions for every namespace that needs access to the same segment.

## Cluster infrastructure network vs. VM workload network
{: #virt-sol-network-options-primary-secondary}

This is the most important concept to get right, because the terminology is counterintuitive. OVN labels networks as **Primary** or **Secondary**, but those names describe their role in the OpenShift cluster, not their importance to your VMs. In practice, the mapping works like this:

Cluster infrastructure network (Primary)
:   Background plumbing that handles Kubernetes-internal traffic - health checks, service discovery, load balancer VIPs, and internal DNS. Typically, your VMs do not use this network for application traffic. Leave it as a masquerade network. It just needs to exist so that OpenShift's internal pods and services keep working.

VM workload network (Secondary)
:   Where your VMs actually live. This is the network your applications use, the IP address you would SSH to, and the subnet you manage. Because it is a Secondary network, you have full control over IP addressing, including static assignment. This is the right choice for almost every VM workload.

The naming feels backwards if you think of it from a VM perspective. Think of it instead from the cluster's perspective: Primary means "the cluster's own network", Secondary means "everything else you attach". For VM workloads, Secondary is not an afterthought - it is the main event.
{: note}

A namespace does not have to have a cluster infrastructure network at all. If your VMs have no need for Kubernetes services, load balancers, or internal DNS, you can skip it entirely and run Secondary networks only.

## Network topologies
{: #virt-sol-network-options-topologies}

### Masquerade - cluster infrastructure network (NAT)
{: #virt-sol-network-options-masquerade}

This is the standard choice for the cluster infrastructure primary network and what the pods running in the cluster use. If you use it for VM networks, it works exactly like a VM sitting behind a NAT rule on an NSX Edge:

* VMs can initiate outbound connections to the outside world.
* Nothing external can reach the VM directly - traffic must go through a load balancer or service, just as you would use a DNAT rule on an NSX Edge to expose a service.
* IP addressing is automatic; you do not manage it.
* Full Kubernetes network features are available: load balancers, services, network policies, DNS.

Some edge use cases might prefer to use the Masquerade network for your VM, such as when your VMs need direct access to Kubernetes services, but the majority of VM use cases do not use the Masquerade network.

### Layer 2 Primary - directly routable VM network (BGP uplink)
{: #virt-sol-network-options-layer2-primary}

This is the equivalent of an NSX overlay segment connected to a Tier-0 gateway with BGP route redistribution enabled. It is a less common choice, used when you need VMs to be directly reachable from external networks without going through a load balancer or NAT:

* VMs receive IPs directly from the segment's subnet and are reachable from outside via routing.
* External networks reach individual VMs through BGP route advertisements from each worker node - equivalent to what the NSX Edge SR does when redistributing connected routes.
* Supports Multicast and Broadcast.
* When you enable a Layer 2 Primary network in a namespace, Masquerade is removed from that namespace. The two cannot coexist.

IP addressing uses a first-come first-served DHCP system. IPs do not change for the lifetime of a VM, but you cannot pre-assign a specific IP to a specific VM. If you need static IPs, use a Layer 2 Secondary network instead.
{: important}

Currently VPC does not support a Dynamic Routing Service (BGP), therefore, custom VPC routes with next hop must be used.
{: important}

There might be some edge use cases where you prefer to use the Primary network for your VM, such as when your VMs need direct access to Kubernetes services, but the majority of VM use cases do not use the Primary network.

### Layer 2 Secondary - VM workload network (isolated segment)
{: #virt-sol-network-options-layer2-secondary}

This is one of the network types you use for almost every VM use case. It is the equivalent of an NSX overlay segment with no external uplink - a dedicated segment you control completely:

* Static IP assignment is supported. You manage addressing yourself, which is the normal expectation for VM workloads.
* No automatic connection to the outside world. External access is provided by attaching a virtual firewall or router to the segment, exactly as you would attach an NSX Edge or a pfSense VM to a segment in vSphere.
* Supports Multicast and Broadcast.
* You can attach multiple Layer 2 Secondary networks to the same VM - useful for separating application, management, and storage traffic onto different segments.
* Can span namespaces when configured as a CUDN.

This is the right choice for the vast majority of VM workloads in OpenShift Virtualization.

### Localnet - direct VLAN passthrough
{: #virt-sol-network-options-localnet}

This is the closest equivalent to a VLAN-backed standard port group or dvPortGroup with no NSX overlay. The VM connects directly to whatever network the physical host is connected to:

* No SDN overlay - traffic bypasses OVN's software switching entirely.
* No OpenShift network services are available (no load balancing, network policies, or internal DNS).
* No built-in IP address management - you assign IPs using VPC VNIs.
* Configured per availability zone.
* Multiple Localnets can be attached to the same VM.

Use Localnet for VMs that need direct access to an existing VPC subnet. For example, migrating a legacy workload that communicates with on-premises systems over a specific VLAN, or connecting to a storage network that exists outside the cluster.

## Summary
{: #virt-sol-network-options-summary}

| Network type | Role | External access | Static IPs | K8s services | Broadcast / multicast |
| ------------ | ---- | --------------- | ---------- | ------------ | --------------------- |
| Masquerade | Cluster infrastructure | Outbound via NAT | No | Yes | No |
| Layer 2 Primary | Directly routable VMs | Yes, via BGP routing | No (DHCP only) | Yes | Yes |
| Layer 2 Secondary | VM workload network | Via attached router/firewall | Yes | Yes | Yes |
| Localnet | Physical VLAN passthrough | Direct VLAN access | Manual | No | Yes |
{: caption="Comparison of OVN network topology options for virtual machine workloads" caption-side="bottom"}

## Next steps
{: #virt-sol-network-options-next-steps}

Now that you understand the available OVN network options, explore these related topics:

* Review [Layer 2 primary UDN examples for Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-primary-udn-examples-for-red-hat-openshift-virtualization)
* Review [Layer 2 Secondary UDN examples for Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-secondary-udn-examples-for-red-hat-openshift-virtualization)
* Review [Localnet UDN examples for Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-localnet-udn-examples)

