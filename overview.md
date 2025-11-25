---

copyright:
  years: 2025
lastupdated: "2025-11-25"

keywords: Red Hat OpenShift Virtualization, virtual servers, ROKS, ROVE, VSI, File Storage, 

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# IBM Cloud Virtualization: Enabling Flexible Workload Deployments
{: #overview}
{: shortdesc}

Virtualization on IBM Cloud provides enterprises with a scalable and secure foundation for running both traditional and cloud-native workloads. By abstracting physical resources into virtualized environments, organizations can optimize infrastructure utilization, reduce operational complexity, and accelerate application delivery.

IBM Cloud offers two primary virtualization options:

1. **Virtual Servers for VPC** – Ideal for traditional workloads requiring isolated compute resources. 
2. **Red Hat OpenShift Virtualization** – Designed for hybrid environments where virtual machines and containers coexist seamlessly.[OpenShift Virtualization]{: tag-red}


## Virtual Servers for VPC
{: #virt-sol-overview-vsi}
[VPC VSI]{: tag-blue}

Virtual Servers for VPC provide compute instances within a logically isolated Virtual Private Cloud. Key features include:

* Dedicated or Shared Instances for performance and cost optimization.
* Wide range of compute profiles (balanced, compute optimized, memory-optimized)
* Integrated with Virtual Private Cloud (VPC) for secure isolation.
    * Advanced networking features:
        * Security Groups
        * Floating IPs
        * Load Balancers
* High-speed private and public network interfaces.
* Data encryption at rest and in transit
* Integration with IBM Cloud services such as File/Block Storage, Object Storage and VPN.
* IBM provided or bring-your-own-license (BYOL) support

## Red Hat OpenShift Virtualization
{: #virt-sol-overview-rove}
[OpenShift Virtualization]{: tag-red}

OpenShift Virtualization extends Kubernetes by enabling virtual machines to run alongside containers. This approach:

* Unifies VM and container management under a single orchestration layer.
* Enables hybrid workloads and multi-cluster deployments with Red Hat Advanced Cluster Management.
* Easily migrate existing VMs to OpenShift Virtualization to run alongside containers, enabling a gradual, low-risk modernization journey
* Integration with IBM Cloud services such as File/Block Storage, Object Storage and VPN.
* Provides consistent CI/CD pipelines for both VM and containerized applications.
