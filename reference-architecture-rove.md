---

# The YAML header is required. For more information about the YAML header, see
# https://test.cloud.ibm.com/docs-internal/writing?topic=writing-reference-architectures

copyright:
  years: 2025
lastupdated: "2025-11-26"

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

OpenShift Virtualization on IBM Cloud is built on the Red Hat OpenShift Container Platform and its virtualization capabilities. This solution enables organizations to run virtual machine (VM) workloads alongside containerized applications within a unified Kubernetes environment.

The solution is deployed on **IBM Cloud managed Red Hat OpenShift (ROKS)** clusters running on bare metal servers within IBM Cloud VPC, ensuring high performance, security, and network isolation. 

**Red Hat OpenShift Data Foundation (ODF)** is a software-defined storage solution integrated with OpenShift  providing high availability, scalability and data protections, offering features like end-to-end encryption, persistent volume-level encryption, and disaster recovery capabilities.

For multi-cluster and hybrid management, **Red Hat Advanced Cluster Management (RHACM)** provides a centralized control plane to manage OpenShift clusters across on-premises data centers, private clouds, and other public cloud environments. RHACM can be deployed into an IBM Cloud ROKS cluster, serving as the hub for orchestrating and governing OpenShift deployments across hybrid and multi-cloud landscapes.



## Architecture diagram
{: #virt-sol-rove-architecture-diagram}

The following diagram shows the high-level reference architecture for OpenShift Virtualization on IBM Cloud:

![Red Hat Virtualization on IBM Cloud Architecture](images/openshift-virtualization-high-level-arch.svg "Red Hat Virtualization on IBM Cloud Architecture"){: caption="Red Hat Virtualization on IBM Cloud Architecture" caption-side="bottom"}

## Components
{: #virt-sol-rove-components}

The following table outlines the products or services used in the architecture for each components.

| Component | Architecture components | How the component is used |
| -------------- | -------------- | -------------- |
| [**Management and Observability**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-observability-design-overview) | Red Hat Advanced Cluster Management (RHACM) | Visibility and control over hybrid cloud from a single console|
| | Red Hat OpenShift Observability| Insights to performance and health of OpenShift Cluster.|
| | IBM CLoud Security and Compliance Workload Protection |Agents deployed within Virtual Machine, providing vulnerability scanning, posture and compliance scans|
| | IBM Cloud Monitoring and Logs | Agents deployed within Virtual Machine to send logs and monitoring to Cloud Service.|
| **Workload Migration** | Red Hat OpenShift Migration tooklkit for Virtualization (MTV) | A suite of tooling to migrate virtual machines from provides eg. Red Hat OpenShift, VMware |
| | IBM Consulting / Expert Labs | Professional Services organizations that provide Red Hat OpenShift services|
| | Customer Self Server / Migration Partners | Professional Services from migration partners eg. WanClouds / Primary IO|
| | 3rd Party Backup Solutions | Self managed backup solutions with OPenShift Virtualization eg Veeam Kasten|
| [**Resilency, backup and disaster recovery**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-resiliency-design-overview) | Red Hat Advanced Cluster Management (RHACM) and ODF | Combining RHACM and ODF enabled to enable disaster recovery replication of persistent volumes and metadata.|
| | 3rd Party Backup Solutions | Self managed backup solutions with OpenShift Virtualization eg Veeam Kasten|
| [**Storage**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-storage-design-overview) | Red Hat OpenShift Data Foundation (ODF) | Software Defined Storage|
| | IBM Cloud Object Storage | Designed for unstructured data. It is ideal for workloads such as backup, archiving, big data analytics, and application data storage.|
| | IBM Cloud File Storage | Persistent, fast, and flexible network-attached, NFS-based File Storage for VPC|
| | IBM Cloud Key Protect | Provision and store encrypted keys used on ROKS Worker Nodes and Storage.|
| [**Compute**](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-compute-design-overview) | Red Hat OpenShift Kubernetes Service, worker nodes | Worker nodes can be Bare metal or virtual servers. Bare metal is needed for OpenShift Virtualization. | 
| | Bare Metal and Virtual Servers | Bare metal servers are need to host OpenShift Virtualization. \n Virtual servers can be use for Container based workloads.|
| [**Networking and Interconnectivity**](\/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-design-overview) | Open Virtual Networking (OVN), OVN-Kubernetes| Software Defined Networking used by OpenShift|
| | Cluster (CUDN)/ User Defined Networks (UDN) | CUDNs creates a network across multiple namespaces, where as a UDN creates it within a namespace.|
| | IBM Cloud Networking | VPC networking, Direct Link, Transit gateways and VPNs|
| |Virtual Network Functions (VNFs)| Virtual Firewalls running on VPC Virtual Servers|
{: caption="Components" caption-side="bottom"}


## Next steps
{: #virt-sol-rove-next-steps}

_Optional section._ Include links to your deployment guide or next steps to get started with the architecture.
