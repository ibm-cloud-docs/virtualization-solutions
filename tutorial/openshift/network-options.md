---

copyright:
  years: 2026
lastupdated: "2026-06-29"

keywords: Red Hat OpenShift Virtualization, IBM Cloud, OVN networking, OpenShift networking, OpenShift Virtualization networking, OVN Kubernetes, User Defined Network, UDN, Cluster User Defined Network, CUDN, masquerade network, Layer 2 primary, Layer 2 secondary, localnet, virtual machine networking, VM workload network, cluster infrastructure network, vSphere networking comparison, ROKS networking

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization
account-plan: paid
completion-time: 60m
---

{{site.data.keyword.attribute-definition-list}}

# Open Virtual Network (OVN) networking in Red Hat OpenShift for vSphere administrators
{: #virt-sol-network-options-overview}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}

Choose the right Open Virtual Network (OVN) networking option for {{site.data.keyword.redhat_openshift_notm}} Virtualization on {{site.data.keyword.cloud_notm}}. Compare masquerade, Layer 2 primary, Layer 2 secondary, and localnet networks, and understand when to use user-defined networks (UDNs) and cluster user-defined networks (CUDNs) for virtual machine workloads.
{: shortdesc}

OVN is the software-defined networking layer that is built into {{site.data.keyword.openshiftlong_notm}}. Similar to how NSX-T replaces standard vSwitches, OVN replaces the older Calico networking stack by moving network intelligence into software instead of relying on the underlying physical network. OVN handles all internal cluster networking, including how virtual machines communicate with each other, how they reach external networks, and how traffic is isolated between tenants or workloads.

When you deploy a new {{site.data.keyword.openshiftlong}} cluster, you choose either OVN or Calico at deployment time because there is no migration path between them. Switching between networking stacks requires planning the transition, similar to switching between a standard vSwitch and a dvSwitch on a running virtual machine.
{: note}

## Network types
{: #virt-sol-network-options-types}

The following table maps OVN concepts to familiar vSphere constructs:

| OVN concept | vSphere equivalent |
| ----------- | ------------------ |
| UDN or CUDN | NSX logical segment or distributed virtual port group (dvPortGroup) |
| Cluster infrastructure network (primary) | Background plumbing such as health checks, DNS, and Kubernetes services |
| VM workload network (secondary) | Your actual virtual machine network and the IP address that your application uses |
| Masquerade | Network Address Translation (NAT) network where virtual machines go out, but nothing comes directly in |
| Layer 2 secondary | Isolated NSX overlay segment with static IP addresses and full control |
| Layer 2 primary | Routed NSX segment with a BGP uplink to the physical network |
| Localnet | VLAN-backed dvPortGroup with direct pass-through to the physical network |
{: caption="OVN concepts mapped to familiar vSphere networking constructs" caption-side="bottom"}

## UDNs and CUDNs
{: #virt-sol-network-options-udn-cudn}

{{site.data.keyword.redhat_openshift_notm}} groups workloads into namespaces. Think of namespaces as resource pools or folders that isolate one application or team from another. Network definitions follow the same pattern:

User-defined network (UDN)
:   Scoped to a single namespace. This network is equivalent to a port group that only one cluster or folder can use.

Cluster user-defined network (CUDN)
:   A CUDN can span multiple namespaces. This network is equivalent to a shared `dvPortGroup` that is accessible from multiple clusters.

You usually configure CUDNs because they are more flexible and avoid duplicate network definitions for every namespace that needs access to the same segment.

## Cluster infrastructure network versus VM workload network
{: #virt-sol-network-options-primary-secondary}

The terminology is counterintuitive, so this concept is important to understand. OVN labels networks as primary or secondary, but those names describe their role in the {{site.data.keyword.openshiftlong_notm}} cluster, not their importance to your virtual machines. The mapping is as follows:

Cluster infrastructure network (primary)
:   Background plumbing that handles Kubernetes internal traffic, such as health checks, service discovery, load balancer virtual IP addresses (VIPs), and internal DNS. Typically, your virtual machines do not use this network for application traffic. Keep it as a masquerade network. This network must exist so that internal pods and services in {{site.data.keyword.openshiftlong_notm}} continue to work.

VM workload network (secondary)
:   Where virtual machines run. Your applications use this network, you connect to its IP address by using SSH, and you manage its subnet. Because it is a secondary network, you have full control over IP addressing, including static assignment. This network is the right choice for almost every virtual machine workload.

The naming might seem counterintuitive from a virtual machine perspective. From the cluster perspective, primary means "the cluster's own network," and secondary means "everything else that you attach." For virtual machine workloads, secondary is not an afterthought. This network is the main network.
{: note}

A namespace does not need a cluster infrastructure network. If your virtual machines do not need Kubernetes services, load balancers, or internal DNS, you can skip it and run only secondary networks.

## Network topologies
{: #virt-sol-network-options-topologies}

### Masquerade - cluster infrastructure network (NAT)
{: #virt-sol-network-options-masquerade}

Use this network as the standard choice for the cluster infrastructure primary network and for the pods that run in the cluster. If you use it for virtual machine networks, it works like a virtual machine behind a NAT rule on an NSX Edge:

* Virtual machines can initiate outbound connections to the outside world.
* Virtual machines cannot receive direct inbound connections from external sources. Traffic must go through a load balancer or service, just as you would use a destination NAT (DNAT) rule on an NSX Edge to expose a service.
* Virtual machines use automatic IP addressing. You do not manage it.
* Virtual machines can use full Kubernetes network features, including load balancers, services, network policies, and DNS.

Some edge use cases might require the masquerade network for your virtual machine, such as cases where your virtual machines need direct access to Kubernetes services. However, most virtual machine use cases do not use the masquerade network.

### Layer 2 primary - directly routable virtual machine network (BGP uplink)
{: #virt-sol-network-options-layer2-primary}

This topology is equivalent to an NSX overlay segment that is connected to a Tier-0 gateway with BGP route redistribution enabled. Use this less common option when you need virtual machines to be directly reachable from external networks without going through a load balancer or NAT:

* Virtual machines receive IP addresses directly from the segment subnet and remain reachable from outside through routing.
* External networks reach individual virtual machines through BGP route advertisements from each worker node. This approach is equivalent to what the NSX Edge service router (SR) does when it redistributes connected routes.
* Virtual machines support multicast and broadcast.
* Namespaces cannot use masquerade when you enable a Layer 2 primary network. The two cannot coexist.

IP addressing uses a first-come, first-served DHCP system. IP addresses do not change during the lifetime of a virtual machine, but you cannot preassign a specific IP address to a specific virtual machine. If you need static IP addresses, use a Layer 2 secondary network instead.
{: important}

Currently, VPC does not support a dynamic routing service (BGP). Use custom VPC routes with a next hop instead.
{: important}

Some edge use cases might require the primary network for your virtual machine, such as cases where your virtual machines need direct access to Kubernetes services. However, most virtual machine use cases do not use the primary network.

### Layer 2 secondary - virtual machine workload network (isolated segment)
{: #virt-sol-network-options-layer2-secondary}

Use this network type for almost every virtual machine use case. This topology is equivalent to an NSX overlay segment with no external uplink: a dedicated segment that you control completely.

* This network supports static IP assignment. You manage IP addressing, which is the normal expectation for virtual machine workloads.
* This network does not connect automatically to the outside world. To provide external access, attach a virtual firewall or router to the segment, just as you would attach an NSX Edge or a pfSense virtual machine to a segment in vSphere.
* Supports multicast and broadcast.
* You can attach multiple Layer 2 secondary networks to the same virtual machine. This setup is useful for separating application, management, and storage traffic onto different segments.
* The network can span namespaces when it is configured as a CUDN.

This network is the right choice for most virtual machine workloads in {{site.data.keyword.redhat_openshift_notm}} Virtualization.

### Localnet - direct VLAN pass-through
{: #virt-sol-network-options-localnet}

This topology is the closest equivalent to a VLAN-backed standard port group or distributed virtual port group (dvPortGroup) with no NSX overlay. The virtual machine connects directly to the network to which the physical host is connected:

* This topology does not use a software-defined networking (SDN) overlay. Traffic bypasses OVN software switching entirely.
* This topology does not provide {{site.data.keyword.openshiftlong_notm}} network services such as load balancing, network policies, or internal DNS.
* This topology does not include built-in IP address management. You assign IP addresses by using VPC virtual network interfaces (VNIs).
* You configure this topology per availability zone.
* This topology supports multiple localnets on the same virtual machine.

Use localnet for virtual machines that need direct access to an existing VPC subnet. For example, you might migrate a legacy workload that communicates with on-premises systems over a specific VLAN or connect to a storage network that exists outside the cluster.

## Summary
{: #virt-sol-network-options-summary}

| Network type | Role | External access | Static IPs | Kubernetes services | Broadcast / multicast |
| ------------ | ---- | --------------- | ---------- | ------------------- | --------------------- |
| Masquerade | Cluster infrastructure | Outbound through NAT | No | Yes | No |
| Layer 2 primary | Directly routable virtual machines | Yes, through BGP routing | No (DHCP only) | Yes | Yes |
| Layer 2 secondary | Virtual machine workload network | Through an attached router or firewall | Yes | Yes | Yes |
| Localnet | Physical VLAN pass-through | Direct VLAN access | Manual | No | Yes |
{: caption="Comparison of OVN network topology options for virtual machine workloads" caption-side="bottom"}

## Next steps
{: #virt-sol-network-options-next-steps}

Review these related topics for more details about specific OVN network options:

* Review [Layer 2 primary UDN examples for {{site.data.keyword.redhat_openshift_notm}} Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-primary-udn-examples-for-red-hat-openshift-virtualization)
* Review [Layer 2 secondary UDN examples for {{site.data.keyword.redhat_openshift_notm}} Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-secondary-udn-examples-for-red-hat-openshift-virtualization)
* Review [Localnet UDN examples for {{site.data.keyword.redhat_openshift_notm}} Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-localnet-udn-examples)
