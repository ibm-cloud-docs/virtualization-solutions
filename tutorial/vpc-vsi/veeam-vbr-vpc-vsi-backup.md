---

copyright:
  years: 2026

lastupdated: "2026-07-23"

keywords: Veeam Backup Replication VPC, VBR agent deployment tutorial, scale-out backup repository SOBR, Cloud Object Storage backup tier, ReFS backup repository VPC, Veeam protection groups VSI, HMAC credentials COS integration, Windows Linux backup agents, VPC backup configuration guide

subcollection: virtualization-solutions

content-type: tutorial
services: vpc, cloud object storage
account-plan: paid
completion-time: 90m

---

{{site.data.keyword.attribute-definition-list}}

# Backing up IBM Cloud VPC virtual servers with Veeam Backup & Replication
{: #veeam-vbr-vpc-vsi-backup}
{: toc-content-type="tutorial"}
{: toc-services="vpc"}
{: toc-completion-time="90m"}

Install Veeam backup agents on {{site.data.keyword.vpc_short}} virtual servers and create protection groups and backup jobs.
{: shortdesc}

## Objectives
{: #veeam-vbr-vpc-backup-objectives}

Deploy {{site.data.keyword.vpc_short}} virtual server instances, install Veeam backup agents, and back up target virtual servers by using Veeam Backup & Replication 13.

- Deploy virtual server instances for Veeam to back up.
- Deploy Veeam agents on target virtual servers.
- Create protection groups.
- Create and configure backup jobs.

## Before you begin
{: #veeam-vbr-vpc-backup-prerequisites}

Review and meet the following prerequisites:

- Basic understanding of backup and recovery concepts
- Familiarity with Linux&reg; and Windows&reg; Server administration
- An {{site.data.keyword.cloud_notm}} account with {{site.data.keyword.vpc_short}} access
- Appropriate IBM Cloud Identity and Access Management ([IAM](/docs/iam?topic=iam-iamoverview)) permissions to create and manage VPC resources
- Familiarity with the [Veeam Backup & Replication configuration tutorial](/docs/virtualization-solutions?topic=virtualization-solutions-veeam-vbr-vpc-vsi-configuration)
- A working VBR 13 installation with configured repositories (Scale-Out Backup Repository (SOBR) that uses a local disk and {{site.data.keyword.cos_full_notm}})

## Best practices
{: #veeam-vbr-vpc-backup-best-practices}

Follow these best practices to provide reliable and efficient backup operations.

### Backup strategy
{: #veeam-vbr-vpc-backup-strategy}

Use these strategy recommendations to balance recovery speed, retention, and operational resilience. A clear backup strategy helps you validate recoverability and maintain consistent protection of your workloads.

- Follow the 3-2-1 rule: Keep three copies of data on two different media types, with one copy off site.
- Use SOBR tiering: Keep recent backups (7-14 days) on local storage for fast restores.
- Perform regular testing: Perform restore tests monthly to validate backup integrity.
- Document procedures: Maintain runbooks for backup and restore operations.
- Monitor job status: Review backup reports daily and address failures promptly.

### Performance optimization
{: #veeam-vbr-vpc-performance-optimization}

Use these optimization practices to improve backup throughput and make better use of compute and storage resources.

- Parallel processing: Configure one concurrent task per central processing unit (CPU) core.
- Backup windows: Schedule backups during off-peak hours.
- Incremental backups: Use forever-forward incremental for efficiency.
- Synthetic full backups: Enable synthetic full backups to reduce backup windows.
- Compression: Use **Optimal** compression to balance speed and storage savings.
- Deduplication: Enable inline deduplication for storage efficiency.

### IAM access policies
{: #veeam-vbr-vpc-backup-access-policies}

Ensure that you have the following IAM access policies:

- VPC Infrastructure Services: Editor or Administrator role.
- {{site.data.keyword.cos_short}}: Writer or Manager role.
- Permission to create and manage the following resources:
   - Virtual servers
   - Block storage volumes
   - Security groups
   - Floating IPs

## Provisioning Windows and Red Hat Enterprise Linux virtual servers
{: #veeam-endpoint-vpc-provision-vsi}
{: step}

Complete the following steps to create Windows and {{site.data.keyword.rhel_full}} virtual servers for Veeam Backup & Replication to back up.

Confirm that `lvm2` is installed on the {{site.data.keyword.rhel_short}} or Linux virtual server.
{: note}

Open the required ports on the Windows virtual server. For more information, see [Ports that are used by Veeam Backup & Replication](https://helpcenter.veeam.com/docs/vbr/userguide/used_ports.html){: external} to determine which ports to open. The Windows Firewall is disabled for testing purposes only. Do not disable the firewall in a production environment.
{: note}

1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/infrastructure/compute/vs){: external} to create Windows and {{site.data.keyword.rhel_short}} virtual servers for backup.
2. Record the private IP address of each virtual server.

## Creating protection groups
{: #veeam-vbr-vpc-create-protection-groups}
{: step}

Complete the following steps to create protection groups to organize your virtual servers before you deploy agents. Protection groups in Veeam 13 help you group servers and apply backup policies consistently.

### Creating a Windows protection group
{: #veeam-vbr-vpc-create-windows-protection-group}

Use a Windows protection group to organize Windows virtual servers and deploy backup agents.

1. In the Veeam Backup & Replication Console, go to **Inventory**.
2. Create a new protection group for Windows servers. In the navigation window, right-click **Physical & Cloud Infrastructure**.
   1. Click **Add protection group**.
   2. Select **Individual computers**.
   3. In the **New Protection Group** wizard:
       1. On the **Name** page, enter `VPC-Windows-Servers` for **Name**.
       2. For **Description**, enter `Windows VSIs in IBM Cloud VPC for backup`.
       3. Click **Next**.
   4. On the **Computers** page, click **Add**.
       1. In the **Add Computer** form:
          1. For **DNS name or IP address**, enter the private IP address of your Windows virtual server instance.
          2. For **Connect using admin credentials**, click **Add**.
             1. For **Username**, enter `Administrator` or a domain account.
             2. For **Password**, enter the password.
             3. For **Description**, enter `Windows VSI Admin Credentials`.
             4. Click **OK**.
          3. Click **OK**.
       2. Click **Next**.
   5. On the **Options** page, select **Install backup agent**.
      1. Select **Install change block tracking driver**.
      2. Select **Enable auto-update for installed components**.
      3. Select **Perform reboot automatically if required**.
      4. Click **Next**.
   6. On the **Review** page, review the computer details, and then click **Apply** if needed.
      1. Repeat steps 1 through 3 to add additional Windows virtual server instances to the protection group.
      2. Click **Next**.
   7. On the **Apply** page, click **Next** if you need to apply any changes.
   8. On the **Summary** page, click **Finish**.
3. Wait for the **Machine rescan** to complete.

### Creating a Linux protection group
{: #veeam-vbr-vpc-create-linux-protection-group}

Use a Linux protection group to organize Linux virtual servers and prepare them for agent deployment.

1. In the Veeam Backup & Replication Console, go to **Inventory**.
2. Create a new protection group for Linux servers. In the navigation window, right-click **Physical & Cloud Infrastructure**.
   1. Click **Add protection group**.
   2. Select **Individual computers**.
   3. In the **New Protection Group** wizard:
      1. On the **Name** page:
         1. For **Name**, enter `VPC-Linux-Servers`.
         2. For **Description**, enter `Linux VSIs in IBM Cloud VPC for backup`.
         3. Click **Next**.
      2. On the **Computers** page, click **Add**.
         1. In the **Add Computer** form:
            1. For **DNS name or IP address**, enter the private IP address of your Linux virtual server instance.
            2. For **Connect using admin credentials**, click **Add**.
               1. For **Username**, enter `root` or a domain account.
               2. For **Password**, enter the password or Secure Shell (SSH) key.
               3. For **Description**, enter `Linux VSI Admin Credentials`.
               4. Click **OK**.
            3. Click **OK**.
         2. Click **Next**.
      3. On the **Options** page, select **Install backup agent**.
         1. Select **Enable auto-update for installed components**.
         2. Select **Perform reboot automatically if required**.
         3. Click **Next**.
      4. On the **Review** page, review the computer details, and then click **Apply** if needed.
         1. Repeat steps 1 through 3 to add additional Linux virtual server instances to the protection group.
         2. Click **Next**.
      5. On the **Apply** page, click **Next** if you need to apply any changes.
      6. On the **Summary** page, click **Finish**.
3. Wait for the **Machine rescan** to complete.

### Verifying protection group deployment
{: #veeam-vbr-vpc-verify-protection-groups}

After you create the protection groups, verify that the servers are online and that the Veeam agents are installed.

1. In the Veeam Backup & Replication Console, go to **Inventory**.
2. Verify protection groups. In the navigation pane, expand **Physical & Cloud Infrastructure**.
   1. Expand **Protection Groups**.
   2. Click **VPC-Windows-Servers** to view Windows virtual server instances.
       - Verify all Windows virtual server instances are listed with the status **Online**.
       - Verify **Agent** column shows **Installed**.
   3. Click **VPC-Linux-Servers** to view Linux virtual server instances.
       - Verify all Linux virtual server instances are listed with the status **Online**.
       - Verify **Agent** column shows **Installed**.
3. If any servers show **Agent not installed** or **Offline**:
   1. Right-click the server.
   2. Click **Install Agent** to retry deployment.
   3. Check network connectivity and credentials if deployment fails.

## Managing Veeam agents
{: #veeam-vbr-vpc-manage-agents}
{: step}

After you create protection groups in the previous step, Veeam agents are automatically deployed to all virtual servers in those groups. The section explains how to verify agent status and manually add more servers if needed.

1. In the Veeam Backup & Replication Console, go to **Inventory**.
2. Check agent status. In the left navigation pane, expand **Physical & Cloud Infrastructure**.
    1. Expand **Protection Groups**.
    2. Click on **VPC-Windows-Servers** or **VPC-Linux-Servers**.
    3. Verify that all servers show by double clicking each server:
       - **Status**: Online (green checkmark).
       - **Agent**: Installed.
       - **Version**: Current Veeam Agent version.
3. If any server shows issues:
   1. Right-click the server.
   2. Click **Rescan** to refresh the status.
   3. If agent is not installed, click **Install Agent** to retry.

## Creating backup jobs
{: #veeam-vbr-vpc-create-backup-jobs}
{: step}

Complete the following steps to create backup jobs for your virtual servers by using the protection groups that you created earlier.

### Creating a Windows backup job
{: #veeam-vbr-vpc-create-windows-backup-job}

Create a managed backup job for the Windows protection group.

1. In the Veeam Backup & Replication Console, go to **Home**.
2. Create a new backup job for Windows servers. In the ribbon, click **Backup Job** > **Windows computer**.
   1. In the **New Agent Backup Job** wizard:
       1. On the **Job Mode** page, select **Server** for **Type**.
       2. Select **Managed by backup server** for **Mode**.
       3. Click **Next**.
   2. On the **Name** page:
       1. For **Name**, enter `VPC-Windows-Backup-Job`.
       2. For **Description**, enter `Daily backup of Windows VSIs in VPC`.
       3. Click **Next**.
   3. On the **Computers** page, click **Add** > **Protection group**.
       1. Select **VPC-Windows-Servers** from the list.
       2. Click **OK**.
       3. If needed, click **Add** > **Individual computer** to add a specific virtual server that is not in the protection group.
       4. Click **Next**.
   4. On the **Backup Mode** page:
       1. Select **Entire computer** for a full system backup.
       2. If you need to protect only selected volumes, select **Volume level** instead.
       3. Click **Next**.
   5. On the **Storage** page:
       1. Select **SOBR-VPC-Backups** for **Backup repository**.
       2. For **Retention policy**, specify:
            - **Restore points to keep**: `14` (adjust based on your requirements)
       3. Click **Advanced**.
       4. In the **Advanced Settings** form:
           1. In the **Storage** tab:
                - For **Compression level**, select **Optimal** or **High**.
                - For **Encryption**, select **Enable backup file encryption**, and then add a password.
           2. Click **OK**.
       5. Click **Next**.
   6. On the **Guest Processing** page, click **Next**.
   7. On the **Schedule** page:
       1. Select **Run the job automatically**.
       2. For **Schedule**, configure:
             - **Daily**: At `10:00 PM` (adjust based on your backup window)
             - **Days**: Monday through Sunday
       3. Click **Apply**.
   8. On the **Summary** page, review the job configuration.
       1. Select **Run the job when I click Finish** to start the first backup immediately.
       2. Click **Finish**.
3. Monitor the Windows backup job.
    1. In the **Home** view, locate the job in the **Jobs** list.
    2. Click the job to view progress and details.
    3. Wait for the first backup to complete.

### Creating a Linux backup job
{: #veeam-vbr-vpc-create-linux-backup-job}

Create a managed backup job for the Linux protection group.

1. In the Veeam Backup & Replication Console, go to **Home**.
2. Create a new backup job for Linux servers. In the ribbon, click **Backup Job** > **Linux computer**.
    1. In the **New Agent Backup Job** wizard:
       1. On the **Job Mode** page, select **Server** for **Type**.
            1. Select **Managed by backup server** for **Mode**.
            2. Click **Next**.
       2. On the **Name** page:
              1. Enter `VPC-Linux-Backup-Job` for **Name**.
              2. For **Description**, enter `Daily backup of Linux VSIs in VPC`.
              3. Click **Next**.
       3. On the **Computers** page, click **Add** > **Protection group**.
              1. Select **VPC-Linux-Servers** from the list.
              2. Click **OK**.
              3. If needed, click **Add** > **Individual computer** to add a specific virtual server that is not in the protection group.
              4. Click **Next**.
       4. On the **Backup Mode** page:
             1. Select **Entire computer** for a full system backup.
             2. If you need to protect only selected volumes, select **Volume level** instead.
             3. Click **Next**.
       5. On the **Storage** page:
            1. For **Backup repository**, select **SOBR-VPC-Backups**.
            2. For **Retention policy**, specify:
                - **Restore points to keep**: `14` (adjust based on your requirements)
            3. Click **Advanced**.
            4. In the **Advanced Settings** form:
                1. In the **Storage** tab:
                   - For **Compression level**, select **Optimal** or **High**.
                   - For **Encryption**, select **Enable backup file encryption**, and then add a password.
                2. Click **OK**.
            5. Click **Next**.
       6. On the **Guest Processing** page, click **Next**.
       7. On the **Schedule** page:
          1. Select **Run the job automatically**.
          2. For **Schedule**, configure:
             - **Daily**: At `10:00 PM` (adjust based on your backup window)
             - **Days**: Monday through Sunday
          3. Click **Apply**.
       8. On the **Summary** page, review the job configuration.
          1. Select **Run the job when I click Finish** to start the first backup immediately.
          2. Click **Finish**.
3. Monitor the Linux backup job.
    1. In the **Home** view, locate the job in the **Jobs** list.
    2. Click the job to view progress and details.
    3. Wait for the first backup to complete..

### Verifying the backup jobs
{: #veeam-vbr-vpc-verify-backup-jobs}

After you create both backup jobs, verify that the jobs run correctly and that the appropriate repositories store the backup files.

1. In the Veeam Backup & Replication Console, go to **Home**.
2. Verify that both backup jobs are listed.
   1. Check **VPC-Windows-Backup-Job** status.
   2. Check **VPC-Linux-Backup-Job** status.
3. Review job statistics.
   1. Click each job to view details.
   2. Verify that the job backs up all virtual servers in the protection groups.
   3. Check backup duration and data transfer rates.
4. Review backup files.
   1. Go to **Backup Infrastructure** > **Scale-out Repositories**.
   2. Click **SOBR-VPC-Backups**.
   3. Verify that the local repository contains backup files.
   4. After 7 days, verify that VBR moves the files to {{site.data.keyword.cos_full_notm}}.

## Next steps
{: #veeam-vbr-vpc-backup-next-steps}

Consider the following next steps:

- Test restores: Validate backup integrity when you perform restore tests.
- Configure replication: Set up Veeam replication for additional protection.
- Implement monitoring: Integrate with tools such as Veeam ONE and Grafana.
- Automate operations: Create PowerShell scripts for common tasks.
- Plan disaster recovery: Document and test DR procedures.
- Optimize costs: Review and adjust retention policies and SOBR settings.

## Related links
{: #veeam-vbr-vpc-backup-related-content}

Explore the following related topics for Veeam Backup & Replication on {{site.data.keyword.vpc_short}} virtual servers:

- [Troubleshooting agent installation or backup failures](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-vbr-vpc-vsi-integration-configuration)
- [{{site.data.keyword.vpc_short}} overview](/docs/vpc?topic=vpc-about-vpc)
- [{{site.data.keyword.cos_short}} overview](/docs/cloud-object-storage?topic=cloud-object-storage-about-cloud-object-storage)
- [Veeam Backup & Replication User Guide](https://helpcenter.veeam.com/docs/vbr/userguide/overview.html){: external}
