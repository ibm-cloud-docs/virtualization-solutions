---

copyright:
  years: 2026
lastupdated: "2026-10-14"
lasttested: "[{LAST_TESTED_DATE}]"

keywords: Red Hat OpenShift Virtualization, ROVS, ROKS, VPC, VSI, Veeam Backup & Replication, VBR, Cloud Object Storage, COS, Scale-out Backup Repository, SOBR, Disaster Recovery, VM recovery, File Level Recovery

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 120m

---

{{site.data.keyword.attribute-definition-list}}

# Backing up Red Hat OpenShift Virtualization VMs by using Veeam Backup & Replication plug-in
{: #virt-sol-openshift-backup-vbr-plugin}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="120m"}

Deploy and configure Veeam Backup & Replication (VBR) on {{site.data.keyword.vpc_short}} to protect virtual machines (VMs) that run on Red Hat OpenShift Virtualization platforms by using the Veeam KubeVirt plug-in. The supported platforms are {{site.data.keyword.redhat_openshift_notm}} on {{site.data.keyword.cloud_notm}} (ROKS) and {{site.data.keyword.redhat_openshift_notm}} Virtualization Service (ROVS).
{: shortdesc}

Veeam Backup & Replication (VBR) version 13.1 is the first Veeam release to support {{site.data.keyword.redhat_openshift_notm}} Virtualization VM backups natively through the KubeVirt plug-in. The installation and configuration process might differ in newer releases, along with new features or user interface changes.

Complete the following tasks to back up and restore VMs on {{site.data.keyword.redhat_openshift_notm}} Virtualization:

- Install Veeam KubeVirt plugin on your pre-configured Veeam Backup & Replication (VBR) server
- Deploy Veeam KubeVirt Proxy (VKP) on your {{site.data.keyword.redhat_openshift_notm}} cluster
- Register the {{site.data.keyword.redhat_openshift_notm}} Virtualization cluster with your VBR server
- Test VM backup
- Test VM restore
- Test file-level recovery

## Before you begin
{: #virt-sol-openshift-backup-vbr-plugin-prereqs}

Before you begin, help ensure that the following prerequisites are in place:

- A {{site.data.keyword.redhat_openshift_notm}} cluster that is deployed with VM workloads to back up. The cluster must have the {{site.data.keyword.redhat_openshift_notm}} Virtualization operator installed and at least one running VM to use as a backup target during testing.
- A Veeam Backup & Replication (VBR) server, version 13.1 or later, that is installed and configured with a Scale-out Backup Repository (SOBR) as the backup destination. The SOBR must include at least one performance tier (local block storage that is formatted as Resilient File System (`ReFS`) or object storage) and optionally a capacity tier that is backed by {{site.data.keyword.cos_full_notm}} for long-term retention.
  - To deploy and set up your VBR server on {{site.data.keyword.vpc_short}}, see [Configuring Veeam Backup & Replication for IBM Cloud VPC virtual servers](/docs/virtualization-solutions?topic=virtualization-solutions-veeam-vbr-vpc-vsi-configuration).
- Open network ports for Veeam backup and restore operations, as described in the [Veeam Ports documentation](https://helpcenter.veeam.com/docs/vbr/userguide/plan_and_manage_used_ports.html){: external}. Firewall rules must permit the required traffic between VBR, the Veeam KubeVirt Proxy (VKP), and the {{site.data.keyword.redhat_openshift_notm}} worker nodes, as described in Veeam's [Ports used by Veeam KubeVirt Proxy](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_ports.html){: external}. VBR connects to VKP over port `443`.
- The Veeam KubeVirt plug-in installer that is downloaded from [Veeam's download page](https://www.veeam.com/products/data-platform-trial-download.html?tab=virtualization-plugins){: external}.

Review Veeam's [Planning guide](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_planning.html){: external} for version compatibility requirements, and the [Sizing Reference for Red Hat OpenShift Virtualization](https://helpcenter.veeam.com/docs/vbr/sizing_reference/sizing_reference.html?ver=13#openshift-virtualization){: external} to ensure that your VBR server and backup infrastructure are sized appropriately for the number of VMs and data volume you intend to protect.

## Architecture overview
{: #virt-sol-openshift-backup-vbr-plugin-architecture}

Several Veeam components work together to protect VMs running on {{site.data.keyword.redhat_openshift_notm}} Virtualization. Understanding how these components interact helps you plan network connectivity, storage sizing, and failure domains before you deploy.

Key components of the solution architecture:

Veeam Backup & Replication (VBR) server
:   A Windows&reg; virtual server running Veeam Backup & Replication. This central orchestrator schedules and manages all backup and restore operations, stores job metadata in its configuration database, and hosts the VBR console that administrators use.

Local repository
:   Resilient File System-formatted block storage that is attached to the VBR server or a dedicated Windows backup proxy. `ReFS` enables fast synthetic full backups and instant VM recovery through block cloning.

Cloud repository
:   {{site.data.keyword.cos_full_notm}} for long-term retention. Older restore points move from the local tier to object storage to reduce costs while they remain accessible for restore.

Scale-out backup repository (SOBR)
:   A logical construct combining the local performance tier and the cloud capacity tier into a single backup destination. Backup jobs target the SOBR, and Veeam automatically manages data placement and tiering policies across the extents.

Veeam KubeVirt plug-in
:   A stand-alone plug-in installed on the VBR server that extends native VBR functions to support {{site.data.keyword.redhat_openshift_notm}} Virtualization workloads. Without this plug-in, VBR cannot detect KubeVirt-based VMs.

Veeam KubeVirt Proxy (VKP)
:   A lightweight service deployed to your {{site.data.keyword.redhat_openshift_notm}} cluster that uses Helm. VKP acts as the data-path component on the cluster side, it receives instructions from VBR, coordinates snapshot creation, and orchestrates data movement to the backup repository.

Veeam Data Mover
:   A temporary service that VKP spins up in the cluster during active backup and restore operations to handle the actual data transfer. VKP creates it on demand and removes it after the job completes.

Gateway server
:   A server in the backup infrastructure that writes data to and reads data from object storage. A gateway server is always required when object storage is used as a backup target, regardless of the connection mode configured for the SOBR capacity tier.

Mount server
:   A server is required for guest OS file and application item restore operations (File Level Recovery). The mount server temporarily mounts backup content so that you can browse and extract individual files or folders without restoring the entire VM.

See Veeam's [Solution Architecture](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_architecture.html){: external} for more information about Veeam components and their interactions.

## Known limitations
{: #virt-sol-openshift-backup-vbr-plugin-limitations}

For a complete list of known limitations for the most recent release, see the [Considerations and limitations](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_limitations.html){: external} section of the Veeam plug-in user guide.
Further, note the following:

- Changed Block Tracking (CBT) is currently not supported, so full and incremental backups read the entire source disk on every run.
- Application-aware backup and application-level restores are currently not supported.
- Federal Information Processing Standards (FIPS) compliance is currently not supported.

The Veeam plug-in for KubeVirt is scoped exclusively to virtual machines within {{site.data.keyword.redhat_openshift_notm}} Virtualization. Native container workloads and namespace-level backups are not supported; protect those workloads by using Veeam Kasten.

When you back up to a local repository, Veeam estimates at the start of each backup job whether the extents have enough free space. Backup jobs might fail with "No scale-out repository extents have sufficient disk space to store the backup file," even though some extents have adequate free space. To resolve this issue, review the two solutions in the [Scale-out Backup Repository free space calculation](https://www.veeam.com/kb2282){: external} knowledge base (KB) article, or contact Veeam support to fine-tune your environment with additional registry keys.
{: note}

For large environments with hundreds of VMs, you might encounter a timeout during the cluster's inventory scan. This timeout occurs because the {{site.data.keyword.redhat_openshift_notm}} HAProxy has a 30-second timeout setting for the VKP route. If you encounter this timeout, run the following command to change the timeout value:

```sh
oc annotate route <vkp-route-name> -n <vkp-namespace> --overwrite haproxy.router.openshift.io/timeout=5m
```
{: pre}

## Best practices
{: #virt-sol-openshift-backup-vbr-plugin-best-practices}

Apply the following best practices to keep your Veeam backup environment performant, resilient, and manageable over time.

### Concurrent tasks
{: #virt-sol-openshift-backup-vbr-plugin-best-practices-concurrent-tasks}

Veeam processes backup and restore operations as concurrent tasks. Both the VBR server and the Veeam KubeVirt Proxy (VKP) have configurable limits on the number of tasks that they can run simultaneously. Setting these limits too low results in longer backup windows as jobs queue up; setting them too high risks saturating CPU, memory, or storage I/O on the nodes involved.

When you tune concurrent task limits, consider the following:

- Maximum concurrent tasks: The number of concurrent backups you can perform is a setting on the {{site.data.keyword.redhat_openshift_notm}} cluster server object. Start with the default value, then monitor CPU and memory utilization on the VKP pod and the worker nodes it runs on. Increase the limit only when you have headroom to spare and backup windows become unacceptably long. See the [Sizing reference for Red Hat OpenShift Virtualization](https://helpcenter.veeam.com/docs/vbr/sizing_reference/sizing_reference.html?ver=13#openshift-virtualization){: external} for guidance.
- SOBR extent task limits: Each extent in your SOBR has a maximum concurrent task count. Ensure that the sum of tasks across all backup jobs targeting the SOBR does not exceed the combined capacity of the extents to avoid backup failures due to resource exhaustion.
- Staggered job scheduling: If you run multiple backup jobs, stagger their start times so that they do not all start at the same time. This smooths out peak load on VKP, the storage network, and the backup repository, making it easier to stay within concurrent task limits.
- CBT limitations: Because CBT is currently not supported (see [Known limitations](#virt-sol-openshift-backup-vbr-plugin-limitations)), every backup reads the entire source disk. This increases storage I/O and CPU use on VKP and the VBR server, and extends backup job runtimes.

### 3-2-1 backup rule
{: #virt-sol-openshift-backup-vbr-plugin-best-practices-321}

Apply the 3-2-1 backup rule as follows:

- Retain three copies of the data: original data in the persistent volume claim (PVC), the snapshot data, and the snapshot data exported to an external location.
- Store the copies on two different media. Veeam supports block, file, and object storage. For example, store your data on PVC and external Simple Storage Service (`S3`). A PVC and a local snapshot in the same Kubernetes cluster do not count as two different media, because both copies are lost if the cluster is unrecoverable in a disaster.
- Keep one backup copy offsite. Set up backup copy jobs to transfer your backups to a public cloud provider or auxiliary storage in a separate site. Exporting backups to a locally hosted profile through Network File System (`NFS`) file storage is not considered offsite.

You can follow the 3-2-1 rule in several ways. The preferred approach is to use Veeam SOBR to keep the same backup copy on the performance tier and capacity tier. For more information, see [SOBR tiering](#virt-sol-openshift-backup-vbr-plugin-best-practices-sobr-tiering).

You can further enhance this to the 3-2-1-1-0 rule by using immutability. For more information, see Veeam's blog on [3-2-1 backup rule](https://www.veeam.com/blog/321-backup-rule.html){: external}.

### SOBR tiering
{: #virt-sol-openshift-backup-vbr-plugin-best-practices-sobr-tiering}

The Scale-out Backup Repository (SOBR) capacity tier offloads restore points from the performance tier to the capacity tier, but this occurs only according to the tiering schedule and policy you configure.

When you configure your SOBR tiering schedule, consider the following:

- Enable the operational restore window: Configure the SOBR tiering schedule to move data to the capacity tier outside your primary backup and restore windows. This prevents tiering I/O from competing with active backup jobs for storage network bandwidth and CPU on the gateway server.
- Set an appropriate offload age: The **Move backups older than** setting controls how long restore points remain on the performance tier before they move to object storage. Set this to a value that balances fast local restores for recent points (no object storage retrieval latency) against local storage capacity. A common starting point is 7 to 14 days, adjusted based on your recovery point objective (RPO), recovery time objective (RTO), and local storage size.
- Enable copy mode for the capacity tier: Rather than moving restore points exclusively to object storage, enable the **Copy backups to object storage as soon as they are created** option alongside the tiering schedule. This provides an immediate second copy on object storage without waiting for the age-based offload, improving data durability from the moment a backup completes.
- Verify tiering activity: After you enable the tiering schedule, confirm that data moves to the capacity tier as expected by reviewing job history in the VBR console.

See Veeam's documentation on [SOBR Capacity Tier](https://helpcenter.veeam.com/docs/vbr/userguide/capacity_tier.html){: external} for full configuration details.

### Immutability
{: #virt-sol-openshift-backup-vbr-plugin-best-practices-immutability}

Enabling immutability on your backup repositories is one of the most effective defenses against ransomware and accidental or malicious deletion of backup data. Veeam supports immutability on both the local (hardened Linux&reg; repository) and cloud (object storage) tiers.

- Hardened Repository immutability (local tier): For the local performance tier, deploy a hardened Linux repository instead of a Windows-based extent. Veeam's hardened repository uses a single-use credential model and sets the immutable flag on backup files at the operating system (OS) level. This means that even if the VBR server is compromised, an attacker cannot delete the backup files directly. See Veeam's [Immutability guide for Hardened Repository](https://helpcenter.veeam.com/docs/vbr/userguide/hardened_repository_immutability.html){: external} for setup instructions.
- Object storage immutability (cloud tier): {{site.data.keyword.cos_full_notm}} supports Object Lock, which lets Veeam mark backup files as immutable for a defined retention period. When enabled, backup files cannot be deleted or overwritten, even by an administrator, until the immutability period expires. Configure this in your SOBR capacity tier settings by enabling the **Make recent backups immutable for** option and setting a retention period that aligns with your recovery time objectives. See Veeam's [Immutability guide for Object Storage Repository](https://helpcenter.veeam.com/docs/vbr/userguide/hiw_immutability_os.html){: external} for detailed configuration steps.
- Align immutability periods with your retention policy: Set the immutability period equal to or slightly longer than your backup retention policy to ensure that restore points remain protected for their full intended lifetime.
- Test restore from immutable storage: Periodically validate that you can successfully restore from immutable backup files to confirm that the Object Lock configuration is correct and that restore paths function.

An additional immutability period that is called Block Generation is added to your Object storage immutability and must be considered when you use cloud tier immutability with Veeam. For more information, see Veeam's documentation on [Block Generation](https://helpcenter.veeam.com/docs/vbr/userguide/object_storage_block_generation.html){: external}.

### Gateway server
{: #virt-sol-openshift-backup-vbr-plugin-best-practices-gateway}

A gateway server is required whenever object storage is used as a backup repository, and its placement directly affects the performance and cost of your backup and restore operations. By default, the VBR server serves as the gateway server.

When you plan gateway server placement, consider the following:

- Colocate with the VBR server: For most deployments on {{site.data.keyword.vpc_short}}, using the VBR server itself as the gateway server is the simplest and most cost-effective option. Data flows from VKP to the VBR server, which then writes to {{site.data.keyword.cos_full_notm}} over the private network endpoint, minimizing egress charges.
- Use private endpoints for {{site.data.keyword.cos_short}}: Ensure that your SOBR capacity tier uses the {{site.data.keyword.cos_short}} private endpoint (`s3.private.<region>.cloud-object-storage.appdomain.cloud`) or direct endpoint (`s3.direct.<region>.cloud-object-storage.appdomain.cloud`) rather than the public endpoint (`s3.<region>.cloud-object-storage.appdomain.cloud`). This eliminates egress costs and keeps backup traffic on the IBM backbone network.
- Use dedicated gateways for high-throughput environments: If you are protecting many VMs or have significant data change rates, deploy one or more dedicated gateway servers to offload object storage I/O from the VBR server. This is especially relevant when the VBR server is also used for restore operations concurrently with ongoing backups.
- Size the gateway server for your throughput: Ensure that the gateway server has sufficient network bandwidth and CPU to sustain the throughput required by your backup jobs. Review [Veeam's sizing recommendations](https://helpcenter.veeam.com/docs/vbr/sizing_reference/sizing_reference.html){: external} for gateway servers as a baseline.

### VBR configuration backup
{: #virt-sol-openshift-backup-vbr-plugin-best-practices-config-backup}

The VBR configuration database contains all job definitions, repository configurations, credentials, and infrastructure settings. If you lose this database without a recent backup, you must rebuild your entire backup infrastructure from scratch, even if all of your VM backup data is intact.

To protect your VBR configuration:

- Enable scheduled configuration backup: In the **VBR console**, click **Main Menu**, and then select **Configuration Backup**. Configure a daily schedule and point the configuration backup to a repository separate from your primary backup infrastructure, for example, a dedicated {{site.data.keyword.cos_short}} bucket or a network share outside the VBR server, so that a failure of the VBR server does not take the configuration backup with it.
- Enable encryption: Always encrypt the configuration backup file. The configuration database contains credentials for your backup infrastructure, and an unencrypted export is a significant security risk. Store the encryption password in a password manager or secrets vault accessible independently of the VBR server.
- Retain multiple restore points: Keep at least a few days' worth of configuration backup restore points so that you can recover from a misconfiguration not immediately noticed.
- Test the restore process: Periodically validate the configuration backup by restoring it to a test VBR instance to confirm that the backup file is valid and that you understand the recovery procedure before you need to run it under pressure.
- Back up before you make changes: Take a manual configuration backup before you make significant changes to your VBR infrastructure (for example, before you add a new repository, upgrade VBR, or modify SOBR settings) so that you have a clean rollback point.

See Veeam's documentation on [Creating Configuration Backups](https://helpcenter.veeam.com/docs/vbr/userguide/export_vbr_config.html){: external} for full configuration details.

## Installing Veeam KubeVirt plug-in
{: #virt-sol-openshift-backup-vbr-plugin-install}
{: step}

Install the Veeam KubeVirt plug-in on the VBR server to enable native backup of {{site.data.keyword.redhat_openshift_notm}} Virtualization VMs. The plug-in adds a KubeVirt inventory type, job wizards, restore flows, and infrastructure management screens to the VBR console.

To install the KubeVirt plug-in, close any active VBR console sessions and run the `.exe` installation file directly on the VBR server. The installer detects the existing VBR installation and integrates with it automatically.

After the installation completes, start the **VBR console** and go to the **Inventory** page. Verify that **Red Hat OpenShift Virtualization** is listed under the **Virtual Infrastructure** section in the navigation pane. If the entry is not listed, ensure that the plug-in installer completed without errors and that you are running VBR 13.1 or later.

For more information, see [Installing Veeam plug-in for KubeVirt](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_install_plugin.html){: external}.

## Deploying Veeam KubeVirt Proxy
{: #virt-sol-openshift-backup-vbr-plugin-deploy-vkp}
{: step}

The Veeam KubeVirt Proxy (VKP) is the cluster-side component that handles backup and restore data-path operations. VKP deploys as a Kubernetes workload by using Helm and exposes a Hypertext Transfer Protocol Secure (HTTPS) endpoint that the VBR server connects to when it orchestrates jobs.

To deploy VKP by using Helm and annotate your storage class, follow the steps in Veeam's deployment guide for [Installing Veeam KubeVirt Proxy](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_install_proxy.html){: external}.

After deployment, note the VKP connection details, the fully qualified domain name (FQDN) of the VKP route and the service account token, because you need them to register VKP with the VBR server in the next step.

### Optionally set up private route for VKP
{: #virt-sol-openshift-backup-vbr-plugin-deploy-vkp-private-route}

For {{site.data.keyword.redhat_openshift_notm}} clusters created with public and private service endpoints, only the public ingress controller is created by default. To have the VKP service communicate with the VBR server over the private network, you need a private ingress controller, a VKP service that uses a private route, and a VKP private route. This configuration is recommended for security and to avoid public network egress.

Clusters configured with private-only endpoints do not require this step because services already route over the private network by default. You do not need to create a new private ingress controller if one exists. Consult your cluster administrator before you proceed with this step.
{: note}

Find detailed instructions in the [Setting up private routes](/docs/openshift?topic=openshift-openshift_routes#private-routes) section of the {{site.data.keyword.redhat_openshift_notm}} documentation. When you follow those instructions for the VKP service, use the following commands:

1. Expose the VKP service privately:

   ```sh
   oc expose deploy vkp --name vkp-private -n vkp --type='ClusterIP' --port 8080
   ```
   {: pre}

2. Create the private VKP route:

   ```sh
   oc create route edge --service vkp-private --hostname <HOSTNAME> --port 8080 --insecure-policy='Redirect' -n vkp
   ```
   {: pre}

Replace `<HOSTNAME>` with the private Domain Name System (DNS) hostname that you want to assign to the VKP route. This hostname must be resolvable from the VBR server.

## Registering Veeam KubeVirt Proxy
{: #virt-sol-openshift-backup-vbr-plugin-register-vkp}
{: step}

With VKP deployed in the cluster, the next step is to register it with the VBR server. This registration establishes a trusted connection between VBR and the {{site.data.keyword.redhat_openshift_notm}} Virtualization cluster, and lists all namespaces and VMs within the cluster in the VBR inventory.

Register the Veeam KubeVirt Proxy (VKP) with your {{site.data.keyword.redhat_openshift_notm}} cluster by following the steps in Veeam's [Adding Red Hat OpenShift Virtualization Server](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_add_server.html){: external}. When prompted, provide the VKP FQDN and the service account token you recorded during the VKP deployment step.

The service account username field can be set to any descriptive value. The VBR console uses this value for display purposes only.
{: note}

After you register VKP, go to the **Inventory** page in the **VBR console** and verify that all namespaces and VMs are listed in your {{site.data.keyword.redhat_openshift_notm}} Virtualization cluster. If namespaces or VMs do not appear, verify that the VKP pod is running and that network connectivity from the VBR server to the VKP endpoint is available.

## Testing VM backup
{: #virt-sol-openshift-backup-vbr-plugin-backup}
{: step}

With VKP registered, create and run a backup job to verify that the end-to-end backup pipeline is working. The following steps describe the general backup job creation process.

1. In the **Veeam Backup & Replication Console**, go to **Home**.
2. In the ribbon, click **Backup Job**, and then select **Virtual Machine**.
   1. On the **Name** page:
      1. Provide a name for the backup job.
      2. Optionally provide a description for the backup job.
      3. Click **Next**.
   2. On the **Virtual machines** page:
      1. Use the **Add** button to select either the entire project or specific virtual machines within the project to back up. You can scope a job to an entire namespace to automatically protect new VMs added to that namespace in the future.
      2. Use the **Exclusion** button to exclude any objects that should not be backed up.
      3. Click **Next**.
   3. On the **Storage** page:
      1. From the **Backup repository** list, select the backup repository that you want as the backup destination.
      2. Adjust **Retention Policy** settings as needed. Retention defines how many restore points are kept before the system deletes the oldest restore point.
      3. Click **Advanced job settings**:
         1. On the **Backup** tab, adjust synthetic full and active full backup settings as needed. Synthetic fulls reduce the load on the source cluster.
         2. On the **Maintenance** tab, enable the health check option if wanted. Health checks periodically validate the integrity of stored backup files.
         3. On the **Storage** tab, adjust compression level and storage optimization settings as needed. Higher compression reduces storage consumption at the cost of additional CPU on the VBR server.
         4. On the **KubeVirt** tab, enable Guest Tools quiescence if wanted. Quiescence instructs the guest OS to flush I/O before the snapshot takes place, resulting in a more consistent backup.
         5. On the **Notifications** tab, configure notification settings if wanted to receive alerts on job success or failure.
      4. Click **Next**.
   4. On the **Schedule** page:
      1. Enable the **Run the job automatically** option and configure the job schedule. Align the schedule with your Recovery Point Objective (RPO).
      2. Adjust **Automatic retry** settings as needed to allow failed jobs to retry without manual intervention.
      3. Click **Apply**.
   5. On the **Summary** page, review the job configuration.
      1. Select **Run the job when I click Finish** to start the first backup immediately.
      2. Click **Finish**.
3. Monitor the backup job.
   1. In the **Home** view, locate the job in the **Jobs** list.
   2. Click the job to view progress and details.
   3. Wait for the backup job to complete and confirm that it finishes with a **Success** or **Warning** status. Investigate any errors before you proceed.

For more details, see Veeam's documentation on [Creating Backup Jobs](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_backup_job.html){: external} and [Managing Backup Jobs](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_manage_jobs.html){: external}.

## Testing VM restore
{: #virt-sol-openshift-backup-vbr-plugin-restore}
{: step}

Validating the restore process is as important as running a successful backup. The following steps describe the general process for restoring a VM to its original location.

1. In the **Veeam Backup & Replication Console**, go to **Home**.
2. In the ribbon, click **Restore**, and then select **KubeVirt**. Select **Entire machine restore**, and then select **Entire VM restore**.
   1. On the **Virtual machines** page:
      1. Use the **Add** button to locate the VM to restore.
      2. By default, the system selects the latest restore point. Use the **Point** button to restore to an earlier restore point if wanted.
      3. Click **Next**.
   2. On the **Restore Mode** page:
      1. Select **Restore to the original location**.
      2. Click **Next**.
      3. When prompted, click **Yes** to acknowledge that the restore overwrites the existing VM.
   3. On the **Reason** page:
      1. Optionally provide a reason for the restore to aid in auditing.
      2. Click **Next**.
   4. On the **Summary** page, review the restore configuration.
      1. Select **Power on the target VM after restoring** if you want the VM to start automatically after the restore completes.
      2. Click **Finish**.
3. Monitor the restore job.
   1. Track the restore progress and details in the Restore session window.
   2. Wait for the restore operation to complete and verify that the VM is running and accessible in the Red Hat OpenShift Virtualization console.

For more details, see Veeam's documentation on [Performing Entire VM Restore](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_entire_vm_restore.html){: external}.

## Testing File Level Recovery
{: #virt-sol-openshift-backup-vbr-plugin-flr}
{: step}

File Level Recovery (FLR) lets you restore individual files or folders from within a VM backup without restoring the entire VM. Use FLR for scenarios such as recovering an accidentally deleted file or retrieving a specific version of a configuration file.

FLR requires a Windows mount server and a Linux mount server that are configured as part of your backup infrastructure. If your VBR server runs on Windows, it can serve as the Windows mount server. For Linux guest file recovery, you need a separate Red Hat Enterprise Linux (RHEL)-based server that is configured as a Linux mount server. See Veeam's documentation on [Mount Server](https://helpcenter.veeam.com/docs/vbr/userguide/system_requirements_mount_server.html){: external} for system requirements.

If you intend to restore files directly back to the VM (rather than downloading them locally), the mount server must have network connectivity to the target VM. Keep all necessary ports open between the mount server and the VM. See Veeam's documentation on [Port requirements](https://helpcenter.veeam.com/docs/vbr/userguide/plan_and_manage_used_ports.html){: external}.

When you restore files directly to the VM, help ensure that there is no time drift between the VBR server and the VM. Clock skew can cause FLR sessions to fail.
{: important}

1. In the **Veeam Backup & Replication Console**, go to **Home**.
2. In the ribbon, click **Restore**, and then select **KubeVirt**. Select **Guest files restore**.
   1. On the **Machine** page, select the VM backup to restore files from, and click **Next**.
   2. On the **Restore Point** page, select the restore point to browse, and click **Next**.
   3. On the **Reason** page, optionally provide a reason for the restore, and click **Next**.
   4. On the **Summary** page, review the restore configuration, and click **Browse**.
3. In the **Backup Browser** window, go to the file system of the backed-up VM and select the file or folder to restore. Click **Restore** and select the wanted restore option (restore to original location, restore to a different location, or copy to).
4. Monitor the restore job.
   1. Track the restore progress and details in the Restore session window.
   2. Wait for the restore operation to complete and verify that the target file or folder is restored.

For more details, see Veeam's documentation on [Performing File-Level Restore](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_guest_flr.html){: external}.

## Upgrading Veeam KubeVirt Proxy
{: #virt-sol-openshift-backup-vbr-plugin-upgrade-vkp}

To upgrade VKP to the latest version, use Helm.

1. Update the `vkp` Helm repository:

   ```sh
   helm repo update vkp
   ```
   {: pre}

2. Upgrade VKP:

   ```sh
   helm upgrade --reuse-values vkp vkp/vkp -n <vkp-namespace>
   ```
   {: pre}

   Replace `<vkp-namespace>` with the namespace where VKP is deployed.

You can roll back to the previous version by using the same upgrade command with the `--version` flag.
{: note}

For more information, see [Upgrading Veeam KubeVirt Proxy](https://helpcenter.veeam.com/docs/vbr/userguide/vkp_upgrading_proxy.html?ver=13){: external}.

## Next steps
{: #virt-sol-openshift-backup-vbr-plugin-next-steps}

Now that you have installed and configured the Veeam KubeVirt plugin and verified backup and restore operations for VMs on Red Hat OpenShift Virtualization platforms (ROKS and ROVS), explore the following related topics to deepen your understanding and harden your backup environment:

- [Veeam KubeVirt plugin documentation](https://helpcenter.veeam.com/docs/vbr/userguide/vkp.html){: external}
- [Veeam Sizing Reference for OpenShift Virtualization](https://helpcenter.veeam.com/docs/vbr/sizing_reference/sizing_reference.html?ver=13#openshift-virtualization){: external}
- [Veeam Immutability guide for Hardened Repository](https://helpcenter.veeam.com/docs/vbr/userguide/hardened_repository_immutability.html){: external} and [Veeam Immutability guide for Object Storage Repository](https://helpcenter.veeam.com/docs/vbr/userguide/hiw_immutability_os.html){: external}
- [Troubleshooting VBR configuration and backup issues](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-vbr-vpc-vsi-integration-configuration)
- [Troubleshooting VBR restore issues](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-vbr-vpc-vsi-integration-restore)
