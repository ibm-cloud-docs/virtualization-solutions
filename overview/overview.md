---

copyright:
  years: 2025
lastupdated: "2025-12-04"

keywords: Red Hat OpenShift Virtualization, virtual servers, ROKS, ROVE, VSI, File Storage, 

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# IBM Cloud Virtualization: Enabling Flexible Workload Deployments
{: #overview}
{: shortdesc}

Virtualization on IBM Cloud® provides enterprises with a scalable and secure foundation for running their virtualized workloads in a public cloud environment. By abstracting physical resources into virtualized environments, organizations can optimize their infrastructure utilization, reduce operational complexity, and accelerate application delivery.

IBM Cloud offers two primary virtualization services:

1. **IBM Cloud Virtual Servers for VPC** – IBM Cloud Virtual Servers for VPC offer fast-provisioning compute capacity, also known as virtual machines, with the highest network speeds and most secure, software-defined networking resources available on IBM. Built on the IBM Cloud Virtual Private Cloud (VPC), this developer-friendly infrastructure helps drive modern workloads faster and easier with pre-set instance profiles, rapid deployment and private network control in an agile public cloud environment. Pay-as-you-use by the hour or reserve your capacity in advance for reduced costs. See [IBM Cloud Virtual Servers for VPC](https://www.ibm.com/products/virtual-servers).
2. **Red Hat OpenShift Virtualization on IBM Cloud** – Red Hat OpenShift on IBM Cloud is a fully managed OpenShift® cloud service designed with built-in security to help organizations efficiently build, deploy, and scale critical applications. The service is highly available and intentionally integrated with IBM Cloud to bring your team the full power of the platform. See [Red Hat OpenShift on IBM Cloud](https://www.ibm.com/products/openshift?utm_content=SRCWW&p1=Search&p4=932346542397&p5=b&p9=177560641800&gclsrc=aw.ds&gad_source=1&gad_campaignid=22243108467&gbraid=0AAAAA-oKwidudiP1l7zoWnPmSLzCsWbNd&gclid=Cj0KCQiAiqDJBhCXARIsABk2kSkYqiLI03iaLmje-8H2XH-GLoNPTtGhXpv-FkpiynxxQWvj4yOFR58aAhrpEALw_wcB).


## IBM Cloud Virtual Servers for VPC
{: #virt-sol-overview-vsi}

[VPC VSI]{: tag-blue}

IBM Cloud Virtual Servers for VPC provide compute instances within a logically isolated Virtual Private Cloud. Key features include:

* Dedicated or Shared virtual server instances:
    * Shared tenancy - using multi-tenant hosts
    * Dedicated hosts - for compliance or licensing needs
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
* Custom images, snapshots, and placement groups (for availability/spread)

At a high level, the key responsibilities for IBM Cloud Virtual Servers for VPC are outlined here. For a comprehensive list, refer to the detailed documentation. [Understanding your responsibilities when using Virtual Private Cloud](https://cloud.ibm.com/docs/vpc?topic=vpc-responsibilities-vpc)

* Your responsibility:
    * Application deployment and management
    * Guest OS patching and maintenance
    * Security hardening and configuration
    * Network configuration within VPC
    * Backup/DR implementation
    * Monitoring and logging setup
* IBM's responsibility:
    * Hypervisor layer
    * VPC network infrastructure
    * Hardware maintenance

## Red Hat OpenShift Virtualization on IBM Cloud
{: #virt-sol-overview-rove}

[OpenShift Virtualization]{: tag-red}

Red Hat OpenShift Virtualization on IBM Cloud extends Kubernetes by enabling virtual machines to run alongside containers. This approach:

* Unifies VM and container management under a single orchestration layer.
* Enables hybrid workloads and multi-cluster deployments with Red Hat Advanced Cluster Management.
* Easily migrate existing VMs to OpenShift Virtualization to run alongside containers, enabling a gradual, low-risk modernization journey
* Integration with IBM Cloud services such as File/Block Storage, Object Storage and VPN.
* Provides consistent CI/CD pipelines for both VM and containerized applications.

Red Hat OpenShift Virtualization on IBM Cloud extends Kubernetes by enabling virtual machines to run alongside containers. This approach:

* Unifies infrastructure management by using a single Kubernetes-based platform that orchestrates both VMs and containers, eliminating the need for separate virtualization and containerization stacks.
* Enables gradual modernization by migrating existing VMs to OpenShift while running them alongside containerized workloads, supporting incremental application refactoring without disruption.
* Provides enterprise orchestration by leveraging Red Hat Advanced Cluster Management for multi-cluster deployments, disaster recovery, and centralized governance across hybrid environments.
* Delivers consistent DevOps workflows using the same CI/CD pipelines, GitOps practices, and Kubernetes-native tools to both VM and container workloads.
* Integrates with IBM Cloud infrastructure, natively connecting to IBM Cloud Block Storage, File Storage, Object Storage, IAM, and VPN services,
* OpenShift Data Foundation provides persistent storage for VMs with features like snapshots, cloning, and DR replication.
* Reduces operational overhead as IBM manages the OpenShift control plane, worker node maintenance, and platform updates, letting you focus on workload management.

At a high-level, key responsibilities for Red Hat OpenShift Virtualization on IBM Cloud are listed below, for a detailed list see [Red Hat OpenShift Virtualization on IBM Cloud](https://cloud.ibm.com/docs/openshift?topic=openshift-responsibilities_iks):

* Your responsibility:
    * Application deployment and management
    * Guest OS patching and maintenance
    * Security hardening and configuration
    * VM networking policy (NetworkAttachmentDefinitions, Services)
    * Backup/DR implementation
    * Monitoring and logging setup
    * OpenShift operators and workload configuration
    * Worker node scaling
    * ODF scaling
    * Network configuration within VPC
* IBM's responsibility:
    * OpenShift control plane management and updates
    * Worker node OS patching and maintenance
    * VPC network infrastructure
    * Hardware maintenance
