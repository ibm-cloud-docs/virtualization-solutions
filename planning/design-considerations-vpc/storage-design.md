---

copyright:
  years: 2025
lastupdated: "2025-12-01"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Storage Design
{: #virt-sol-storage-design-overview}

IBM Cloud offers a range of storage solutions designed to meet diverse workload requirements, from high-performance computing to long-term archival. These options provide flexibility, scalability, and enterprise-grade security for hybrid and multicloud environments.
{: shortdesc}

The key compute architecture elements are shown in the following diagram.

![Red Hat OpenShift Virtualization on IBM Cloud Storage](../../images/vpcvsi/vpcvsi-high-level-storage.svg "Red Hat OpenShift Virtualization on IBM Cloud Storage"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Storage" caption-side="bottom"}



## Storage Options
{: #virt-sol-storage-options}

OpenShift Virtualization utilizes Kubernetes PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) to manage storage for virtual machines. It supports multiple storage backends, including block storage, file storage, and local storage.

Red Hat OpenShift on IBM Cloud offers integrated add-ons for block and file storage by leveraging IBM Cloud resources. These include support for bare metal, file storage, and block storage solutions, enabling flexible and scalable storage options for virtualized workloads. The available storage add-ons are listed below.

### OpenShift Data Foundation (ODF)
{: #virt-sol-storage-odf-summary}

[OpenShift Virtualization]{: tag-red}

OpenShift Data Foundation (ODF) is Red Hat’s integrated storage solution for OpenShift that provides persistent, software-defined storage for containerized applications. It delivers highly available and scalable storage by combining object, block, and file storage under a unified platform. ODF offers features like snapshots, replication, and scalable storage management, all integrated seamlessly with the OpenShift console and APIs, making it easier to manage storage across diverse workloads.

The primary storage option for OpenShift Virtualization is **OpenShift Data Foundation**. It is a highly available storage solution that consists of several open source operators and technologies like **Ceph**, **NooBaa**, and **Rook**. These operators allow you to provision and manage File, Block, and Object storage for your clusters using storage classes. 

ODF abstracts your underlying storage, and you can use ODF to create File, Block, or Object storage claims from the same underlying raw block storage. In virtualization, ODF uses local NVMe disks on VPC BMSs to create a performant virtualized storage layer, where your app data is replicated in multiples of 3 for high availability by default. Using ODF with OpenShift Virtualization is especially critical if you want to use disaster recovery (DR) capabilities for your VM workloads.

For more details of ODF, refer to [Understanding OpenShift Data Foundation](https://cloud.ibm.com/docs/openshift?topic=openshift-ocs-storage-prep)


### IBM Cloud Object Storage 
{: #virt-sol-storage-cos-summary}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

IBM Cloud Object Storage can be used with backup solutions, or other Object Storage related use cases. COS allows backup data to be stored outside of the ODF cluster in case a disaster occurs. 

IBM Cloud Object Storage is a fully managed IBM Cloud service, while object storage built in the ODF clusters is a self managed on ROKS managed worker nodes. Depending on the use case, there might be a use case for either, or even both of these.


### File Storage for VPC
{: #virt-sol-storage-file-summary}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

Customers can use File Storage for VPC, which is a network-attached storage with NFS support. 

IBM Cloud File Storage for VPC is persistent, fast, and flexible network-attached, NFS-based File Storage for VPC that you can add to your apps by using persistent volumes claims (PVCs). You can choose between predefined storage classes that meet the GB sizes and IOPS that meet the requirements of your workloads. 

* All file shares are provisioned with zonal availability, except regional classes which have regional availability.
* All classes are elastic network interface (ENI) enabled.
* All classes support cross-zone mounting.

Regional file share classes can be a better choice for workloads that prioritize durability, availability, or symmetric access across zones over low latency.

* Volume size can range between 1 GiB to 32,000 GiB.
* Volume performance is fixed at 35,000 IOPS.
* The tunable throughput range is 1-8192 Mbps.

See [About File Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about) for the details.

For NFS based file share needs for your workloads, you can also use the NFS storage built in the ODF clusters. The key differences here is that File Storage for VPC is a fully managed IBM Cloud service, while NFS storage built in the ODF clusters is a self managed on ROKS managed worker nodes. Also the IOPS and size settings are independent of your clusters. Depending on the use case, there might be a use case for either, or even both of these. 

For OpenShift Virtualization workloads, keep the following considerations in mind:
- Snapshots are not supported.
- Each Persistent Volume Claim (PVC) created with this provisioner will provision an NFS share and a mount target in your IBM Cloud VPC.
- A PVC can be mounted as a volume in multiple pods or VMs; however, it cannot be shared accross multiple VM deployments.
- For VMs deployment, one VM disk equals one PVC, which means one NFS share per VM disk.

Deploy the File storage shares for VPC add-on on your ROKS cluster by following the instructions in the IBM Cloud documentation [Enabling the IBM Cloud File Storage for VPC cluster add-on](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-install)

The add-on automatically installs the PersistentVolume provisioner `vpc.file.csi.ibm.io` and creates a set of StorageClasses named `ibmc-vpc-file-*`, each offering different IOPS tiers as well as varying reclaim and binding policies. For the full list of available StorageClasses and detailed explanations of their parameters, refer to [Storage class reference](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-sc-ref)

### Block Storage for VPC
{: #virt-sol-storage-block-summary}

[VPC VSI]{: tag-blue} 

This add-on provisions hypervisor-mounted, high-performance, block-level data storage for your worker nodes by using Kubernetes persistent volume claims (PVCs). It enables you to store virtual machine disks on IBM Cloud Block Storage volumes.  

See [About Block Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-about) for details.

### IBM Cloud Key Protect (encryption)
{: #virt-sol-storage-encyrption}

IBM Cloud Key Protect is a cloud-based key management service (KMS) that enables organizations to create, manage, and control encryption keys used to secure data across IBM Cloud services and applications. It provides centralized key lifecycle management with strong security and compliance features, helping businesses meet regulatory requirements and maintain data confidentiality.

[VPC VSI]{: tag-blue} 

Data on a file share is encrypted at rest with IBM-managed encryption by default. You can optionally use your own root keys to protect your file shares with customer-managed keys. 
[About File Storage for VPC > Securing your data](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-data-security)

[OpenShift Virtualization]{: tag-red}

The ODF add-on supports multiple layers of encryption, including:

- Cluster encryption: Encrypts data at rest across the entire storage cluster.
- In-transit encryption: Secures data as it moves between nodes, pods, and clients.
- Storage volume encryption: Provides encryption at the level of individual PersistentVolumes or storage volumes. 

For instructions on setting up storage volume encryption in ROKS, refer to [Setting up encryption by using Hyper Protect Crypto Services](https://cloud.ibm.com/docs/openshift?topic=openshift-deploy-odf-classic&interface=ui#odf-create-hscrypto-classic) for how to set up storage volume encryption
