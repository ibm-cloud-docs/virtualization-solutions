---

copyright:
  years: 2025
lastupdated: "2025-11-24"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Storage Design
{: #storage-design}
{: shortdesc}

This document presents a detailed guide to leveraging storage in OpenShift Virtualization for provisioning, managing, and running virtual machines on Red Hat OpenShift on IBM Cloud. It covers the available storage options and explains the workflows involved in using them.


## Storage Options
{: #storage-options}

OpenShift Virtualization utilizes Kubernetes PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) to manage storage for virtual machines. It supports multiple storage backends, including block storage, file storage, and local storage.

Red Hat OpenShift on IBM Cloud offers integrated add-ons for block and file storage by leveraging IBM Cloud resources. These include support for bare metal, file storage, and block storage solutions, enabling flexible and scalable storage options for virtualized workloads. The available storage add-ons are listed below.

### OpenShift Data Foundation (ODF)
{: #odf-summary}

OpenShift Data Foundation (ODF) is Red Hat’s integrated storage solution for OpenShift that provides persistent, software-defined storage for containerized applications. It delivers highly available and scalable storage by combining object, block, and file storage under a unified platform. ODF offers features like snapshots, replication, and scalable storage management, all integrated seamlessly with the OpenShift console and APIs, making it easier to manage storage across diverse workloads.

ODF is natively integrated in Red Hat OpenShift on IBM Cloud (ROKS) as an add-on, providing default management of backend block storage directly from the worker nodes, while also allowing integration with external storage systems.

For more details of ODF, refer to [Understanding OpenShift Data Foundation](https://cloud.ibm.com/docs/openshift?topic=openshift-ocs-storage-prep)

### File Storage for VPC
{: #file-summary}

This add-on provisions persistent, high-performance, and flexible network-attached, NFS-based file storage for worker nodes using Kubernetes PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs). It enables you to store virtual machine disks directly on IBM Cloud File Storage shares, providing scalable capacity that can grow with your workloads. The storage is fully managed, highly available, and optimized for low-latency access, making it suitable for I/O-intensive applications and virtualized environments. See [About File Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about) for the details.

When using this add-on, keep the following considerations in mind:
- Snapshots are not supported.
- Each Persistent Volume Claim (PVC) created with this provisioner will provision an NFS share and a mount target in your IBM Cloud VPC.
- A PVC can be mounted as a volume in multiple pods or VMs; however, it cannot be shared accross multiple VM deployments.
- For VMs deployment, one VM disk equals one PVC, which means one NFS share per VM disk.

### Block Storage for VPC
{: #block-summary}

This add-on provisions hypervisor-mounted, high-performance, block-level data storage for your worker nodes by using Kubernetes persistent volume claims (PVCs). It enables you to store virtual machine disks on IBM Cloud Block Storage volumes.  See [About Block Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-about) for details.


## Use Openshift Data Foundation (ODF)
{: #use-odf}

Deploy the ODF add-on on your ROKS cluster by following the instructions in [Deploying OpenShift Data Foundation on VPC clusters](https://cloud.ibm.com/docs/openshift?topic=openshift-deploy-odf-vpc). Use the provided options to configure your preferred billing model, storage cluster size, encryption settings, and other ODF parameters.

The ODF add-on supports multiple layers of encryption, including:
- Cluster encryption: Encrypts data at rest across the entire storage cluster.
- In-transit encryption: Secures data as it moves between nodes, pods, and clients.
- Storage volume encryption: Provides encryption at the level of individual PersistentVolumes or storage volumes. For instructions on setting up storage volume encryption in ROKS, refer to [Setting up encryption by using Hyper Protect Crypto Services](https://cloud.ibm.com/docs/openshift?topic=openshift-deploy-odf-classic&interface=ui#odf-create-hscrypto-classic) for how to set up storage volumn encryption

The ODF add-on automatically creates a storage system by claiming eligible local disks across all worker nodes in the cluster. It also provisions a block storage pool and a file storage pool, along with the corresponding StorageClasses that you can use for persistent volumes and virtual machine disks.

In OpenShift Data Foundation (ODF), a storage pool is a logical grouping of storage resources used to provide persistent, highly available storage to applications and virtual machines.  ODF typically provisions two main types of storage pools:

- Block Storage Pool – Provides replicated block storage for workloads that require low latency and high performance, such as virtual machine disks. 

- File Storage Pool – Offers shared file-system storage (via CephFS) suitable for applications that need POSIX-compliant file access.

The block storage pool is powered by Ceph RBD (RADOS Block Device), while the file storage pool is backed by CephFS. Both pool types support replication to ensure data durability and high availability by storing multiple copies of data across different worker nodes.

- Replica Count (2-way or 3-way): ODF creates three replicas of each block by default. This means every piece of data is stored on three different ODF storage nodes to protect against node or disk failure.

- Failure Tolerance: With 3-way replication, the storage pool can tolerate the loss of one full node (or disk) without data loss. Ceph automatically re-replicates data to maintain the replication level.

- Synchronous Replication: Writes are committed across all replicas before the operation is acknowledged, ensuring strong consistency for block storage—ideal for virtual machine disks.

- Automatic Healing: If a node becomes unhealthy or a disk fails, Ceph detects the issue and automatically rebalances and re-replicates data to other healthy nodes.


OpenShift Data Foundation (ODF) automatically creates a set of StorageClasses named `ocs-storagecluster-ceph-*` that map to the default Ceph storage pools used for PersistentVolumes. Among these, the `ocs-storagecluster-ceph-rbd-virtualization` StorageClass is the recommended option for virtual machine disks.  

You can create your own storage pools under the default storage system and customize them with different data protection policies, such as 2-way or 3-way replication. You can also enable data compression to improve storage efficiency by reducing the size of replicated data.

Make sure to create a new StorageClass that references the Block storage pool you created, so your virtual machines can use it. When defining this StorageClass, include the parameter mapOptions: 'krbd:rxbounce' to ensure proper support for Windows-based virtual machines.


## Use IBM Cloud File storage shares for VPC
{: #use-nfs}

- Deploy the File storage shares for VPC add-on on your ROKS cluster by following the instructions in the IBM Cloud documentation [Enabling the IBM Cloud File Storage for VPC cluster add-on](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-install)

- The add-on automatically installs the PersistentVolume provisioner `vpc.file.csi.ibm.io` and creates a set of StorageClasses named `ibmc-vpc-file-*`, each offering different IOPS tiers as well as varying reclaim and binding policies. For the full list of available StorageClasses and detailed explanations of their parameters, refer to [Storage class reference](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-sc-ref)

- You can create your own StorageClasses if your virtual machine disks require IOPS levels that differ from those offered by the default StorageClasses installed by the add-on, as long as they are supported by IBM Cloud File Storage for VPC. When defining custom StorageClasses, ensure that you use the `vpc.file.csi.ibm.io` PersistentVolume provisioner. You can configure these StorageClasses using the IBM NFS provisioner (CIS) and match them to the appropriate IBM Cloud File Storage for VPC performance profiles. For details on supported profiles, refer to [File Storage for VPC profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-profiles)

- You can use your existing File Storage shares with this provisioner by manually creating a PVC and a PV. For detailed instructions, refer to [Attaching existing file storage to an app](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-file-vpc-apps#vpc-add-file-static) 

## Use IBM Cloud Block storage for VPC
{: #use-block-storage}

- Deploy the Block storage for VPC add-on on your ROKS cluster by following the instructions in [Setting up Block Storage for VPC](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-block)

- The add-on automatically installs the PersistentVolume provisioner `vpc.block.csi.ibm.io` and creates a set of StorageClasses named `ibmc-block-file-*`, each offering different IOPS tiers as well as varying reclaim and binding policies. For the full list of available StorageClasses and detailed explanations of their parameters, refer to [Block Storage for VPC storage class reference](https://cloud.ibm.com/docs/openshift?topic=openshift-storage-block-vpc-sc-ref)

- You can create your own StorageClasses if your virtual machine disks require IOPS levels that differ from those offered by the default StorageClasses installed by the add-on, as long as they are supported by IBM Cloud File Storage for VPC. When defining custom StorageClasses, ensure that you use the `vpc.block.csi.ibm.io` PersistentVolume provisioner. You can configure these StorageClasses using the IBM NFS provisioner (CIS) and match them to the appropriate IBM Cloud Block Storage for VPC performance profiles. For details on supported profiles, refer to [Block Storage for VPC profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles)


## IBM Cloud Storage References
{: #vpc-storage-references}

This section provides references to IBM Block Storage and File Storage for VPC resources that can be used with OpenShift Virtualization, as described above.

## Bare-metal Storage
{: #bare-metal-storage-ref}

- ROKS clusters running on bare-metal servers contribute their local disks to the ODF storage system, allowing ODF to aggregate and manage them as part of the distributed storage cluster.
- All profiles of Bare Metal Servers for VPC provide one 0.96 TB SATA M.2 mirrored SSD as the boot disk. Profile bx2d-metal-96x384 provides an extra set of NVMe (Non-Volatile Memory Express) U.2 solid-state drives (SSD) as secondary local storage. NVMe SSDs provides fast and affordable storage. 
- See [Storage overview for Bare Metal Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-bare-metal-servers-storage)

## File Storage for VPC
{: #file-storage-ref}

- Provides NFS-based file shares that are made accessible over the network to allow multiple clients to access the same folders and files simultaneously. 
- File Storage for VPC requires NFS versions v4.1 or higher.
- See [About File Storage for VPC > Access protocols](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-allowed-access-protocols)
- Data is available within a zone or within all zones in a region, across multiple VPCs. File Storage with regional availability requires special access.
- See [About File Storage for VPC > Zonal file shares overview](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#zonal-file-storage-overview)
- See [About File Storage for VPC > Regional file shares overview](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#regional-file-storage-overview)
- File Storage performance is bound by Share Profiles, which define the capacity, performance, and data availability characteristics of file shares. Currently, all file shares are created based on the high-performance dp2 profile.
- See [File Storage for VPC profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-profiles&interface=ui)
- Data on a file share is encrypted at rest with IBM-managed encryption by default. You can optionally use your own root keys to protect your file shares with customer-managed keys. 
- See [About File Storage for VPC > Securing your data](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-data-security)
- Create a mount target to obtain a network endpoint and path at which to mount the file share on a VSI. You can attach a VNI to the mount target to make the file share available to all zones in a region.
- See [About File Storage for VPC > Mount targets for file shares](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-share-mount-targets)
- You can create read-only replicas of your file shares in another zone within your VPC, or another zone in a different region if you have multiple VPCs in the same geography. The replica is updated regularly (as often as every 15 minutes) based on the replication schedule that you specify.
- See [About File Storage for VPC > Replication and failover](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-repl-failover-overview)
- Windows operating systems are not supported.
- See [About File Storage for VPC > Limitations in this release](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-limitations)
- There are limits to the number of file shares you can have, their size, their number of connections and when they can be deleted.
- See [About File Storage for VPC > Limitations in this release](https://cloud.ibm.com/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-limitations)

## IBM Cloud Block Storage for VPC
{: #block-storage-ref}

- Used to create primary boot volumes and secondary data volumes.
- Pay for only the capacity that you need.
- Your provisioned storage and associated charges are calculated based on GiB (not GB), where 1 GiB equals 1,073,741,824 bytes.
- See [Block Storage for VPC profiles > Storage capacity](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#note-about-measurement-unit)
- Block Storage performance is bound by Volume Profiles, which define the capacity and performance characteristics of storage volumes.
- See [Block Storage capacity and performance](https://cloud.ibm.com/docs/vpc?topic=vpc-capacity-performance&interface=ui)
- See [Block Storage for VPC profiles > Block Storage profile families](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#block-storage-profile-overview)
- Three Volume Profiles families are available: SDP, tiered and custom.
- SDP profile is a second-generation Volume profile that lets you specify the capacity, IOPS performance and the maximum throughput limit.
- See [Block Storage for VPC profiles > SSD defined performance profile](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#defined-performance-profile)
- Tiered profiles let you select from three predefined IOPS tiers: 3 IOPS/GB, 5 IOPS/GB and 10 IOPS/GB.
- See [Block Storage for VPC profiles > Tiered volume profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#tiers)
- Use custom profiles when you have well-defined performance requirements that do not fall within a predefined IOPS tier.
- See [Block Storage for VPC profiles > Custom volume profiles](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#custom)
- SPD profiles are available in select regions, while Tiered and Custom profiles are available in all regions.
- See [Block Storage for VPC profiles > Block Storage profile families](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#block-storage-profile-overview)
- Network bandwidth allocated to storage is distributed proportionally across each attached Block Storage volume, up to its max throughput limit.
- See [Bandwidth allocation for Block Storage volumes](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-bandwidth&interface=ui)
- The allocation does not change unless a volume is detached or attached to the instance. If you change the storage-networking bandwidth ratio, detach and reattach the data volumes for the new bandwidth allocation to be realized.
- See [Bandwidth allocation for Block Storage volumes](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-bandwidth&interface=ui)
- The provisioned volume throughput limit is determined by the number of IOPS multiplied by the preset throughput multiplier. The preset value is 16 KB for 3 IOPS/GB or 5 IOPS/GB tiered profiles, or 256 KB for 10 IOPS/GB tiered or custom volume profiles. IOPS values vary based on the chosen volume profile.
- See [Block Storage capacity and performance](https://cloud.ibm.com/docs/vpc?topic=vpc-capacity-performance&interface=ui)
- See [Block Storage for VPC profiles > Block Storage profile families](https://cloud.ibm.com/docs/vpc?topic=vpc-block-storage-profiles&interface=ui#block-storage-profile-overview)
