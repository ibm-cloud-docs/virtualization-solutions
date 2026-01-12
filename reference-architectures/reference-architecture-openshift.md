---

copyright:
  years: 2025
lastupdated: "2026-01-12"

keywords: 

subcollection: virtualization-solutions # Use deployable-reference-architectures, or the subcollection value from your toc.yaml file if docs-only.
authors:
- name: Bryan Buckland, Neil Taylor, Sami Kuronen

production: false

---

{{site.data.keyword.attribute-definition-list}}

# Red Hat OpenShift Virtualization
{: #virt-sol-rove-architecture}

Red Hat OpenShift Virtualization on {{site.data.keyword.cloud}} is used to run virtual server workloads alongside containerized applications that are within a unified Kubernetes environment. Red Hat OpenShift Virtualization is based on the KubeVirt Kubernetes operator. Which means that you can deploy both new and existing virtual server workloads on a single, managed platform on {{site.data.keyword.cloud_notm}}.
{: #shortdesc}

Red Hat OpenShift Virtualization servers run on bare metal servers within {{site.data.keyword.cloud_notm}} VPC, which helps provide high performance, security, and network isolation.

**Red Hat OpenShift Data Foundation (ODF)** is software-defined storage that provides highly available, scalable, block, file, and object storage from local NVMe drives that are in the bare metal servers. ODF offers features such as encryption at rest and in transit, snapshots, and disaster recovery replication.

**Red Hat Advanced Cluster Management (RHACM)** provides a centralized control plane for multi-cluster and hybrid management to manage Red Hat OpenShift clusters across on-premises data centers, private clouds, and other public cloud environments. You can deploy RHACM into an {{site.data.keyword.cloud_notm}} ROKS cluster that serves as the hub for orchestrating and governing Red Hat OpenShift deployments across hybrid and multi-cloud landscapes.

## Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}} architecture overview
{: #virt-sol-rove-architecture-diagram}

The following diagram shows the high-level reference architecture for Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}}.

![Red Hat OpenShift Virtualization on IBM Cloud Architecture](../images/openshift/openshift-virtualization-high-level-arch.svg "Red Hat OpenShift Virtualization on IBM Cloud Architecture"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Architecture" caption-side="bottom"}

## Components
{: #virt-sol-rove-components}

The following table outlines the products or services that are used in the architecture for each component.

| Component | Architecture components | Description |
| -------------- | -------------- | -------------- |
| [**Workload migration**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-migration-design) | Red Hat OpenShift Migration toolkit for Virtualization (MTV) | A set of tools to migrate virtual servers from providers such as Red Hat OpenShift and VMware. |
| | IBM Consulting and expert labs | Professional services organizations that provide Red Hat OpenShift services. |
| | Self-service and migration partners | Professional services from migration partners such as WanClouds and Primary IO. |
| [**Resiliency**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-resiliency-design) | Red Hat Advanced Cluster Management (RHACM), OADP, and ODF | RHACM, OADP, and ODF are combined to provide disaster recovery replication of persistent volumes and required cluster resources. |
| | 3rd-party backup options | Self-managed backup options with Red Hat OpenShift Virtualization such as Veeam Kasten K10. |
| [**Observability**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-openshift-observability-design-overview) | Red Hat Advanced Cluster Management (RHACM) | Visibility and control over a hybrid cloud from a single console. |
| | Red Hat OpenShift Observability | Information about the performance and health of Red Hat OpenShift Cluster. |
| | {{site.data.keyword.cloud_notm}} Security and compliance workload protection | Agents that are deployed within virtual servers that provide vulnerability, posture, and compliance scans. |
| | {{site.data.keyword.cloud_notm}} Monitoring and logs | Agents that are deployed within virtual servers that send logs and metrics to {{site.data.keyword.cloud_notm}} logging and monitoring services. |
| [**Storage**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-storage-design-overview) | Red Hat OpenShift Data Foundation (ODF) | Software-defined storage that provides block, file, and object storage. |
| | {{site.data.keyword.cloud_notm}} Object Storage | Designed for unstructured data such as backup, archiving, big data analytics, and application data storage. |
| | {{site.data.keyword.cloud_notm}} File Storage | Persistent, fast, and flexible network-attached, NFS-based File Storage for VPC |
| | {{site.data.keyword.cloud_notm}} Key Protect | Provision and store encrypted keys that are used on ROKS worker nodes and storage. |
| [**Compute**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-compute-design) | Red Hat OpenShift Kubernetes Service worker nodes | Worker nodes can be bare metal or virtual servers. A bare metal is needed to use Red Hat OpenShift Virtualization. |
| | Bare metal and virtual servers | Bare metal servers are recommended to host Red Hat OpenShift Virtualization. Red Hat supports only bare metal servers for production workloads  \n You can use virtual servers for container-based workloads. |
| [**Networking**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-network-design) | Open Virtual Networking (OVN), OVN-Kubernetes | Software-defined networking that is used by Red Hat OpenShift. |
| | Cluster (CUDN) and user-defined networks (UDN) | CUDNs create a network across multiple namespaces.  \n A UDN creates a network within a namespace. |
| | {{site.data.keyword.cloud_notm}} networking | VPC networking, Direct Link, Transit gateways, and VPNs |
| | Virtual Network Functions (VNFs) | Virtual firewalls that run on virtual servers. |
{: caption="Reference Architecture OpenShift Components" caption-side="bottom"}
