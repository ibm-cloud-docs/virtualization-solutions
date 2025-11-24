---

copyright:
  years: 2025
lastupdated: "2025-11-24"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# Resiliency Design
{: #resiliency-design}

This is a short description that introduces the content in this topic.
{: shortdesc}

## IBM Cloud VPC
{: #vpc-resiliency}

### Backup
{: #vpc-resiliency-backup}

- From [About Backup for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-backup-service-about&interface=ui)
- Use the Backup for VPC service to automatically create backups and manually restore Block Storage for VPC volumes and File Storage for VPC shares from backup snapshots.
- Backups are in effect, automated snapshots with a retention date.
- Create backup policies to identify the resources to back up. The policy identifies resources by type and/or tag.
- Create backup plans to schedule the frequency of your backups.
- You can create up to 10 backup policies in one region.
- You can create up to 4 backup plans per backup policy.
- You can specify the frequency of your backups to be daily, weekly, or monthly. Or you can use a cron expression to specify the frequency.
- Use a consistency group to backup all of the Block Storage for VPC volumes that are attached to a specific virtual server instance as a group.
- When the first backup snapshot is taken, the entire contents of the volume are copied and retained in a regional storage repository. Subsequent backups of the same volume capture the changes that occurred since the previous backup.
- You can restore data from a backup snapshot to a new, fully provisioned volume. If the backup is of a boot volume, you can use it to provision a new instance.
- You must set a retention period for your backups. It can be based on…
- The number of days that you want to keep the backups.
- The total number of backups that you want to retain before the oldest ones are deleted.
- All of the above.
- The default retention period is 30 days, but you can set the retention period value anywhere between 1 and 1000 days.
- You can have 750 snapshots of first-generation storage volumes and 512 snapshots of second-generation storage volumes.
- You can copy a Block storage backup snapshot from one region to another region, and later use that snapshot to restore a volume in the new region.
- From [Getting started with Backup and Recovery](https://cloud.ibm.com/docs/backup-recovery?topic=backup-recovery-getting-started-backup-recovery)
- It is a fully managed service that provides backup solutions for various IBM Cloud® services and customer workloads running on IBM Cloud.
- It is, in fact, Cohesity.
- It lets you define backup schedules to routinely protect data sources using a secure, agent-based, application-consistent backup service.
- To use it, you…
- Create a data source connection to establish connectivity between your source and the service.
- Deploy a data source connector for virtual machines to move data between your sources and the Backup service.
- Install the backup agents onto the source servers you want to back up.
- Register your source servers with the Backup and Recovery service instance so they can be protected.
- Create protection policies to schedule backup jobs and define retention policies.
- From [About Veeam](https://cloud.ibm.com/docs/vpc?topic=vpc-about-veeam)
- You can use Veeam software to back up your storage data on a virtual server instance for IBM Cloud® Virtual Private Cloud.
- You can use Veeam software backups to protect the following resources:
- Individual volumes
- Folders and files.
- Before using Veeam to protect your VSIs, you must order Veeam licenses. You can do so via either the Veeam website or the IBM Cloud Portal
- See [Ordering Veeam licenses](https://cloud.ibm.com/docs/vpc?topic=vpc-ordering-veeam-licenses&interface=ui)
- Use the Veeam Agent for Linux or Windows to create backup and restore jobs directly from the VSI.
- See [Installing and operating the Veeam Agent](https://cloud.ibm.com/docs/vpc?topic=vpc-using-veeam-agent&interface=ui)

### Disaster Recovery
{: #vpc-resiliency-dr}

- From [Understanding high availability and disaster recovery for IBM Cloud VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-ha-dr-vpc)
- The strategy for disaster recovery is to provide scripting automation to restore a VPC workload in a recovery location. The VPC API, SDK, and CLI can be used by customers to create scripts to recover resources in an available location during a disaster.
- IBM Cloud VPC supports cross-zonal and cross-regional recovery of volumes and file shares via volume snapshots, file share snapshots and replica shares.
- Customers are responsible for the backup and recovery of their VPC configurations.
- Customers can use 3rd party services to provide continuous backup and recovery of their Block Storage and File Storage.
- When you restore a boot volume from a snapshot, expect some performance degradation while the data is copied to the boot volume from the snapshot. Use Fast Restore Snapshots to achieve faster recovery times.
- See [FAQ for Block Storage for VPC Snapshots > What is a fast restore snapshot?](https://cloud.ibm.com/docs/vpc?topic=vpc-snapshots-vpc-faqs&interface=ui#faq-snapshot-fr)
- From [Building a cross region disaster recovery solution with IBM Cloud services](https://cloud.ibm.com/docs/resiliency?topic=resiliency-KeyServicesTitle)
- For backup and restore, choose between IBM Storage Protect and IBM Cloud Backup for VPC
- IBM Storage Protect is, in fact, Spectrum Protect.
- See [Building a Cross Region DR Solution With IBM Cloud Services > Backup and restore options](https://cloud.ibm.com/docs/resiliency?topic=resiliency-KeyServicesTitle#backup-and-restore-options)
- Use Wanclouds for a managed DR solution. Their managed offering enables your entire VPC stack, including IKS clusters, network setups, and COS bucket data, to be backed up and restored quickly whenever necessary.
- See [IBM Disaster Recovery and Cloud Migration with VPC+ DRaaS](https://www.wanclouds.net/ibm)
