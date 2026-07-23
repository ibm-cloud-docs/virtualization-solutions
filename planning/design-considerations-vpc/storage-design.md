---

copyright:
  years: 2025, 2026
lastupdated: "2026-07-21"

keywords: VPC storage design, Block Storage for VPC, File Storage for VPC, IBM Cloud Object Storage, VPC storage profiles, IOPS tiers, storage encryption VPC, Key Protect encryption, NFS storage VPC, persistent volumes VPC

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Designing storage for IBM Cloud VPC virtual servers
{: #virt-sol-storage-design-overview}

Design storage for IBM Cloud VPC virtual servers, including Block Storage, File Storage, Object Storage, and Key Protect encryption.
{: shortdesc}

The key storage architecture elements are shown in the following diagram.

![VPC Virtual Servers on IBM Cloud Storage](../../images/vpc-vsi/vpc-vsi-high-level-storage.svg "VPC Virtual Servers on IBM Cloud Storage"){: caption="VPC Virtual Servers on IBM Cloud Storage" caption-side="bottom"}


## Storage options
{: #virt-sol-storage-options}

{{site.data.keyword.redhat_openshift_full}} Virtualization uses Kubernetes PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) to manage storage for virtual servers. It supports multiple storage backends, including block storage, file storage, and local storage.

{{site.data.keyword.redhat_openshift_notm}} on IBM Cloud offers integrated add-ons for block and file storage by using IBM Cloud resources. These include support for bare metal, file storage, and block storage solutions, enabling flexible and scalable storage options for virtualized workloads. The available storage add-ons are listed in the following sections.

### Red Hat OpenShift Data Foundation (ODF)
{: #virt-sol-storage-odf-summary}

Red Hat OpenShift Data Foundation (ODF) is Red Hat's integrated storage solution for Red Hat OpenShift that provides persistent, software-defined storage for containerized applications. It delivers highly available and scalable storage by combining object, block, and file storage under a unified platform. ODF offers features such as snapshots, replication, and scalable storage management, all integrated with the Red Hat OpenShift console and APIs, making it easier to manage storage across diverse workloads.

The primary storage option for Red Hat OpenShift Virtualization is Red Hat OpenShift Data Foundation. It is a highly available storage solution that consists of several open-source operators and technologies such as Ceph, NooBaa, and Rook. These operators are used to provision and manage file, block, and object storage for your clusters by using storage classes.

ODF abstracts your underlying storage, and you can use ODF to create file, block, or object storage claims from the same underlying raw block storage. In virtualization, ODF uses local NVMe disks on VPC bare metal servers to create a performant virtualized storage layer, where your application data is replicated in multiples of 3 for high availability by default. Using ODF with Red Hat OpenShift Virtualization is especially critical if you want to use disaster recovery (DR) capabilities for your virtual server workloads.

For more information about ODF, see [Understanding OpenShift Data Foundation](/docs/openshift?topic=openshift-ocs-storage-prep).

### IBM Cloud Object Storage
{: #virt-sol-storage-cos-summary}

IBM Cloud Object Storage can be used with backup solutions, or other Object Storage related use cases. {{site.data.keyword.cos_full}} allows backup data to be stored outside of the ODF cluster in case a disaster occurs.

IBM Cloud Object Storage is a fully managed IBM Cloud service, while object storage built in the ODF clusters is self-managed on {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service managed worker nodes. Depending on the use case, there might be a use case for either, or even both of these.

### File Storage for VPC
{: #virt-sol-storage-file-summary}

You can use File Storage for VPC, which is a network-attached storage with Network File System (NFS) support.

IBM Cloud File Storage for VPC is persistent, fast, and flexible network-attached, NFS-based File Storage for VPC that you can add to your apps by using persistent volumes claims (PVCs). You can choose between predefined storage classes that meet the GB sizes and input/output operations per second (IOPS) that meet the requirements of your workloads.

* All file shares are provisioned with zonal availability, except regional classes which have regional availability.
* All classes are elastic network interface (ENI) enabled.
* All classes support cross-zone mounting.

Regional file share classes can be a better choice for workloads that prioritize durability, availability, or symmetric access across zones over low latency.

* Volume size can range between 1 GiB to 32,000 GiB.
* Volume performance is fixed at 35,000 IOPS (input/output operations per second).
* The tunable throughput range is 1-8192 Mbps.

See [About File Storage for VPC](/docs/vpc?topic=vpc-file-storage-vpc-about) for the details.

For NFS-based file share needs for your workloads, you can also use the NFS storage built in the ODF clusters. The key differences here are that File Storage for VPC is a fully managed IBM Cloud service, while NFS storage built in the ODF clusters is self-managed on {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service managed worker nodes. Also the IOPS and size settings are independent of your clusters. Depending on the use case, there might be a use case for either, or even both of these.

For OpenShift Virtualization workloads, keep the following considerations in mind:
- Snapshots are not supported.
- Each Persistent Volume Claim (PVC) created with this provisioner will provision an NFS share and a mount target in your IBM Cloud VPC.
- A PVC can be mounted as a volume in multiple pods or virtual servers; however, it cannot be shared across multiple virtual server deployments.
- For virtual server deployment, one virtual server disk equals one PVC, which means one NFS share per virtual server disk.

Deploy the File storage shares for VPC add-on on your {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster by following the instructions in the IBM Cloud documentation [Enabling the IBM Cloud File Storage for VPC cluster add-on](/docs/openshift?topic=openshift-storage-file-vpc-install).

The add-on automatically installs the PersistentVolume provisioner `vpc.file.csi.ibm.io` and creates a set of StorageClasses named `ibmc-vpc-file-*`, each offering different IOPS tiers as well as varying reclaim and binding policies. For the full list of available StorageClasses and detailed explanations of their parameters, see [Storage class reference](/docs/openshift?topic=openshift-storage-file-vpc-sc-ref).

### Block Storage for VPC
{: #virt-sol-storage-block-summary}

This add-on provisions hypervisor-mounted, high-performance, block-level data storage for your worker nodes by using Kubernetes persistent volume claims (PVCs). It enables you to store virtual machine disks on IBM Cloud Block Storage volumes.

See [About Block Storage for VPC](/docs/vpc?topic=vpc-block-storage-about) for details.

### IBM Cloud Key Protect (encryption)
{: #virt-sol-storage-encryption}

IBM Cloud Key Protect is a cloud-based key management service (KMS) that enables organizations to create, manage, and control encryption keys used to secure data across IBM Cloud services and applications. It provides centralized key lifecycle management with strong security and compliance features, helping businesses meet regulatory requirements and maintain data confidentiality.

Data on a file share is encrypted at rest with IBM-managed encryption by default. You can optionally use your own root keys to protect your file shares with customer-managed keys.
[About File Storage for VPC > Securing your data](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-data-security)

The ODF add-on supports multiple layers of encryption, including:

- Cluster encryption: Encrypts data at rest across the entire storage cluster.
- In-transit encryption: Secures data as it moves between nodes, pods, and clients.
- Storage volume encryption: Provides encryption at the level of individual PersistentVolumes or storage volumes.

For instructions on setting up storage volume encryption in {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service, see [Setting up encryption by using Hyper Protect Crypto Services](https://cloud.ibm.com/docs/openshift?topic=openshift-deploy-odf-classic&interface=ui#odf-create-hscrypto-classic).

## Next steps
{: #virt-sol-storage-design-next-steps}

Now that you understand the storage design options for VPC virtual servers, explore these related topics:

- **Security**: Review [encryption and data protection](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-security-design-overview) for storage
- **Resiliency**: Learn about [backup and snapshot strategies](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vpc-resiliency-design) for storage volumes
- **Compute**: Explore [compute design options](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-compute-design-overview) and storage requirements
- **Reference architecture**: Review the complete [VPC virtual server reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vsi-architecture)
