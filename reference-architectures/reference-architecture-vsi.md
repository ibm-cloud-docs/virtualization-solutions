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



# Virtual Servers for VPC Architecture
{: #virt-sol-vpc-vsi-architecture}
{: toc-content-type="reference-architecture"}
{: toc-version="1.0"}

IBM Cloud Virtual Servers for VPC provide compute instances within a logically isolated Virtual Private Cloud environment. This Infrastructure-as-a-Service (IaaS) solution delivers flexible, scalable compute resources with integrated networking, storage, and security capabilities for deploying traditional VM-based workloads on IBM Cloud.

The solution offers a wide range of compute profiles including balanced, compute-optimized, memory-optimized, GPU, and very high memory configurations, running on shared tenancy infrastructure with optional dedicated host placement for compliance and licensing requirements.

Virtual Servers for VPC integrate seamlessly with IBM Cloud VPC networking features including Security Groups, Network ACLs, VPN connectivity, and Load Balancers, providing comprehensive network isolation and security controls. Storage options include boot and data volumes using IBM Cloud Block Storage, with integration to IBM Cloud File Storage and Object Storage for additional data management capabilities.

For enterprise management and governance, IBM Cloud services such as IBM Cloud Monitoring, IBM Cloud Logging, and IBM Cloud Security and Compliance Center provide visibility, audit trails, and compliance scanning across VPC infrastructure.

### Architecture diagram
{: #virt-sol-vpc-vsi-architecture-diagram}

The following diagram shows the high-level reference architecture for Virtual Servers for VPC on IBM Cloud:

[IBM Cloud Virtual Servers for VPC Architecture]

### Components
{: #virt-sol-vpc-vsi-components}

The following table outlines the products or services used in the architecture for each component.

| Component | Architecture components | How the component is used |
| -------------- | -------------- | -------------- |
| **Management and Observability** | IBM Cloud Console / CLI / API | Web-based console, command-line interface, and REST APIs for managing VPC resources |
| | IBM Cloud Monitoring | Agent-based monitoring for metrics collection from virtual server instances |
| | IBM Cloud Logging | Agent-based log aggregation and analysis for virtual server instances |
| | IBM Cloud Activity Tracker | Audit logging for VPC resource management activities |
| | IBM Cloud Security and Compliance Center | Posture management and compliance scanning for VPC infrastructure |
| **Workload Migration** |  IBM Consulting / Expert Labs | Professional services organizations providing migration and deployment services |
| | 3rd Party Migration Tools | Tools in the IBM Cloud Catalog such as RackWare RMM, Wanclouds |
| | Customer Self-Service | Direct migration using image import, instance provisioning, and configuration management tools |
| **Resiliency, backup and disaster recovery** | IBM Cloud Snapshots | Point-in-time copies of Block Storage volumes for backup and recovery |
| | IBM Cloud VPC Backup | Scheduled point-in-time copies of Block Storage volumes for backup and recovery |
| | IBM Cloud Backup and Recovery | Agent-based backup service for file-level and folder-level backup |
| | 3rd Party Backup Solutions | Self-managed backup solutions such as Veeam, Commvault, or Rubrik |
| | Multi-zone deployment | Distribution of instances across availability zones for high availability |
| **Storage** | IBM Cloud Block Storage for VPC | High-performance block storage volumes with configurable IOPS for boot and data disks |
| | IBM Cloud File Storage for VPC | Persistent, fast, and flexible network-attached, NFS-based file storage |
| | IBM Cloud Object Storage | Designed for unstructured data. Ideal for workloads such as backup, archiving, big data analytics, and application data storage |
| | IBM Cloud Key Protect | Provision and store encrypted keys used for volume encryption |
| **Compute** | Virtual Server Instances | Virtual machines with customizable profiles running on shared tenancy infrastructure |
| | Dedicated Hosts | Optional single-tenant physical servers for compliance and licensing requirements |
| | Instance Profiles | Balanced, compute-optimized, memory-optimized, GPU, and very high memory configurations |
| | Custom Images | User-provided OS images for specialized workload requirements |
| **Networking and Interconnectivity** | Virtual Private Cloud (VPC) | Logically isolated network environment with user-defined IP address ranges |
| | Subnets | Network segments within availability zones with configurable routing |
| | Security Groups | Stateful firewalls controlling inbound and outbound traffic at the instance level |
| | Network ACLs | Stateless firewalls controlling traffic at the subnet level |
| | Public Gateways | Enable outbound internet connectivity for instances on private subnets |
| | Floating IPs | Static public IP addresses attachable to instances for inbound internet access |
| | Load Balancers | Application and network load balancers for distributing traffic across instances |
| | VPN Gateways | Site-to-site VPN connectivity to on-premises networks |
| | Transit Gateway | Hub for connecting multiple VPCs and Classic infrastructure |
| | Direct Link | Dedicated, low-latency connections to on-premises data centers |
| | Virtual Network Functions (VNFs) | Virtual firewalls and network appliances running as VPC instances |

## Next steps
{: #next-steps}

_Optional section._ Include links to your deployment guide or next steps to get started with the architecture.

