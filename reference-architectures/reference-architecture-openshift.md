---

# The YAML header is required. For more information about the YAML header, see
# https://test.cloud.ibm.com/docs-internal/writing?topic=writing-reference-architectures

copyright:
  years: 2025
lastupdated: "2025-12-01"

keywords: # Not typically populated

subcollection: virtualization-solutions # Use deployable-reference-architectures, or the subcollection value from your toc.yaml file if docs-only.

authors:
  - name: "Bryan Buckland"
    url: "https://www.linkedin.com/in/bryan-buckland/"
  - name: "name"
    url: "linkedIn profile URL"

# The release that the reference architecture describes
version: 1.0

related_links:
  - title: 'Title'
    url: 'https://url.com'
    description: 'Description.'
  - title: 'related or follow-on architectures'
    url: 'https://url'
    description: 'Description'



content-type: reference-architecture


# For reference architectures in https://github.com/terraform-ibm-modules only.
# All reference architectures stored in the /reference-architectures directory

# Set production to true to publish the reference architecture to IBM Cloud docs.

production: false

---


{{site.data.keyword.attribute-definition-list}}



# OpenShift Virtualization Architecture
{: #virt-sol-rove-architecture}
{: toc-content-type="reference-architecture"}
{: toc-version="1.0"}

Red Hat OpenShift Virtualization on IBM Cloud enables organizations to run virtual machine (VM) workloads alongside containerized applications within a unified Kubernetes environment. The virtualization solution, based on the KubeVirt Kubernetes operator available for Red Hat OpenShift, allows you to run and deploy both new and existing VM workloads on a single, managed platform on IBM Cloud. It aims to ease VM migration, simplify your operations, accelerate time to value, add flexibility and optimize TCO.

The solution leverages worker nodes running on bare metal servers within IBM Cloud VPC, ensuring high performance, security, and network isolation. 

**Red Hat OpenShift Data Foundation (ODF)** is a software-defined storage solution integrated with OpenShift, that provides highly available, scalable, block, file and object storage from local NVMe drives in the bare metal servers. ODF offers features like encryption at rest and in transit, snapshots and replication for disaster recovery capabilities.

For multi-cluster and hybrid management, **Red Hat Advanced Cluster Management (RHACM)** provides a centralized control plane to manage OpenShift clusters across on-premises data centers, private clouds, and other public cloud environments. RHACM can be deployed into an IBM Cloud ROKS cluster, serving as the hub for orchestrating and governing OpenShift deployments across hybrid and multi-cloud landscapes.

## Architecture diagram
{: #virt-sol-rove-architecture-diagram}

The following diagram shows the high-level reference architecture for OpenShift Virtualization on IBM Cloud:

![Red Hat OpenShift Virtualization on IBM Cloud Architecture](../images/openshift/openshift-virtualization-high-level-arch.svg "Red Hat OpenShift Virtualization on IBM Cloud Architecture"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Architecture" caption-side="bottom"}

## Components
{: #virt-sol-rove-components}

The following table outlines the products or services used in the architecture for each components.

| Component | Architecture components | How the component is used |
| -------------- | -------------- | -------------- |
| **Workload Migration** | Red Hat OpenShift Migration toolkit for Virtualization (MTV) | A suite of tooling to migrate virtual machines from providers eg. Red Hat OpenShift, VMware |
| | IBM Consulting / Expert Labs | Professional Services organizations that provide Red Hat OpenShift services|
| | Customer Self Server / Migration Partners | Professional Services from migration partners eg. WanClouds / Primary IO|
| [**Resiliency, backup and disaster recovery**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-resiliency-design-overview) | Red Hat Advanced Cluster Management (RHACM), OADP and ODF | Combining RHACM, OADP and ODF enables disaster recovery replication of persistent volumes and required cluster resources|
| | 3rd Party Backup Solutions | Self managed backup solutions with OpenShift Virtualization eg Veeam Kasten K10|
| [**Management and Observability**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-observability-design-overview) | Red Hat Advanced Cluster Management (RHACM) | Visibility and control over hybrid cloud from a single console|
| | Red Hat OpenShift Observability| Insights to performance and health of OpenShift Cluster.|
| | IBM Cloud Security and Compliance Workload Protection |Agents deployed within Virtual Machine, providing vulnerability scanning, posture and compliance scans|
| | IBM Cloud Monitoring and Logs | Agents deployed within Virtual Machines send logs and metrics to IBM Cloud Logging and Monitoring services.|
| [**Storage**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-storage-design-overview) | Red Hat OpenShift Data Foundation (ODF) | Software Defined Storage that provides block, file and object storage|
| | IBM Cloud Object Storage | Designed for unstructured data. It is ideal for workloads such as backup, archiving, big data analytics, and application data storage.|
| | IBM Cloud File Storage | Persistent, fast, and flexible network-attached, NFS-based File Storage for VPC|
| | IBM Cloud Key Protect | Provision and store encrypted keys used on ROKS Worker Nodes and Storage.|
| [**Compute**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-compute-design-overview) | Red Hat OpenShift Kubernetes Service, worker nodes | Worker nodes can be Bare metal or virtual servers. Bare metal is needed for OpenShift Virtualization. | 
| | Bare Metal and Virtual Servers | Bare metal servers are recommended to host OpenShift Virtualization. Red Hat only supports bare metal servers for production workloads \n Virtual servers can be use for Container based workloads.|
| [**Networking and Interconnectivity**](\/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-design-overview) | Open Virtual Networking (OVN), OVN-Kubernetes| Software Defined Networking used by OpenShift|
| | Cluster (CUDN)/ User Defined Networks (UDN) | CUDNs creates a network across multiple namespaces, where as a UDN creates it within a namespace.|
| | IBM Cloud Networking | VPC networking, Direct Link, Transit gateways and VPNs|
| |Virtual Network Functions (VNFs)| Virtual Firewalls running on VPC Virtual Servers|
{: caption="Reference Architecture OpenShift Components" caption-side="bottom"}


## Next steps
{: #virt-sol-rove-next-steps}

_Optional section._ Include links to your deployment guide or next steps to get started with the architecture.
