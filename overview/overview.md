---

copyright:
  years: 2025
lastupdated: "2025-12-09"

keywords: Red Hat OpenShift Virtualization, virtual servers, ROKS, ROVE, VSI, File Storage,

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# IBM Cloud Virtualization: Enabling Flexible Workload Deployments
{: #overview}
{: shortdesc}

Virtualization on {{site.data.keyword.cloud}} provides enterprises with a scalable and secure foundation for running their virtualized workloads in a public cloud environment. By abstracting physical resources into virtualized environments, organizations can optimize their infrastructure utilization, reduce operational complexity, and accelerate application delivery.

{{site.data.keyword.cloud_notm}} offers two primary virtualization services:

- **{{site.data.keyword.cloud_notm}} Virtual Servers for VPC** – {{site.data.keyword.cloud_notm}} Virtual Servers for VPC offer fast-provisioning compute capacity, also known as virtual machines, with the highest network speeds and most secure, software-defined networking resources available on IBM. Built on the {{site.data.keyword.vpc_full}} (VPC), this developer-friendly infrastructure helps drive modern workloads faster and easier with pre-set instance profiles, rapid deployment and private network control in an agile public cloud environment. Pay-as-you-use by the hour or reserve your capacity in advance for reduced costs. For more information, see [IBM Cloud Virtual Servers for VPC](https://www.ibm.com/products/virtual-servers).
- **Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}}** – Red Hat OpenShift on {{site.data.keyword.cloud_notm}} is a fully managed OpenShift® cloud service designed with built-in security to help organizations efficiently build, deploy, and scale critical applications. The service is highly available and intentionally integrated with {{site.data.keyword.cloud_notm}} to bring your team the full power of the platform. See [Red Hat OpenShift on IBM Cloud](https://www.ibm.com/products/openshift?utm_content=SRCWW&p1=Search&p4=932346542397&p5=b&p9=177560641800&gclsrc=aw.ds&gad_source=1&gad_campaignid=22243108467&gbraid=0AAAAA-oKwidudiP1l7zoWnPmSLzCsWbNd&gclid=Cj0KCQiAiqDJBhCXARIsABk2kSkYqiLI03iaLmje-8H2XH-GLoNPTtGhXpv-FkpiynxxQWvj4yOFR58aAhrpEALw_wcB).


## {{site.data.keyword.cloud_notm}} Virtual Servers for VPC
{: #virt-sol-overview-vsi}

[VPC VSI]{: tag-blue}

{{site.data.keyword.cloud_notm}} Virtual Servers for VPC provide compute instances within a logically isolated Virtual Private Cloud.

Key features include:

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
* Integration with {{site.data.keyword.cloud_notm}} services such as File/Block Storage, Object Storage and VPN.
* IBM provided or bring-your-own-license (BYOL) support
* Custom images, snapshots, and placement groups (for availability/spread)

The following table details the high-level, key responsibilities for {{site.data.keyword.cloud_notm}} Virtual Servers for VPC.

| Responsibility |
| -------------- |
| Application deployment and management |
| Guest OS patching and maintenance |
| Security hardening and configuration |
| Network configuration within VPC |
| Backup/DR implementation |
| Monitoring and logging setup |
{: caption="{{site.data.keyword.cloud_notm}} Virtual Servers for VPC key responsibilities" caption-side="bottom"}
{: tab-title="Your responsibilities"}
{: tab-group="vpc-reponsibilities"}
{: class="simple-tab-table"}
{: #simpletabtable1}

| Responsibility |
| -------------- |
| Hypervisor layer |
| VPC network infrastructure |
| Hardware maintenance |
{: caption="{{site.data.keyword.cloud_notm}} Virtual Servers for VPC key responsibilities" caption-side="bottom"}
{: tab-title="IBM's responsibilities"}
{: tab-group="vpc-reponsibilities"}
{: class="simple-tab-table"}
{: #simpletabtable1}


For a comprehensive list of responsibilites, see [Understanding your responsibilities when using Virtual Private Cloud](https://cloud.ibm.com/docs/vpc?topic=vpc-responsibilities-vpc).

## Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}}
{: #virt-sol-overview-rove}

[OpenShift Virtualization]{: tag-red}

Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}} extends Kubernetes by enabling virtual machines to run alongside containers.

Key features include:

* Unifies infrastructure management by using a single Kubernetes-based platform that orchestrates both VMs and containers, eliminating the need for separate virtualization and containerization stacks.
* Enables gradual modernization by migrating existing VMs to OpenShift while running them alongside containerized workloads, supporting incremental application refactoring without disruption.
* Provides enterprise orchestration by leveraging Red Hat Advanced Cluster Management for multi-cluster deployments, disaster recovery, and centralized governance across hybrid environments.
* Delivers consistent DevOps workflows using the same CI/CD pipelines, GitOps practices, and Kubernetes-native tools to both VM and container workloads.
* Integrates with {{site.data.keyword.cloud_notm}} infrastructure, natively connecting to {{site.data.keyword.cloud_notm}} Block Storage, File Storage, Object Storage, IAM, and VPN services,
* OpenShift Data Foundation provides persistent storage for VMs with features like snapshots, cloning, and DR replication.
* Reduces operational overhead as IBM manages the OpenShift control plane, worker node maintenance, and platform updates, letting you focus on workload management.

The following table details the high-level, key responsibilities for Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}}.

| Responsibility |
| -------------- |
| Application deployment and management |
| Guest OS patching and maintenance |
| Security hardening and configuration |
| VM networking policy (NetworkAttachmentDefinitions, Services) |
| Backup/DR implementation |
| Monitoring and logging setup |
| OpenShift operators and workload configuration |
| Worker node scaling |
| ODF scaling |
| Network configuration within VPC |
{: caption="RedHat Openshift virtualization key responsibilities" caption-side="bottom"}
{: tab-title="Your responsibilities"}
{: tab-group="red-hat-openshift-reponsibilities"}
{: class="simple-tab-table"}
{: #simpletabtable1}

| Responsibility|
| -------------- |
| OpenShift control plane management and updates |
| Worker node OS patching and maintenance |
| VPC network infrastructure |
| Hardware maintenance |
{: caption="RedHat Openshift virtualization key responsibilities" caption-side="bottom"}
{: tab-title="IBM's responsibilities"}
{: tab-group="red-hat-openshift-reponsibilities"}
{: class="simple-tab-table"}
{: #simpletabtable1}

 For a comprehensive list of responsibilities, see  [Red Hat OpenShift Virtualization on IBM Cloud](https://cloud.ibm.com/docs/openshift?topic=openshift-responsibilities_iks).
