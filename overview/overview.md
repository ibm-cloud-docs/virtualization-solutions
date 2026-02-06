---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-02"

keywords: virtual servers, ROKS, ROVE, VSI, virtualization

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Enable flexible workload deployments with IBM Cloud Virtualization
{: #overview}
{: shortdesc}

Virtualization on {{site.data.keyword.cloud}} provides enterprises with a scalable and secure foundation for running their virtualized workloads in a public cloud environment. By abstracting physical resources into virtualized environments, organizations can optimize their infrastructure usage, reduce operational complexity, and accelerate application delivery.

{{site.data.keyword.cloud_notm}} offers two primary virtualization services.

- **{{site.data.keyword.cloud_notm}} Virtual Servers for VPC** – {{site.data.keyword.cloud_notm}} Virtual Servers for VPC offer fast-provisioning compute capacity with the highest network speeds and most secure software-defined networking resources available on {{site.data.keyword.cloud_notm}}. Built on the {{site.data.keyword.vpc_full}} (VPC), this developer-friendly infrastructure helps drive modern workloads with pre-set instance profiles, rapid deployment, and private network control in an agile public cloud environment. Pay-as-you-use by the hour or reserve your capacity in advance for reduced costs. For more information, see [IBM Cloud Virtual Servers for VPC](https://www.ibm.com/products/virtual-servers).
- **Red Hat OpenShift&reg; Virtualization on {{site.data.keyword.cloud_notm}}** – Red Hat OpenShift on {{site.data.keyword.cloud_notm}} is a fully managed Red Hat OpenShift cloud service that is designed with built-in security to help you efficiently build, deploy, and scale critical applications. The service is highly available and integrated with {{site.data.keyword.cloud_notm}} to bring your team the full power of the platform. See [Red Hat OpenShift on IBM Cloud](https://www.ibm.com/products/openshift?utm_content=SRCWW&p1=Search&p4=932346542397&p5=b&p9=177560641800&gclsrc=aw.ds&gad_source=1&gad_campaignid=22243108467&gbraid=0AAAAA-oKwidudiP1l7zoWnPmSLzCsWbNd&gclid=Cj0KCQiAiqDJBhCXARIsABk2kSkYqiLI03iaLmje-8H2XH-GLoNPTtGhXpv-FkpiynxxQWvj4yOFR58aAhrpEALw_wcB).

## {{site.data.keyword.cloud_notm}} Virtual Servers for VPC
{: #virt-sol-overview-vsi}

[Virtual Servers for VPC]{: tag-blue}

{{site.data.keyword.cloud_notm}} Virtual Servers for VPC provide compute instances within a logically isolated virtual private cloud.

See the following key features of virtual workloads:

- Dedicated or shared virtual server instances
   - Shared tenancy through multi-tenant hosts
   - Dedicated hosts for compliance or licensing needs
- Wide range of compute profiles (balanced, compute optimized, memory optimized)
- Integrated with {{site.data.keyword.cloud_notm}} Virtual Private Cloud for secure isolation.
- Advanced networking features
   - Security groups
   - Floating IPs
   - Load balancers

- High-speed private and public network interfaces.
- Data encryption at rest and in transit
- Integration with {{site.data.keyword.cloud_notm}} services such as file and block storage, object storage, and VPN.
- {{site.data.keyword.cloud_notm}}-provided license or bring-your-own-license (BYOL) support
- Custom images, snapshots, and placement groups

The following table details the high-level, key responsibilities for {{site.data.keyword.cloud_notm}} Virtual Servers for VPC.

| Responsibility |
| -------------- |
| Application deployment and management |
| Guest operating system maintenance and security fixes |
| Security hardening and configuration |
| Network configuration within your VPC |
| Backup and DR implementation |
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

For a comprehensive list of responsibilities, see [Understanding your responsibilities when you use Virtual Private Cloud](/docs/vpc?topic=vpc-responsibilities-vpc).

## Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}}
{: #virt-sol-overview-rove}

[Red Hat OpenShift Virtualization]{: tag-red}

Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}} extends Kubernetes by enabling virtual servers to run alongside containers.

See the following key features of Red Hat OpenShift.

- Unifies infrastructure management by using a single Kubernetes-based platform that orchestrates both virtual servers and containers that eliminate the need for separate virtualization and containerization stacks.
- Enables gradual modernization by migrating existing virtual servers to Red Hat OpenShift while you run them alongside containerized workloads, which supports incremental application refactoring without disruption.
- Provides enterprise orchestration by using Red Hat Advanced Cluster Management for multi-cluster deployments, disaster recovery, and centralized governance across hybrid environments.
- Delivers consistent DevOps workflows by using the same CI/CD pipelines, GitOps practices, and Kubernetes-native tools to both virtual servers and container workloads.
- Integrates with {{site.data.keyword.cloud_notm}} infrastructure, by natively connecting to {{site.data.keyword.cloud_notm}} Block Storage, File Storage, Object Storage, IAM, and VPN services,
- Red Hat OpenShift Data Foundation provides persistent storage for virtual servers with features such as snapshots, cloning, and disaster recovery replication.
- Reduces operational usage as {{site.data.keyword.cloud_notm}} manages the Red Hat OpenShift control plane, worker node maintenance, and platform updates, so you can focus on workload management.

The following table details the high-level, key responsibilities for Red Hat OpenShift Virtualization on {{site.data.keyword.cloud_notm}}.

| Responsibility |
| -------------- |
| Application deployment and management |
| Guest operating system maintenance and security fixes |
| Security hardening and configuration |
| Virtual server networking policy (NetworkAttachmentDefinitions, Services) |
| Backup and disaster recovery implementation |
| Monitoring and logging setup |
| Red Hat OpenShift operators and workload configuration |
| Worker node scaling |
| Red Hat OpenShift Data Foundation scaling |
| Network configuration within VPC |
{: caption="RedHat Openshift virtualization key responsibilities" caption-side="bottom"}
{: tab-title="Your responsibilities"}
{: tab-group="red-hat-openshift-reponsibilities"}
{: class="simple-tab-table"}
{: #simpletabtable1}

| Responsibility |
| -------------- |
| Red Hat OpenShift control plane management and updates |
| Worker node operating system maintenance and security fixes |
| VPC network infrastructure |
| Hardware maintenance |
{: caption="RedHat Openshift virtualization key responsibilities" caption-side="bottom"}
{: tab-title="IBM's responsibilities"}
{: tab-group="red-hat-openshift-reponsibilities"}
{: class="simple-tab-table"}
{: #simpletabtable1}

 For a comprehensive list of responsibilities, see [Red Hat OpenShift Virtualization on IBM Cloud](/docs/openshift?topic=openshift-responsibilities_iks).
