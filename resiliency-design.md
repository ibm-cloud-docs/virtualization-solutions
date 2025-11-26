---

copyright:
  years: 2025
lastupdated: "2025-11-26"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Backup and Restore Solutions
{: #virt-sol-resiliency-design-overview}

Data protection is a critical component of any cloud strategy, ensuring business continuity and compliance in the event of failures, disasters, or cyber incidents. IBM Cloud provides a range of backup and recovery services designed to safeguard workloads running on virtual servers, bare metal, and containerized environments. These services enable automated backups, secure storage, and rapid recovery to minimize downtime and data loss.

By combining IBM Cloud’s built-in capabilities with third-party solutions, enterprises can achieve flexible, secure, and compliant backup and recovery architectures tailored to their operational and regulatory requirements.

The key compute architecture elements are shown in the following diagram.

![Red Hat Virtualization on IBM Cloud Backup and Restore](images/openshift-virtualization-high-level-backup.svg "Red Hat Virtualization on IBM Cloud Backup and Restore"){: caption="Red Hat Virtualization on IBM Cloud Backup and Restore" caption-side="bottom"}


## Regional Disaster Recovery
{: #virt-sol-resiliency-design-red-hat}

[OpenShift Virtualization]{: tag-red}

You can use **Red Hat Advanced cluster management (RHACM)** to set up the Disaster Recovery solutions for ODF clusters. ODF Regional Disaster Recovery (async) on Red Hat OpenShift can be used to setup a DR solution between two ROKS clusters located in two IBM Cloud regions. Regional DR ensures business continuity during the unavailability of a geographical region.

ODF Regional DR is composed of Red Hat Advanced Cluster Management and OpenShift Data Foundation components to provide application and data mobility across Red Hat OpenShift Container Platform clusters.

**Red Hat Advanced Cluster Management** provides the ability to manage multiple clusters and application lifecycles. Hence, it serves as a control plane in a multi-cluster environment. RHACM is split into two parts. **RHACM Hub** includes components that run on the multi-cluster control plane. **Managed clusters** include components that run on the clusters that are managed.

**OpenShift Data Foundation** provides the ability to provision and manage storage for stateful applications in an OpenShift Container Platform cluster. OpenShift Data Foundation is backed by Ceph as the storage provider, whose lifecycle is managed by Rook in the OpenShift Data Foundation component stack. Ceph-CSI provides the provisioning and management of Persistent Volumes for stateful applications.

OpenShift Data Foundation stack includes the following capabilities for disaster recovery:

* Enable RBD block pools for mirroring across OpenShift Data Foundation instances (clusters)
* Ability to mirror specific images within an RBD block pool
* Provides csi-addons to manage per Persistent Volume Claim (PVC) mirroring

**OpenShift DR** is a set of orchestrators to configure and manage stateful applications across a set of peer OpenShift clusters which are managed using RHACM and provides cloud-native interfaces to orchestrate the life-cycle of an application’s state on Persistent Volumes. These include:

* Protecting an application and its state relationship across OpenShift clusters
* Failing over an application and its state to a peer cluster
* Relocate an application and its state to the previously deployed cluster

OpenShift DR is split into three components. **ODF Multicluster Orchestrator** is installed on the multi-cluster control plane (RHACM Hub), it orchestrates configuration and peering of OpenShift Data Foundation clusters for Metro and Regional DR relationships. **OpenShift DR Hub Operator** is automatically installed as part of ODF Multicluster Orchestrator installation on the hub cluster to orchestrate failover or relocation of DR enabled applications. **OpenShift DR Cluster Operator** is automatically installed on each managed cluster that is part of a Metro and Regional DR relationship to manage the lifecycle of all PVCs of an application.



## Backup System Concepts and Features
{: #virt-sol-resiliency-design-concepts}

There is basic concept terminology used when describing backup, restore and replication system capabilities. A basic understanding of what each term means will provide better understanding of a backup system's capabilities. 

### Backup 3-2-1 Rule
{: #virt-sol-resiliency-design-321}

In backups the 3-2-1 best practice rule is a data protection strategy that requires three copies of your data, with two copies stored on different media and at least one copy kept off-site. 

This means having your primary running application, a local snapshot, and an external copy, ensuring redundancy and protection against hardware failure, software glitches, or site-wide disasters.  

- 3 Copies of Data: The original application data on the Persistent Volume Claim (PVC), a local snapshot copy, and an exported copy in an external location.
- 2 Different Media Types: The original data on your primary storage (e.g., block storage for PVCs) and backup copies stored on a different type of media, such as object storage like IBM Cloud Object Storage.
- 1 Copy Off-site: The exported copy should be in a separate physical location or a different cloud region/account to protect against site-specific disasters.  

When implementing your backup and restore solution you should make sure that you have a way to satisfy the 3-2-1 rule, and that you can run a disaster recovery test scenario and it is successful.

### Incremental Backups
{: #virt-sol-resiliency-design-incremental}

An incremental backup is a data protection strategy that only copies the data that has changed since the most recent previous full or incremental backup.
This method is designed to optimize backup operations by minimizing storage consumption and reducing the time and network bandwidth required for each backup, which is especially beneficial for large, dynamic environments. 

Incremental backups can be achieved through multiple mechanisms:
- Changed Block Tracking (CBT) : Works at the storage block level. It tracks which data blocks  have been altered since a previous snapshot or backup.
- Container Storage Interface (CSI) Snapshots : OpenShift uses the Kubernetes CSI standard for managing storage. CSI drivers can create snapshots of Persistent Volumes (PVs). The incremental aspect comes from managing a chain of snapshots. Subsequent backups compare the current state to the previous snapshot and copy only the changed data to an object storage repository.
- Restic : Restic is an open source program that operates at the file level. It reads the data from the PV (typically by mounting it to a temporary data mover pod) and backs up new or modified files or portions of files to an S3-compatible object storage

### Deduplication and Compression 
{: #virt-sol-resiliency-design-depdup}

Deduplication and compression are data efficiency techniques primarily handled by the underlying storage solution or third-party backup software. Combining compression and deduplication reduces storage requirements.

- Deduplication : Identify and store only one unique instance of a file or data block. Use a pointer or reference to that single stored copy everywhere else it is referenced.
- Compression : The process of reducing the size of individual data blocks or files by rewriting them using less physical space.

### Application Aware and Crash Consistent Backups
{: #virt-sol-resiliency-design-app-aware}

Application-aware backups ensure data integrity by synchronizing applications and data before a backup is taken, while crash-consistent backups simply take a point-in-time snapshot without this synchronization.
- Application-aware backups : A backup tool interacts with the application to quiesce operations before the snapshot is taken. This ensures all data and transactions are safely written to the disk. Necessary for stateful applications, especially databases, where data integrity is critical.
- Crash Consistent backup : A snapshot is taken of the Persistent Volumes (PVs) with no regard to the applications running on them. It does not account for data in memory or pending I/O transactions.  It may require manual application recovery steps to fix inconsistencies after a restore.

### Local and Remote Backups
{: #virt-sol-resiliency-design-local-and-remote-backup}

References to local and remote backups is distinguishing between the physical storage location of the backup files, snapshots, and data.
- Local backups : Involve storing the backup data on a storage device within the same physical site or on-premises environment as the OpenShift cluster. Local backup data may be vulnerable to the same issues that may affect the OpenShift cluster. Restoring from local data may be quicker as the transfer is over a high-speed local network, minimizing downtime.
- Remote (external) backups : Involve sending copies of the data to a secure, separate location, typically to a cloud-based object storage service such as IBM Cloud Object Storage. Remote data would be protected from any local issues affecting the OpenShift cluster. Restoration of the local copy would depend on internet bandwidth and latency, which can lead to longer recovery times.   


## Restore Operation Concepts and Features
{: #virt-sol-resiliency-design-restore-sys}

Defining restore operation concepts and capabilities is important for a better understanding of the possibilities when recovering a system or data.

### File/Folder Level Granularity
{: #virt-sol-resiliency-design-restore-file}

Granular Recovery Technology (GRT) is a general term for data recovery approaches that do not require a full system restore. It is the ability to restore individual files and folders within a persistent volume (PV), rather than requiring the backup or restoration of the entire volume.  

### Application Level Granularity
{: #virt-sol-resiliency-design-restore-app}

To achieve application level granularity the tooling must understand the application's data structure and performing the backup and restore in a way that maintains the application integrity. When you restore using application level granularity you do not have to restore the whole namespace or cluster.

### Namespace Level Granularity
{: #virt-sol-resiliency-design-restore-namespace}

Namespace level granularity means that you can restore a whole project, or "namespace", that contains applications and their associated resources, including application data and configuration. A comprehensive namespace backup includes both the Kubernetes/OpenShift metadata (such as deployments, services, routes, secrets, config maps) and the actual application data stored in PVs.

### Restore from Local or Remote Backup
{: #virt-sol-resiliency-design-restore-backup}

The options to restore from local or remote refer to the physical or network location where the backup data is stored and from which it will be retrieved during a restore operation. Local restore usually has better performance than a remote restore. The 3-2-1 best practice combines local and remote storage.

A local restore retrieves data from a storage device located within the same physical site or local network as the OpenShift cluster. The local data could be an attached storage device (external hard disks, USB drives) or Network Attached Storage (NAS) devices within the same local network.   
A remote restore retrieves data from an off-site location, typically a cloud storage service (like IBM Cloud Object Storage) or a dedicated remote data center.


## What are the backup and restore needs for OpenShift Virtualization
{: ##virt-sol-resiliency-design-rove}

[OpenShift Virtualization]{: tag-red}

Backups are essential in OpenShift environments to ensure business continuity. A robust backup and recovery strategy safeguards both critical application data and the cluster's control plane operational metadata.  

Whether you're managing **Kubernetes containers** or **traditional virtual machines (VMs) in OpenShift Virtualization**, ensuring data integrity and availability is crucial. Implementing an effective backup and restore strategy is key to minimizing downtime and safeguarding critical data.

Kubernetes offers powerful mechanisms for managing containers, but it doesn't natively provide backup and restore capabilities. Backing up Kubernetes containers involves ensuring that both the application state (running containers, pods, deployments) and persistent data (volumes, databases) are properly saved.  

In order to adequately backup a Kubernetes environment you must be able to collect and protect both the cluster state and the application data. Backup solutions for Kubernetes will include the ability to handle:  

1. **Stateful Data**: While containers are ephemeral and can be easily recreated, one of the challenges lies in preserving the environments stateful data (persistent volumes). Kubernetes provides Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) for this purpose. **Velero** is an open-source backup/restore tool that is integrated with OpenShift that you can use to automate the backup of both container configurations and persistent volumes.

2. **Cluster State**: Backing up Kubernetes cluster configurations, including custom resources, namespaces, and deployments, is essential for disaster recovery. Tools like **Velero** can capture snapshots of cluster state and restore them in the event of failure, ensuring applications and services are quickly brought back online.

3. **Restore**: Restoring from a backup involves recreating the cluster environment and redeploying containers. For persistent data, volumes can be restored from their snapshots, while Kubernetes configurations can be reapplied using YAML files or backup software.

OpenShift, the enterprise Kubernetes platform from Red Hat, introduces a hybrid model that allows you to run both containers and traditional VMs in the same environment through **OpenShift Virtualization**. This capability brings together the best of both worlds, enabling businesses to run modern containerized applications alongside legacy VM-based workloads. OpenShift backup solutions must be able to handle:  

1. **Containerized Workloads**: An OpenShift Backup and restore solution must handle containerized workloads. Kubernetes-native tools such as **Velero** can be used to backup and restore both persistent volumes and cluster configurations.

2. **VM Workloads**: OpenShift Virtualization allows for the management of VMs within an OpenShift cluster, which can be backed up using VM backup tools that integrate with OpenShift, such as **Red Hat OpenShift APIs for Data Protection (OADP)**. This solution ensures that VMs running inside OpenShift are properly captured in backups, similar to how traditional VM solutions work.

3. **Unified Restore**: A unified restore process for OpenShift Virtualization allows administrators to restore both VMs and containers in the same workflow. This makes disaster recovery simpler and more consistent for environments running both containerized and VM-based applications.



## IBM Cloud Backup as a Service - Agent based
{: #virt-sol-resiliency-design-restore-brs-agent}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

**IBM Cloud Backup and Recovery** is a fully managed service that provides backup solutions for various IBM Cloud services and customer workloads running on IBM Cloud. 

To register your data sources with the IBM Cloud Backup and Recovery service, you need to establish connectivity between your source and the service using a Data Source Connection. A Data Source Connection consists of one or more Data Source Connectors, which are virtual machines (VMs) that facilitate the movement of data between your data sources and the IBM Cloud Backup and Recovery service.

An agent is IBM Cloud Backup and Recovery software installed on physical appliances, VMs or VSIs that interacts locally with the OS / Source data being protected and with the Data Source Connector / Backup and Recovery instance during backup and recovery operations. Windows and Linux agents are currently available with support for additional agent types available in the future.


## 3rd Party Backup Solutions
{: #virt-sol-resiliency-design-3rd-party}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

Various 3rd party backup solutions are alternatives (such as Veeam Kasten) as roll-your-own and bring-your-own-license models.

Veeam Kasten delivers secure, Kubernetes-native data protection and application mobility at scale and across a wide range of distributions and platforms, such as OpenShift environment. Kasten provides unified backup and recovery for virtual machines moving to and running on OpenShift Virtualization, enabling consistent protection for VM data alongside containerized workloads through a single policy engine. Kasten is available as an IBM Cloud tile with a BYOL model.

Key features include policy-driven automation for scalable and consistent protection, application-level consistency with deep database integrations, and seamless application mobility to migrate data across clouds and environments. It also offers robust features like immutable backups for ransomware protection, automated disaster recovery (DR), and granular restore capabilities.  

Kasten can be integrated with Veeam Backup and Replication (VBR) which provides additional recovery granularity that goes beyond the normal pod restore functionality. This allows for granular VM file recovery without full VM restore and integration with Veeam explorers for application-specific recovery.
A Veeam Repository may be used as the destination for persistent volume snapshot data in compatible environments.

