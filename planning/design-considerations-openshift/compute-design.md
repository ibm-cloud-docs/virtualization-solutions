---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-09"

keywords: ROKS, red hat OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Compute design for Red Hat OpenShift virtualization
{: #virt-sol-openshift-compute-design}

{{site.data.keyword.cloud}} provides compute options that support diverse workloads, from traditional applications to modern cloud-native solutions. These offerings deliver flexibility, scalability, and enterprise-grade security, that help you deploy diverse workloads.
{: shortdesc}

IBM Cloud VPC offers two compute options:

- [Virtual servers](/docs/vpc?topic=vpc-about-advanced-virtual-servers)
- [Bare metal server](/docs/vpc?topic=vpc-about-bare-metal-servers)

The key compute architecture elements are shown in the following diagram.

![Red Hat OpenShift Virtualization on IBM Cloud Compute](../../images/openshift/openshift-virtualization-high-level-compute.svg "Red Hat OpenShift Virtualization on IBM Cloud Compute"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Compute" caption-side="bottom"}

## Red Hat OpenShift worker nodes
{: #virt-sol-openshift-compute-design-workers}

{{site.data.keyword.openshiftshort}} (ROKS) is a managed Kubernetes service that uses Red Hat OpenShift clusters where you can deploy and manage virtualized and containerized applications. A ROKS cluster has a managed control plane and one or more worker pools. A worker pool consists of two or more compute hosts that are called worker nodes. Worker pools contain worker nodes of the same profile of CPU, memory, operating system, attached storage, and other properties. The worker nodes are managed by the Kubernetes control plane, which controls and monitors all Kubernetes resources that are in the cluster. The Kubernetes scheduler decides which worker node to deploy resources on and accounts for deployment requirements and available capacity in the cluster.

After a cluster is created, you can add more worker nodes to a pool by resizing it or by adding extra worker pools. Clusters that have a worker pool in only one zone are called single zone clusters. For high availability, you can create multizone clusters with worker pools that span multiple availability zones. You can create multizone clusters, but they are not recommended for virtualization workloads because of storage latency.

IBM Cloud Bare Metal Servers for VPC are recommended in the worker pool to run your production virtualized workloads because Red Hat supports only bare metal worker nodes for production virtualized workloads. The bare metal nodes must be provisioned with local NVMe drives so they can be used by Red Hat OpenShift Data Foundation (ODF) as backing storage.

For more information, see [Red Hat OpenShift overview](/docs/openshift?topic=openshift-overview).

## IBM Cloud Virtual Servers for VPC
{: #virt-sol-openshift-compute-design-vsi}

IBM Cloud Virtual Servers for VPC provide secure, isolated virtual machines that are deployed within a Virtual Private Cloud environment. You can use these instances for production workloads, and development and test environments that require flexible resource allocation and comprehensive infrastructure control.

For more information about virtual servers, see [About virtual server instances for VPC](/docs/vpc?topic=vpc-about-advanced-virtual-servers).

The following table lists the key features for IBM Cloud Virtual Servers for VPC.

| Feature | Description |
| -------------- | -------------- |
| Customizable profiles | Balanced, compute-optimized, memory-optimized, GPU, and very high memory configurations |
| Flexible tenancy | Shared tenancy infrastructure with optional dedicated host placement for compliance requirements |
| Advanced networking | Integration with VPC Security Groups, Network ACLs, Load Balancers, and VPN connectivity |
| Persistent storage | IBM Cloud Block Storage and File Storage for VPC with configurable IOPS and encryption |
| Operating system flexibility | IBM-provided stock images or bring-your-own custom images |
| Scalability | Vertical scaling through profile changes and horizontal scaling through instance groups with auto-scaling |
{: caption="IBM Cloud Virtual Servers for VPC key features" caption-side="bottom"}

For more information about virtual server instance profiles that are supported by ROKS, see [IBM Cloud Docs - Worker Nodes VPC flavors](/docs/openshift?topic=openshift-vpc-flavors).

## IBM Cloud Bare Metal Servers for VPC
{: #virt-sol-openshift-compute-design-bms}

IBM Cloud Bare Metal Servers for VPC provide single-tenant, dedicated physical servers that deliver maximum performance, security, and control. These servers are required for production deployments of Red Hat OpenShift Virtualization.

The following table lists the key features for IBM Cloud Bare Metal Servers for VPC.

| Feature | Description |
| -------------- | -------------- |
| Single-tenant isolation | Dedicated physical hardware with no resource sharing |
| High-performance NVMe storage | Local NVMe drives for Red Hat OpenShift Data Foundation deployments |
| Consistent performance | Predictable performance without virtualization resource overuse |
| Large memory configurations | Support for memory-intensive virtualization workloads |
| Network performance | High-bandwidth, low-latency networking for VM traffic |
{: caption="IBM Cloud Bare Metal Servers for VPC key features" caption-side="bottom"}

Bare Metal worker nodes that are supported by ROKS varies by region and Availability Zone. For more information, see [Worker node VPC flavors](/docs/openshift?topic=openshift-vpc-flavors).

The following table shows the typical specifications for a bare metal profile.

| Instance profile | Cores | GiB RAM | Storage |
| -------------- | -------------- | -------------- | -------------- |
| cx2.metal.96x192 | 48 | 192 | - |
| cx2d.metal.96x192 | 48 | 192 | 8x3.2TB SSD NVMe drives |
| bx2.metal.96x384 | 48 | 384 | - |
| bx2d.metal.96x384 | 48 | 384 | 8x3.2TB SSD NVMe drives |
| mx2.metal.96x768 | 48 | 768 | - |
| mx2d.metal.96x768 | 48 | 768 | 8x3.2TB SSD NVMe drives |
{: caption="Typical bare metal profile specifications" caption-side="bottom"}

For more information about bare metal servers, see [About Bare Metal Servers for VPC](/docs/vpc?topic=vpc-about-bare-metal-servers).

## Next steps
{: #virt-sol-openshift-compute-design-next-steps}

Now that you understand the compute design for Red Hat OpenShift Virtualization, explore these related topics:

- **Networking**: Review [networking design considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-network-design) for OpenShift and OVN
- **Storage**: Explore [storage design options](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-storage-design-overview) including OpenShift Data Foundation
- **Security**: Learn about [security design patterns](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-security-design-overview) for OpenShift workloads
- **Migration**: Review [migration strategies](/docs/virtualization-solutions?topic=virtualization-solutions-vsphere-openshift-migration) using MTV
