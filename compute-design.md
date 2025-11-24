---

copyright:
  years: 2025
lastupdated: "2025-11-24"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Compute options for IBM ROKS
{: #compute-design-roks}

Virtual Machine / Compute for ROKS. Currently only supported in VPC BareMetal Gen2 hosts.

-   Supported on Gen 2 BareMetal profiles only Gen3 Support will be in a future release. Profile arability varies by location. See [Profile Availability](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-flavors#us-south-physical-table) for details.

-   Minimum of 3 similar hosts required per availability zone.
-   Hosts can operate at any VPC network speed 10Gb - 200Gb. For optimal results 50G or higher is recommended.
-   Host will use NVME drives for workload storage using Openshift ODF which is based on Ceph. Hosts need 4 or more NVME drives for ODF. Hosts with NFS storage for primary workloads are not supported currently. VPC block storage is not supported on baremetal hosts currently either.
-   Hosts will also only support a single VNIC / Server Network attachment. This will prevent the use of Public Address ranges, Vlan VNI and other newer VPC network features. VNI attachments will be supported in a future release. VNI support will allow for better segregation of workload and storage traffic, as well support for direct connection into a VPC subnet.
-   Clusters need to be dedicated to VM or Pod deployments. Mixed usage is not recommended by Redhat or IBM.
-   ~~IBM also recommends using IBM Fusion for cluster backups to S3, NFS or 3rd party systems. See [IBM Fusion](https://www.ibm.com/docs/en/storage-fusion/storage/2.6.0?topic=fusion-installing-storage-cloud) features for more information .~~
-   ROKS / Openshift Compute Supports many of the same features as VMWare vSphere as well as some new options. One of the new features for VM deployment is templates with predefined “T-Shirt” sizes (Small, Medium, Large, etc.). The sizes can be customized to fit user’s needs. IE Small + 200G of extra storage. VM sizes can also be arbitrary in size (6 cores, 10G ram, 5 disks, more than one network)

-   ROKS networks can also contain both POD and VM deployments as well as the reverse POD and VMS can attach to both L2 and Masquerade networks. When using OVN networking VMs can attach to more than one network.

-   VMs can have more than one drive attachment as well as access to shared storage. Drives can be resized on the fly, provided the VM’s Operating system supports it.

-   ROKS fully support Redhat’s Forlift migration tool allowing for migrations from vSphere to ROKS. However, at this time Forklift does not support migrations from VMWare’s vCloud Director. However, for single tenant setups IBM can provide customers direct access to their vSphere setup. For multi-tenant users automated support will be available in a future release, MT users will need assistance from IBM for access.

## IBM Cloud VPC
{: #vpc-compute}

- Compute is offered via Virtual Servers Instances (VSIs).
- Compute resources (CPU and RAM) of a VSI are specified in terms of profiles. 
- A profile is a combination of instance attributes, such as the number of vCPUs, amount of RAM, network bandwidth, and default bandwidth allocation.
- Profiles are available for both x86-64 (Intel or AMD) and s390x (IBM Z or LinuxONE) CPU architectures.
- Profiles are grouped into families, with each family being optimized for a specific use case.
- Available profiles differ depending on whether or not you are running on dedicated hosts.
- See [x86-64 instance profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-profiles&interface=ui) for the list of available x86-64 profiles.
- See [s390x instance profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-vs-profiles&interface=ui) for the list of available s390x profiles.
- See [x86-64 dedicated host profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-dh-profiles&interface=ui) for the list of available dedicated x86-64 profiles.
- See [s390x dedicated host profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-s390x-dh-profiles&interface=ui) for the list of available dedicated s390x profiles.
- Profile names encode information about the family, architecture, generation and compute and storage resources offered by the profile.
- See [Understanding the naming rule of x86-64 profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-profiles&interface=ui#profiles-naming-rule)
- See [Understanding the naming rule of s390x profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-vs-profiles&interface=ui#vs-profiles-naming-rule)
- See [Understanding dedicated x86-64 profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-dh-profiles&interface=ui#dh-profiles-naming-rule)
- See [Understanding dedicated s390x profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-s390x-dh-profiles&interface=ui#s390x-dh-profiles-naming-rule)
- You can specify optional user data as either a cloud-init config or a script to perform common configuration tasks when creating a new VSI.
- See [User data](https://cloud.ibm.com/docs/vpc?topic=vpc-user-data&interface=ui)
- You can start a VSI in secure boot mode to verify the digital signatures of all code in the boot process and sure that your server starts with trusted software.
- See [Secure boot for Virtual Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-confidential-computing-with-secure-boot-vpc&interface=ui)
- Use confidential computing profiles to ensure that your VSIs and their data are confidential and protected from everyone including the Cloud Service Provider (CSP).
- See [Confidential computing for x86 Virtual Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-confidential-computing-vpc&interface=ui)
- See [Confidential computing with LinuxONE](https://cloud.ibm.com/docs/vpc?topic=vpc-about-se&interface=ui)
- Compute costs are accrued while your VSI is powered on. You can stop billing when you power off the instance.
- See [Suspend billing for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-suspend-billing&interface=ui)
- Compute costs can be lowered by reserving your resources in advance. You can choose a 1 or 3-year term, server quantity, specific profile, and provision those servers when needed.
- See [About Reservations for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-reserved-virtual-servers-vpc&interface=ui)
- Your VSI will be restarted automatically if the host its running on fails unexpectedly. Detection of a host issue occurs within 30 seconds of occurrence. The VSI will be automatically restarted on a healthy host. You can control whether or not to restart yor VSI on host failure.
- See [Host failure recovery policies](https://cloud.ibm.com/docs/vpc?topic=vpc-host-failure-recovery-policies&interface=ui)
