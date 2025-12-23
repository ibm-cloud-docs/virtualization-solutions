---

copyright:
  years: 2025
lastupdated: "2025-12-23"

keywords: File Storage, Block Storage, Encryption, backup, disaster recovery

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Resiliency Design for VPC virtual servers
{: #virt-sol-vpc-vpc-resiliency-design}

See the [Resiliency in IBM Cloud](https://cloud.ibm.com/docs/resiliency?topic=resiliency-resiliency-overview) Solution Guide which is general guide on resiliency in IBM Cloud. The guide focuses on the perspective of IBM clients, their solution planners, architects, and builders and the resilient solutions that they create on the IBM Cloud platform. This guide focuses on specific information for VPC VSIs.

The key backup and restore architecture elements are shown in the following diagram.

![IBM Cloud VPC VSI Backup and Restore](../../images/vpc-vsi/vpc-vsi-high-level-resiliency.svg "IBM Cloud VPC VSI Backup and Restore"){: caption="IBM Cloud VPC VSI Backup and Restore" caption-side="bottom"}


## IBM Cloud VPC Block Storage Snapshots
{: #virt-sol-vpc-vpc-resiliency-design-vpc-snap}

IBM Cloud VPC Block Storage Snapshots provide point-in-time copies of Block Storage volumes attached to virtual server instances. Snapshots are stored regionally in IBM Cloud Object Storage and can be used for data protection, disaster recovery, and creating new volumes from a known good state.

The following tables lists the key capabilities for IBM Cloud VPC Block Storage Snapshots.

| Feature | Description |
| -------------- | -------------- |
| Fast snapshot creation | Snapshots are created quickly using copy-on-write technology without impacting volume performance |
| Space-efficient storage | Only changed blocks are stored after the initial full snapshot, minimizing storage costs |
| Regional availability | Snapshots are stored within the same region as the source volume and can be used to create volumes in any zone within that region |
| Bootable volume support | Snapshots of boot volumes can be used to create new virtual server instances with identical configurations |
| Consistency groups | Create crash-consistent snapshots across multiple volumes attached to the same instance (available for certain configurations) |
| Fast restore snapshot clones | By keeping a clone of the data in a zone within your VPC region and not in a separate regional storage repository, this feature can achieve a recovery time objective (RTO) quicker than restoring from a regular snapshot |
| Cross-regional snapshot copies | Copies a snapshot from one region to another region. This feature can be used in disaster recovery scenarios when you need to start your virtual server instance and data volumes in a different region. The snapshot is created as normal, and when the snapshot is stable, a copy of the snapshot is created in the regional storage repository in the target region. When the snapshot copy in the remote region is stable, you can use and manage it independently from the parent volume or the original snapshot. The creation of the copy in the remote region takes time, for example, the creation of a full snapshot of a 3 TB volume in a remote region can take up to 12.5 hours. |
{: caption="IBM Cloud VPC Block Storage Snapshots features" caption-side="bottom"}

The use cases applicable to these features include the following.

* Pre-change backups before system updates or configuration changes
* Creating golden images for rapid virtual server deployment
* Cross-region disaster recovery using snapshot copies
* Development and test environment provisioning from production snapshots
* Multi-volume application-consistent backups using consistency groups

## IBM Cloud VPC Block Storage Snapshots Limitations
{: #virt-sol-vpc-vpc-resiliency-design-vpc-snap-limitations}

* Cumulative size of all snapshots for a volume cannot exceed 10 TB
* Creating crash-consistent snapshots of multiple volumes leads to short-lived I/O suspension that can last from a few milliseconds to a few seconds, depending on the size and quantity of volumes
* No application-aware quiescing (snapshots are crash-consistent)
* Individual snapshot management (not policy-driven without Backup for VPC service)

For more information, see [About Block Storage for VPC snapshots](https://cloud.ibm.com/docs/vpc?topic=vpc-snapshots-vpc-about&interface=ui).

## IBM Cloud Backup for VPC
{: #virt-sol-vpc-vpc-resiliency-design-vpc-bu}

IBM Cloud Backup for VPC provides a policy-driven approach to snapshot lifecycle management, allowing automated backup of VPC Block Storage volumes with configurable schedules and retention policies.

The following table details each backup policy component for IBM Cloud Backup for VPC.

| Backup policy component | Description |
| -------------- | -------------- |
| Backup plan | Defines the cron-based schedule and retention rules for backups |
| Backup policy | Container for one or more backup plans with target resource selection via user tags |
| Backup jobs | Automated execution of snapshot operations based on defined schedules |
{: caption="IBM Cloud Backup for VPC backup policy components" caption-side="bottom"}

The following tables lists the key capabilities for IBM Cloud Backup for VPC.

| Feature | Description |
| -------------- | -------------- |
| Backup policies | Create backup policies with up to four plans to automate backups on daily, weekly, monthly, or yearly schedules |
| Automated retention management | Configure retention of backups based on age or total count, with automatic deletion of expired backups |
| Tag-based automation | Target Block Storage volumes for backup using tags configured in backup policy that match user-provided tags on volumes |
| Consistency group support | Automate multi-volume snapshot consistency groups for crash-consistent backups across multiple volumes |
| Cross-region snapshot copies | Integrate with cross-region snapshot copy feature for geographic disaster recovery |
| Centralized management | Manage backup policies and monitor backup status through IBM Cloud console, CLI, API, or Terraform |
{: caption="IBM Cloud Backup for VPC features" caption-side="bottom"}

The use cases applicable to these features include the following.

* Automated daily/weekly/monthly backups for production virtual server instances
* Compliance and regulatory data retention requirements
* Operational recovery from accidental deletion or corruption
* Multi-volume application backups using consistency groups
* Geographic disaster recovery with cross-region snapshot copies

**Comparison with Manual Snapshots:**

| Feature | Manual Snapshots | Backup for VPC |
|---------|------------------|----------------|
| **Automation** | Manual or external scripting | Policy-based automation |
| **Scheduling** | External orchestration required | Built-in cron-based scheduling |
| **Retention management** | Manual deletion | Automatic retention policy enforcement |
| **Volume targeting** | Direct volume selection | Tag-based automatic targeting |
| **Use case** | Ad-hoc backups, golden images | Ongoing operational backups |
| **Management** | Per-snapshot management | Policy-driven centralized management |
{: caption="Comparison with Manual Snapshots" caption-side="bottom"}

The following list is the best practice for IBM Cloud Backup for VPC.

* Schedule automated backup policies during off-peak hours to minimize performance impact from I/O suspension
* Use consistent tagging strategy across volumes to simplify backup policy application
* Combine Backup for VPC with cross-region snapshot copies for comprehensive disaster recovery
* Monitor backup job status and configure alerts for failed backup operations
* Test restore procedures regularly to validate recovery time objectives (RTO)
* Consider combining with IBM Cloud Backup and Recovery for application-aware backups and file-level recovery

See [About Backup for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-backup-service-about&interface=ui)

## IBM Cloud Backup and Recovery
{: #virt-sol-vpc-vpc-resiliency-design-bar}

**IBM Cloud Backup and Recovery** is a provider managed backup service for file, folder and database servers (MS SQL Server and SAP HANA) in VPC environments running on IBM Cloud. This service lets you define backup schedules to routinely protect data sources using a secure, agent-based, application-consistent backup service. Backup infrastructure is managed by IBM. The service is comprised of:

| Service | Description |
| -------------- | -------------- |
| IBM Cloud Backup and Recovery service | Managed by IBM, once provisioned via the IBM Cloud catalog, you access via a web browser to manage your backup policies, download the agents and restore. |
| VPE Gateway | To improve performance it is recommended to use a VPE gateway to access the service instead of the native connection. To create one or more VPE gateways use the IBM Cloud catalog to order a VPC gateway and configure it to use the Backup and Recovery service. |
| Data Source Connector | Installed via the IBM Cloud Catalog which install a VSI in your VPC. Install one or more (at least two recommended for HA) data connectors and increase as need to increase backup throughput. Data source connectors are used to establish connectivity between your source VSI and the service. The data source connectors also interacts with the service's IBM Cloud Object Service bucket where the backups are located. This bucket is managed by the provider and is not contained within your account. |
| Agent | An agent is IBM Cloud Backup and Recovery software installed on the VSI that interacts locally with the operating system and source data being protected. The agent communicates with the Data Source Connector and Backup and Recovery instance during backup and recovery operations. Windows and Linux agents are currently available, with support for additional agent types planned for the future. |
{: caption="IBM Cloud Backup and Recovery services" caption-side="bottom"}

The following list is the key capabilities for IBM Cloud Backup and Recovery.

* Agent-based backup for virtual server instances
* Support for file-level and folder-level backups
* Integration with IBM Cloud Object Storage for long-term retention
* Scheduled and on-demand backup operations
* Centralized management through IBM Cloud console:
    * Scheduled backups - Customize backup plans to run at daily, weekly or custom intervals
    * Policy-based backup - Use policies to define how and when the objects and files in a source are protected based on your use case. Define parameters such as the data to be protected, backup frequency, and how long to retain the backup copy
    * Security - Take advantage of granular role-based access control to stop unauthorized actors from modifying or deleting data
    * Application-consistent backup - Capture backups of your application data in a consistent state, allowing for clean restoration to a specific point in time without data corruption or loss.

For more information, see [Getting started with Backup and Recovery](https://cloud.ibm.com/docs/backup-recovery?topic=backup-recovery-getting-started-backup-recovery)



## Veeam Backup & Replication with Agent-Based Backup
{: #virt-sol-vpc-vpc-resiliency-design-veeam-vbr}

Veeam Backup & Replication (VBR) is an enterprise-grade backup and disaster recovery solution that provides comprehensive data protection for physical servers, virtual machines, and cloud workloads. For IBM Cloud VPC Virtual Server Instances, Veeam utilizes agent-based backup to provide application-aware, image-level backup and recovery capabilities.

Veeam Backup & Replication with agents delivers centralized management, flexible recovery options, and advanced data protection features including immutable backups, ransomware protection, and cloud-native integration with IBM Cloud Object Storage.

Veeam is not available directly in the IBM Cloud catalog, however, you can use Veeam software to back up your data on a VPC VSI and protect the following resources:

* Individual volumes
* Folders and files
* Veeam Plug-Ins for Enterprise Applications further enhance Veeam Backup & Replication by enabling transactionally consistent backups of SAP HANA, Oracle, and Microsoft SQL Server databases

| Service | Description |
| -------------- | -------------- |
| Veeam Licenses | You can order a Veeam license for the use of Veeam Agent and Veeam Backup and Replication software through the Veeam website or through the process described at [Ordering Veeam stand-alone licenses from the IBM Cloud console](https://cloud.ibm.com/docs/vpc?topic=vpc-ordering-veeam-licenses#ordering-veeam-license-procedure). |
| Veeam Backup Server | The core component that serves as the configuration and control center for the entire backup infrastructure. The backup server manages job scheduling, resource allocation, and centralized administration of all backup operations.  \n The Veeam Backup and Replication software can be installed only on a Microsoft Windows operating system. See [Installing and operating the Veeam Backup and Replication software](https://cloud.ibm.com/docs/vpc?topic=vpc-using-veeam-backup-replication-software). |
| Veeam Agents | Lightweight software installed on protected computers that perform data backup operations such as creating volume snapshots, reading backed-up data, and transferring data to target locations. Supported Linux® distributions include CentOS, RHEL, Ubuntu, and Debian. With the Veeam Agent for Linux® and the Veeam Agent for Microsoft™ Windows™ you can create backups and perform restores, see [Installing and operating the Veeam Agent](https://cloud.ibm.com/docs/vpc?topic=vpc-using-veeam-agent). |
{: caption="Veeam services" caption-side="bottom"}

For more information, see [About Veeam](https://cloud.ibm.com/docs/vpc?topic=vpc-about-veeam).

**Protection Groups** - Containers in the Veeam Backup & Replication inventory that specify computers on which Veeam Agents should be installed and managed, with selection based on individual IP addresses, Active Directory objects, or CSV files.

**Backup Repository** - Storage location where backup files are stored. Repositories can be:

* VPC Block storage
* IBM Cloud Object Storage (S3-compatible)

**Veeam features** - Veeam Backup & Replication offers automated deployment and management of Veeam Agents, allowing administrators to perform deployment, administration, data protection, and disaster recovery tasks remotely from the Veeam Backup & Replication console without installing and configuring agents on every computer individually.

| Feature | Description |
| -------------- | -------------- |
| Centralized Deployment | - Automatic agent discovery and deployment to VPC virtual server instances  \n - Manual deployment using installation packages for restricted environments  \n - Pre-installed agent management for third-party deployment tools  \n Remote upgrade and patch management for deployed agents |
| Centralized Job Management | = Agent backup jobs run on the backup server, via a schedule, allocating infrastructure resources, and managing job execution  \n - Single backup job can process multiple protection groups and individual computers  \n - Backup policies for scheduling agent jobs directly on protected computers  \n - Unified console for monitoring all backup operations |
| Centralized Backup Management | - Restore data from agent backups through the Veeam console  \n - Copy backups to secondary repositories for 3-2-1 compliance  \n - Export backups to standalone files for archival  \n - Import existing agent backups into Veeam infrastructure |
| Application-Aware Processing | - For Windows-based computers, Veeam Agent leverages Microsoft VSS technology to create VSS snapshots for transactionally consistent backups  \n - Support for VSS-aware applications including Microsoft SQL Server, Exchange, Active Directory, and Oracle databases  \n - For Linux systems, support for Oracle, MySQL, and PostgreSQL database processing to create transactionally consistent backups  \n - Application item-level restore (database, mailbox, Active Directory objects) |
| Advanced Data Protection | - Immutable, direct-to-object storage backups that naturally scale with needs  \n - Inline malware detection during backup operations  \n - Encryption in-flight and at-rest with AES-256  \n - Built-in deduplication and compression to reduce backup file sizes and data traffic  \n - WAN acceleration for remote site backups |
| Flexible Backup Targets | - Local backup repositories (fast local recovery)  \n - IBM Cloud Object Storage for long-term retention and offsite protection  \n - Copy jobs to create secondary backup copies (3-2-1 rule compliance) |
| Granular Recovery | - File-level restore from image-level backups without full system restore. Note image-level means the OS image, not the VSI image.  \n - Application item-level restore including databases, mailboxes, and specific application objects  \n - Volume-level restore for partial system recovery  \n - Restore to original or alternate locations |
{: caption="Veeam features" caption-side="bottom"}

## Third-Party Backup Solutions
{: #virt-sol-vpc-vpc-resiliency-design-3rd-party}

Various third-party backup solutions provide alternatives for VPC VSIs backup, available as self-managed with bring-your-own-license (BYOL) models. Additional third-party backup solutions compatible with IBM Cloud VPC include:

* **Commvault** - Enterprise backup and recovery with application-aware capabilities
* **Rubrik** - Cloud data management and ransomware protection
* **Veritas NetBackup** - Enterprise data protection across hybrid environments
* **Cohesity** - Data management platform with backup, DR, and archival capabilities

These solutions typically support both agent-based backup for virtual server instances, providing flexibility based on organizational requirements and existing tool investments.

## Wanclouds VPC+ DRaaS (Disaster Recovery as a Service)
{: #virt-sol-vpc-component-design-wanclouds-draas}

Wanclouds VPC+ DRaaS is a comprehensive SaaS-based Disaster Recovery as a Service (DRaaS) solution that enables IBM Cloud customers to backup their entire Virtual Private Cloud resources including network, compute, and storage, and restore them across different regions in IBM Cloud. This approach eliminates the need for expensive standby environments, replacing them with flexible on-demand recovery. The service is available directly from IBM Cloud Catalog. Key differentiators:

* **On-Demand Recovery Model**: Instead of maintaining a constantly running replica of your production environment, you can create an on-demand disaster recovery scenario, minimizing costs and maximizing operational efficiency
* **Single Pane of Glass**: Consolidated view of all cloud accounts and backups (VPC configs, VSIs, Data, COS buckets) under a single management interface

Wanclouds VPC+ DRaaS can backup and restore the entire IBM Cloud Virtual Private Cloud construct, configurations, and resources including:

| Component | Description |
| -------------- | -------------- |
| Network Components | - VLANs and Subnets  \n - Security Groups and Network ACLs  \n - VPN Gateways  \n - Load Balancers (ALB and NLB)  \n - Floating IPs  \n - Public Gateways  \n - Virtual Private Endpoints (VPE)
| Compute Resources | - Virtual Server Instances (VSIs) - backup different versions of Windows and Linux OS flavors including Ubuntu, Red Hat Enterprise Linux (RHEL), Debian, and CentOS  \n - Attached Data Volumes (Block Storage)  \n - Instance configurations and metadata |
| Storage | - Cloud Object Storage Buckets (COS Buckets) - backup data from one COS bucket to other buckets on demand  \n - Volume attachments and configurations |
{: caption="Wanclouds VPC+ DRaaS components" caption-side="bottom"}

Wanclouds VPC+ DRaaS provides flexible restore options:

| Restore option | Description |
| -------------- | -------------- |
| Cross-Region Restore | - Restore or replicate infrastructure on demand in the same or across different regions  \n - Restore data on-demand within or across regions for ultimate flexibility and security  \n - Restore backed-up workloads, resources, applications, and data into an existing VPC or create a new VPC and restore on-demand in any region across IBM Cloud |
| Granular Restore | - Restore entire VPC infrastructure  \n - Individual resource restoration  \n - Restore on-demand in the same VPC, Region, or across different VPCs and regions |
{: caption="Wanclouds VPC+ DRaaS restore options" caption-side="bottom"}

Wanclouds VPC+ DRaaS also includes:

| Other features | Description |
| -------------- | -------------- |
| Automated Discovery | - Automatically discover VPC resources and track infrastructure across multiple regions  \n - Real-time inventory of all cloud resources |
| Topology Visualization | - Visualize VPC topology and inter-resource relationships  \n - Visualize resource relationships for faster diagnostics  \n - Dependency mapping for impact analysis |
| Compliance Management | - Apply compliance policies and set up compliance policies  \n - Discover and track all resources with compliance and visualization features  \n - Audit trails for backup and restore operations |
{: caption="Wanclouds VPC+ DRaaS other features" caption-side="bottom"}

For more information, see [IBM documentation](https://docs.wanclouds.net/ibm/) and [VPC+ DRaaS (VPC+ Disaster Recovery as a Service)](https://cloud.ibm.com/catalog/services/vpc-draas-vpc-disaster-recovery-as-a-service)


## RackWare DR
{: #virt-sol-vpc-component-design-rackware-dr}

## RackWare RMM for VPC VSI Cross-Region Disaster Recovery
{: #virt-sol-component-design-rackware-vpc-dr}

RackWare Management Module (RMM) platform integrates with IBM Cloud VPC to enable intelligent provisioning, workload mobility, and cross-region DR planning. RackWare enables both hot and warm standby deployments with rapid failover and rollback capabilities across IBM Cloud VPC with policy-driven DR strategies.

The RackWare Management Module (RMM) is deployed in IBM Cloud VPC and provides a centralized interface for managing, scheduling, and automating migration and disaster recovery tasks. The following list includes the features of RMM.

* Deployed as VSI in IBM Cloud VPC
* Uses Floating IP for external GUI access
* SSH connectivity to source VSIs (primary region)
* API access to IBM Cloud VPC for auto-provision in DR region
* Available in IBM Cloud Catalog with seamless deployment
* Agentless approach supporting any current version of Windows and Linux
* Autoprovision capabilities with matching source specifications
* TNG sync for improved RPO
* Policy-driven automation for failover and failback
* GUI-based management with real-time monitoring

The following tables lists the disaster recovery configuration models.

| Disaster recovery configuration model | Description |
| -------------- | -------------- |
| Passthrough where RMM acts as proxy between source and target servers  \n - enabled by default | - Source and target VSIs cannot communicate directly  \n - Cross-region networking not configured  \n - Additional security layer desired  \n - Centralized traffic monitoring required  \n No direct connectivity required between regions  \n Centralized control and monitoring  \n - Simplified firewall rules  \n All data flows through RMM  \n RMM bandwidth becomes bottleneck for large datasets  \n Additional network hop adds latency |
| Direct Mode | - Source and target VSIs both use Floating IPs  \n - Transit Gateway configured between regions  \n - High-bandwidth requirements  \n - Minimize network latency  \n - Better performance for large datasets  \n - Reduced load on RMM server  \n - Lower latency for data transfer  \n - Network connectivity between regions (Transit Gateway or Floating IPs)  \n - Firewall rules allowing SSH between source and target  \n - Security considerations for cross-region traffic |
{: caption="RMM disaster recovery configuration models" caption-side="bottom"}

RackWare provides flexible scheduling with options for hot and cold standby of replicated workloads.

| Aspect | Hot Standby | Warm Standby |
|--------|-------------|--------------|
| **DR VSIs State** | Running continuously | Powered off |
| **Failover Time** | Minutes | 5-15 minutes (boot time) |
| **Cost** | Higher (running compute) | Lower (storage only) |
| **Use Cases** | Mission-critical, RTO < 5 min | Important workloads, RTO < 30 min |
| **Sync Impact** | Faster (VSI already running) | Must boot before final sync |
{: caption="Rackware capabilities" caption-side="bottom"}

For more informatiopn, see the following:

- [Protect critical workloads across hybrid and multi-cloud environments](https://www.rackwareinc.com/rackware-platform/disaster-recovery){: external}
- [RackWare and IBM Cloud](https://www.rackwareinc.com/solutions/cloud-environments/rackware-and-ibm){: external}
- [RackWare RMM users Guide for IBM Cloud](https://rackware.attachments9.freshdesk.com/data/helpdesk/attachments/production/5193588906/original/Rackware%20RMM%20Users%20Guide%20for%20IBM%20Cloud%20v2.2.pdf?response-content-type=application%2Fpdf&Expires=1764355600&Signature=gLDEmBxGd1dCMuzHjAP1FT3cCOzV6J7PGG7AHJ7dTKpTyGCvsY2IkzwQKI7VcJu~vnXprXUmkR9IUUUm0yhyD3hFdHU9tYZd4-6NfrZ7Ix2wXNfY44D2-rFWDoNy-LfiFaD2huPCdY2m-~1kw0ZtPqHLF7h5194~VPrhNgRPIrj~sfgN8wF8M8TLzkgZ84-MxbU~nn98rQFcnRgrKIc2inkfD~VYuAGaScmepjhRRoc8Gkd5LRIbPfyU1rAKWEj0L8AhfhjvLLVMGiu6CbfTGD1gLawXa24zm4ZR8-80LKdKstsJJx4vqgHEAbIQAD--XZsvY7CQ5AupWhdtPVzySA__&Key-Pair-Id=APKAJ7JARUX3F6RQIXLA){: external}
