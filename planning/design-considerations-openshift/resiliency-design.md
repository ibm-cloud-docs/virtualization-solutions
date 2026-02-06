---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-04"

keywords: ROKS, Red Hat OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption, backup, disaster recovery

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Resiliency design for Red Hat OpenShift Virtualization
{: #virt-sol-openshift-resiliency-design}

Red Hat Advanced Cluster Management (RHACM) enables disaster recovery solutions for Red Hat OpenShift Data Foundation clusters. RHACM provides multi-cluster management and application lifecycle orchestration, serving as the control plane in a multi-cluster environment.
{: shortdesc}

See the [Resiliency in IBM Cloud](/docs/resiliency?topic=resiliency-resiliency-overview) Guide that is an overview about resiliency in IBM Cloud. The guide focuses on the perspective of IBM clients, their solution planners, architects, and builders and the resilient solutions that they create on the IBM Cloud platform. The following guide provides specific information for Red Hat Red Hat OpenShift on VPC.

The key backup and restore architecture elements are shown in the following diagram.

![Red Hat Red Hat OpenShift Virtualization on IBM Cloud Backup and Restore](../../images/openshift/openshift-virtualization-high-level-backup.svg "Red Hat Red Hat OpenShift Virtualization on IBM Cloud Backup and Restore"){: caption="Red Hat Red Hat OpenShift Virtualization on IBM Cloud Backup and Restore" caption-side="bottom"}

## Regional disaster recovery
{: #virt-sol-openshift-resiliency-design-rhacm}

Red Hat OpenShift Virtualization uses Red Hat Advanced Cluster Management (RHACM) and ODF Regional Disaster Recovery together for regional disaster recovery.

ODF Regional Disaster Recovery on Red Hat Red Hat OpenShift provides asynchronous data replication between two ROKS clusters that are in different IBM Cloud regions, which helps provide business continuity during regional outages.

ODF Regional Disaster Recovery combines Red Hat Advanced Cluster Management and Red Hat OpenShift Data Foundation components to provide application and data mobility across Red Hat Red Hat OpenShift Container Platform clusters. Red Hat OpenShift Data Foundation provides storage provisioning and management for stateful applications in Red Hat OpenShift Container Platform clusters. ODF is backed by Ceph as the storage provider, with lifecycle management provided by Rook in the ODF component stack. Ceph-CSI is used for provisioning and management of persistent volumes for stateful applications.

Red Hat OpenShift DR provides orchestrators to configure and manage stateful applications across peer Red Hat OpenShift clusters that are managed by RHACM. It offers cloud-native interfaces to orchestrate the lifecycle of an application's state on persistent volumes, which include the following actions:

   * Protecting an application and its state relationship across Red Hat OpenShift clusters
   * Failing over an application and its state to a peer cluster
   * Relocating an application and its state to the previously deployed cluster

Red Hat OpenShift API for Data Protection (OADP) provides backup and restore capabilities for non-PVC cluster resources and application metadata. OADP is the Red Hat operator for Velero, the open source Kubernetes backup tool.

### RHACM and ODF architecture components
{: #virt-sol-openshift-resiliency-design-rhacm-components}

The following table details the architecture components of each solution.

| Architecture component | Description |
| -------------- | -------------- |
| Red Hat Advanced Cluster Management (RHACM) Hub | Components that run on the multi-cluster control plane. |
| Managed clusters | Components that run on the managed clusters. |
{: caption="Red Hat Advanced Cluster Management (RHACM) architecture components" caption-side="bottom"}
{: summary="This table provides architecture components for Red Hat Advanced Cluster Management (RHACM)."}
{: #openshift-rhacm}
{: tab-title="Red Hat Advanced Cluster Management (RHACM)"}
{: tab-group="resilency-architecture-components"}

| Architecture component | Description |
| -------------- | -------------- |
| Red Hat OpenShift Data Foundation | Enables RBD block pools for mirroring across Red Hat OpenShift Data Foundation instances. \n - Mirror specific images within RBD block pools. \n - Provide csi-addons to manage per Persistent Volume Claim (PVC) mirroring|
| Red Hat OpenShift DR | The ODF Multicluster Orchestrator is installed on the multi-cluster control plane (RHACM Hub) to orchestrate configuration and peering of Red Hat OpenShift Data Foundation clusters for metro and regional DR relationships. n\ - Red Hat OpenShift DR Hub Operator - Automatically installs as part of ODF Multicluster Orchestrator to orchestrate failover or relocation of DR-enabled applications. \n - Red Hat OpenShift DR Cluster Operator - Automatically installs on each managed cluster in a metro or regional DR relationship to manage the lifecycle of all PVCs for an application. |
| Red Hat OpenShift API for Data Protection (OADP) | Manages the following items: \n - Kubernetes objects and custom resources (deployments, services, routes, ConfigMaps, secrets). \n - Cluster-scoped resources and namespaced resources. \n - Application metadata and configuration. \n - Resource relationships and dependencies |
{: caption="ODF Regional Disaster Recovery architecture components" caption-side="bottom"}
{: summary="This table provides architecture components for ODF Regional Disaster Recovery."}
{: #openshift-odf}
{: tab-title="ODF Regional Disaster Recovery"}
{: tab-group="resilency-architecture-components"}

OADP works with Red Hat OpenShift DR to provide comprehensive data protection. While Red Hat OpenShift DR provides PVC replication and application mobility between clusters, OADP helps make sure that all supporting Kubernetes resources and configurations are backed up and can be restored. OADP backs up data to S3-compatible object storage such as IBM Cloud Object Storage or NooBaa Multi-Cloud Gateway. NooBaa Multi-Cloud Gateway is included with Red Hat OpenShift Data Foundation, and can use ODF storage or external object storage that enables both cluster-local recovery and cross-cluster disaster recovery scenarios.

For more information, see [Red Hat Red Hat OpenShift on VPC multiregion DR](/docs/pattern-openshift-vpc-dr-multiregion?topic=pattern-openshift-vpc-dr-multiregion-overview).

## IBM Cloud Backup and Recovery
{: #virt-sol-openshift-resiliency-design-bar}

**IBM Cloud Backup and Recovery** is a provider-managed backup service for file, folder, and database servers (MS SQL Server and SAP HANA) in VPC environments. You use this service to define backup schedules that routinely protect data sources by using a secure, agent-based, application-consistent backup service. Backup infrastructure is managed by IBM.

The following are a list of the IBM Cloud Backup and Recovery key capabilities:

* Agent-based backup for virtual server instances
* Support for file-level and folder-level backups
* Integration with IBM Cloud Object Storage for long-term retention
* Scheduled and on-demand backup operations
* Centralized management through IBM Cloud console:
    * Scheduled backups - Customize backup plans to run daily, weekly, or custom times.
    * Policy-based backup - Use policies to define how and when the objects and files in a source are protected. Define parameters such as the data to protect, backup frequency, and how long to retain the backup copy.
    * Security - Take advantage of granular role-based access control to stop unauthorized actors from modifying or deleting data.
    * Application-consistent backup - Consistently capture backups of your application data for a clean restoration to a specific time without data corruption or loss.

### IBM Cloud Backup and Recovery architecture components
{: #virt-sol-openshift-resiliency-ibm-cloud-architecture}

The following table details the architecture components of IBM Cloud Backup and Recovery.

| Architecture component | Description |
| -------------- | -------------- |
| IBM Cloud Backup and Recovery service | Managed by IBM. You can access the backup and recovery service from a web browser to manage your backup policies and download agents and restoration files. |
| VPE Gateway | To improve performance, use a VPE gateway to access the service instead of the native connection. To create one or more VPE gateways use the IBM Cloud catalog to order a VPC gateway and configure it to use the backup and recovery service. |
| Data Source Connector | Install one or more (install at least two for high availability) data connectors and increase as needed to backup throughput. Data source connectors are used to establish connectivity between your source virtual server and the backup and recovery service. The data source connectors also interact with the IBM Cloud Object Storage bucket where the backups are located. This bucket is managed by the provider and is not contained within your account. |
| Agent | IBM Cloud Backup and Recovery software that interacts locally with the operating system and protected source data. The agent communicates with the Data Source Connector and backup and recovery instance during backup and recovery operations. Windows and Linux agents are currently available. |
{: caption="IBM Cloud Backup and Recovery architecture components" caption-side="bottom"}

For more information, see [Getting started with Backup and Recovery](/docs/backup-recovery?topic=backup-recovery-getting-started-backup-recovery).



## Veeam Kasten K10
{: #virt-sol-openshift-resiliency-design-kasten}

Veeam Kasten K10 delivers secure, Kubernetes-native data protection and application mobility at scale across a range of distributions and platforms, including Red Hat Red Hat OpenShift environments. Kasten provides unified backup and recovery for virtual servers that are migrating to and running on Red Hat Red Hat OpenShift Virtualization. Which enable consistent protection for the virtual server data alongside containerized workloads through a single policy engine. Kasten K10 is available from the IBM Cloud catalog tile with a BYOL model.

See the following list of Veeam Kasten K10 key capabilities:

* **Policy-driven automation** - Scalable and consistent protection across environments
* **Application-level consistency** - Deep database integrations for transactional consistency
* **Application mobility** - Seamless migration of data across clouds and environments
* **Immutable backups** - Ransomware protection through immutable backup storage
* **Automated disaster recovery** - Simplified DR orchestration and testing
* **Granular restore capabilities** - File-level, application-level, and namespace-level restore options

**Integration with Veeam Backup and Replication (VBR):**

You can integrate Kasten K10 with Veeam Backup and Replication (VBR) by providing extra recovery granularity beyond normal pod?VM restore functions.

* Granular virtual server file recovery without a full virtual server restoration.
* Integrates with Veeam Explorers for application-specific recovery (SQL Server, Exchange, Active Directory).
* Uses Veeam repositories as destinations for persistent volume snapshot data in compatible environments.

## Red Hat Red Hat OpenShift API for Data Protection (OADP)
{: #virt-sol-openshift-resiliency-design-oadp}

Red Hat Red Hat OpenShift API for Data Protection (OADP) is an operator that provides backup and restore capabilities for Red Hat Red Hat OpenShift cluster resources and application data. OADP is based on the open source Velero project and extends with Red Hat support, extra features, and seamless integration with Red Hat Red Hat OpenShift environments.

OADP enables comprehensive protection for Red Hat OpenShift workloads that include containerized applications, virtual servers that are running on Red Hat Red Hat OpenShift Virtualization, and cluster configuration resources. OADP provides a unified backup solution for both application metadata and persistent data.

The following table details the architecture components of the OADP solution.

| Architecture component | Description |
| -------------- | -------------- |
| OADP Operator | Manages the lifecycle of backup and restore operations within Red Hat Red Hat OpenShift clusters. The operator deploys and configures Velero and associated components. |
| Velero | The core backup engine that handles resource discovery, backup creation, and restore operations. Velero interacts with the Kubernetes API to capture cluster resources and coordinates with storage providers. |
| Restic or Kopia | File-level backup tools that are used for backing up persistent volume data. OADP supports both Restic and Kopia as data movers for PVC backup. Subsequent backups by using Restic or Kopia capture only changed data, reducing storage consumption and backup time. |
| Container Storage Interface (CSI) Snapshots | OADP can use CSI snapshot capabilities for efficient, storage-native snapshots of persistent volumes when supported by the underlying storage provider. |
| Object Storage Backend | OADP requires S3-compatible object storage for storing backups. Supported backends include:  \n - IBM Cloud Object Storage. \n NooBaa Multi-Cloud Gateway (included with Red Hat Red Hat OpenShift Data Foundation, which can use ODF's underlying Ceph storage or act as a gateway to external object storage) |
{: caption="Red Hat Red Hat OpenShift for API Data Protection architecture components" caption-side="bottom"}

The following table details the backup capabilities that OADP provides for Red Hat Red Hat OpenShift environments:

| Backup feature | Description |
| -------------- | -------------- |
| Cluster Resource Backup | - Kubernetes objects and custom resources (Deployments, Services, Routes, ConfigMaps, Secrets) \n - Red Hat OpenShift-specific resources (BuildConfigs, ImageStreams, DeploymentConfigs) \n - Custom Resource Definitions (CRDs) and custom resources \n - Role-Based Access Control (RBAC) configurations \n Network policies and security context constraints |
| Persistent Volume Backup | - File-system-based backup by using Restic or Kopia \n - CSI snapshot-based backup for supported storage providers \n - Volume snapshots with incremental backup capabilities \n - Support for ReadWriteOnce (RWO) and ReadWriteMany (RWX) volumes |
| Red Hat Red Hat OpenShift Virtualization Support | - Virtual server definitions  \n - VM disk data (DataVolumes, PVCs used by VMs) \n VM snapshots and configurations \ Network attachment definitions for VMs \n - VM-specific ConfigMaps and Secrets
| Namespace and Application-Level Backups | - Backup entire namespaces with all contained resources \n - Selective resource backup by using label selectors \n Application-consistent backups with pre-backup and post-backup hooks. \n - Ordered backup of resources with dependencies|
{: caption="OADP backup features for Red Hat OpenShift" caption-side="bottom"}

## Third-party backup options
{: #virt-sol-openshift-resiliency-design-3rd-party}

Third-party backup options provide alternatives for Red Hat Red Hat OpenShift VM backup. The following options are available as self-managed with bring-your-own-license (BYOL) models.

* **Commvault** - Enterprise backup and recovery with application-aware capabilities
* **Rubrik** - Cloud data management and ransomware protection
* **Veritas NetBackup** - Enterprise data protection across hybrid environments
* **Cohesity** - Data management platform with backup, dis, and archival capabilities
* **Veeam** - Enterprise backup and recovery with application-aware capabilities and ransomware protection

These solutions can support both agent-based backup for virtual servers and Kubernetes-native backup for Red Hat OpenShift workloads, providing flexibility that is based on organizational requirements and existing tool investments.

<!--- Caution on the third party information - we don't include third party information in our content. This list is fine, but if you want to provide any further details, it must be a link to the third party documentation. If we don't support it, we don't document it. >
