---

copyright:
  years: 2025
lastupdated: "2025-12-04"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Storage Design
{: #virt-sol-openshift-storage-design-overview}

IBM Cloud offers a range of storage solutions designed to meet diverse workload requirements, from high-performance computing to long-term archival. These options provide flexibility, scalability, and enterprise-grade security for hybrid and multicloud environments.
{: shortdesc}

The key compute architecture elements are shown in the following diagram.

![Red Hat OpenShift Virtualization on IBM Cloud Storage](../../images/openshift/openshift-virtualization-high-level-storage.svg "Red Hat OpenShift Virtualization on IBM Cloud Storage"){: caption="Red Hat OpenShift Virtualization on IBM Cloud Storage" caption-side="bottom"}

## Storage Options
{: #virt-sol-openshift-storage-options}

[OpenShift Virtualization]{: tag-red}

OpenShift Virtualization utilizes Kubernetes PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) to manage storage for virtual machines. It supports multiple storage backends, including block storage (not available with bare metal servers), file storage, and local storage (only available with bare metal servers).

Red Hat OpenShift on IBM Cloud offers integrated add-ons for OpenShift Data Foundation (ODF), block and file storage by leveraging IBM Cloud resources.

### OpenShift Data Foundation (ODF)
{: #virt-sol-openshift-storage-odf-summary}

OpenShift Data Foundation (ODF) is Red Hat’s integrated storage solution for OpenShift that provides persistent, software-defined storage for containerized applications. It delivers highly available and scalable storage by combining object, block, and file storage under a unified platform. ODF offers features like snapshots, replication, and scalable storage management, all integrated seamlessly with the OpenShift console and APIs, making it easier to manage storage across diverse workloads.

The primary storage option for OpenShift Virtualization is **OpenShift Data Foundation**. It is a highly available storage solution that consists of several open source operators and technologies like **Ceph**, **NooBaa**, and **Rook**. These operators allow you to provision and manage File, Block, and Object storage for your clusters using storage classes. 

ODF abstracts your underlying storage, and you can use ODF to create File, Block, or Object storage claims from the same underlying raw block storage. In virtualization, ODF uses local NVMe disks on VPC bare metal servers to create a performant virtualized storage layer, where your application data is replicated in multiples of typically 3 for high availability by default. Using ODF with OpenShift Virtualization is especially critical if you want to use disaster recovery (DR) capabilities for your VM workloads.

For more details of ODF, refer to [Understanding OpenShift Data Foundation](https://cloud.ibm.com/docs/openshift?topic=openshift-ocs-storage-prep)

The ODF add-on supports multiple layers of encryption, including:

- Cluster encryption: Encrypts data at rest across the entire storage cluster.
- In-transit encryption: Secures data as it moves between nodes, pods, and clients.
- Storage volume encryption: Provides encryption at the level of individual PersistentVolumes or storage volumes. 

For instructions on setting up storage volume encryption in ROKS, refer to [Setting up encryption by using Hyper Protect Crypto Services](https://cloud.ibm.com/docs/openshift?topic=openshift-deploy-odf-classic&interface=ui#odf-create-hscrypto-classic) for how to set up storage volume encryption

### IBM Cloud Object Storage 
{: #virt-sol-openshift-storage-cos-summary}

IBM Cloud Object Storage can be used with backup solutions, or other Object Storage related use cases. COS allows backup data to be stored outside of the ODF cluster in case a disaster occurs. 

IBM Cloud Object Storage is a fully managed IBM Cloud service, while object storage built in the ODF clusters is a self managed on ROKS managed worker nodes. Depending on the use case, there might be a use case for either, or even both of these.

### File Storage for VPC
{: #virt-sol-openshift-storage-file-summary}

Customers can use File Storage for VPC, which is a network-attached storage with NFS support. 

IBM Cloud File Storage for VPC is persistent, fast, and flexible network-attached, NFS-based File Storage for VPC that you can add to your apps by using persistent volumes claims (PVCs). You can choose between predefined storage classes that provide the required capacity in GB and IOPS to meet your workload requirements.

* All file shares are provisioned with zonal availability.
* All classes support cross-zone mounting.

Data on a file share is encrypted at rest with IBM-managed encryption by default. You can optionally use your own root keys to protect your file shares with customer-managed keys. 
[About File Storage for VPC > Securing your data](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-data-security)

See [About File Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about) for the details.

For NFS based file share needs for your workloads, you can also use the NFS storage built in the ODF clusters. The key differences here is that File Storage for VPC is a fully managed IBM Cloud service, while NFS storage built in the ODF clusters is a self managed on ROKS managed worker nodes. The IOPS and GB settings are independent of your clusters. Depending on the use case, there might be a use case for either, or even both of these. 

For OpenShift Virtualization workloads using VPC File storage, keep the following considerations in mind:

- Snapshots are not supported.
- Each Persistent Volume Claim (PVC) created with this provisioner will provision an NFS share and a mount target in your IBM Cloud VPC.
- A PVC can be mounted as a volume in multiple pods or VMs; however, it cannot be shared across multiple VM deployments.
- For VMs deployment, one VM disk equals one PVC, which means one NFS share per VM disk.

Deploy the File storage shares for VPC add-on on your ROKS cluster by following the instructions in the IBM Cloud documentation [Enabling the IBM Cloud File Storage for VPC cluster add-on](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-install)

The add-on automatically installs the PersistentVolume provisioner `vpc.file.csi.ibm.io` and creates a set of StorageClasses named `ibmc-vpc-file-*`, each offering different IOPS tiers as well as varying reclaim and binding policies. For the full list of available StorageClasses and detailed explanations of their parameters, refer to [Storage class reference](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-sc-ref)

### Block Storage for VPC
{: #virt-sol-openshift-storage-block-summary}

Block storage for VPC is only available for VSI worker nodes. 
{: note}

This add-on provisions hypervisor-mounted, high-performance, block-level data storage for your VSI worker nodes by using Kubernetes persistent volume claims (PVCs). It enables you to store virtual machine disks on IBM Cloud Block Storage volumes.

Data on a block volume is encrypted at rest with IBM-managed encryption by default. You can optionally use your own root keys to protect your file shares with customer-managed keys. 
[About Block Storage for VPC > Securing your data](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-vpc-about&interface=ui#fs-data-security)

See [About Block Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-about) for details.
