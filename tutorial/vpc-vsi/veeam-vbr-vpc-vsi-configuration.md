---

copyright:
  years: 2026
lastupdated: "2026-07-23"

keywords: Veeam Backup Replication VPC, VBR deployment tutorial, scale-out backup repository SOBR, Cloud Object Storage backup tier, ReFS backup repository VPC, HMAC credentials COS integration, VPC backup configuration guide


subcollection: virtualization-solutions

content-type: tutorial
services: vpc, cloud object storage
account-plan: paid
completion-time: 120m

---

{{site.data.keyword.attribute-definition-list}}

# Configuring Veeam Backup & Replication for IBM Cloud VPC virtual servers
{: #veeam-vbr-vpc-vsi-configuration}
{: toc-content-type="tutorial"}
{: toc-services="vpc, cloud object storage"}
{: toc-completion-time="120m"}

Deploy and configure Veeam Backup & Replication (VBR) on IBM Cloud VPC virtual server, with a two tier Scale-Out Backup Repository (SOBR) as backup destination.
{: shortdesc}

## Objectives
{: #veeam-vbr-vpc-objectives}

This tutorial guides you through implementing a comprehensive backup solution for {{site.data.keyword.vpc_short}} virtual server instances using Veeam Backup & Replication 13.

You learn how to build a production-ready backup infrastructure that combines local high-performance storage with cloud-based long-term retention, following industry best practices for data protection.

- Provision a Windows® virtual server for Veeam Backup & Replication server.
- Install and configure Veeam Backup & Replication.
- Set up {{site.data.keyword.cos_full_notm}} (Cloud Object Storage, COS) as a backup repository.
- Configure local disk repositories with Resilient File System (ReFS).
- Create a scale-out backup repository (SOBR) for tiered storage.
- Implement backup best practices for VPC environments.

## Before you begin
{: #veeam-vbr-vpc-prerequisites}

Before you begin, ensure that you meet the following prerequisites:

- An {{site.data.keyword.cloud_notm}} account with {{site.data.keyword.vpc_short}} access.
- Appropriate IBM Cloud Identity and Access Management (IAM) permissions to create and manage virtual private cloud (VPC) resources.
- A valid Veeam Backup & Replication license (trial or purchased).
- A basic understanding of backup and recovery concepts.
- Familiarity with Windows Server administration.

## Best practices
{: #veeam-vbr-vpc-best-practices}

Follow these best practices to optimize your Veeam Backup & Replication deployment in {{site.data.keyword.vpc_short}}. These recommendations cover storage configuration, network setup, and security to help ensure reliable and secure backup operations.

### Storage configuration
{: #veeam-vbr-vpc-storage-best-practices}

Use these storage recommendations to improve repository efficiency, reduce contention, and support faster restore operations.

- Use ReFS for local repositories to benefit from block cloning and increased resilience.
- Use a `64K` block size to optimize Veeam's backup format and improve performance.
- Use separate volumes for backup data and cache to reduce I/O contention.
- Monitor disk space regularly and adjust SOBR offload policies as needed.

### Network configuration
{: #veeam-vbr-vpc-network-best-practices}

Use these network practices to keep backup traffic efficient, private, and predictable across your VPC environment.

- Use private network connectivity within VPC for all backup traffic.
- Configure security groups to allow the following Veeam Transmission Control Protocol (TCP) ports:
   - `TCP 2500-3300`: Veeam Backup & Replication services
   - `TCP 6160-6163`: Veeam Agent communication
   - `TCP 9392-9394`: Veeam Backup Enterprise Manager (if used)
- Ensure adequate bandwidth between the VBR server and target virtual servers. A minimum of 1 Gbps is recommended.
- Use VPC routing to avoid using the internet gateway for backup traffic.

### Security
{: #veeam-vbr-vpc-security-best-practices}

Use these security practices to protect backup data, credentials, and administrative access throughout your deployment. These recommendations help reduce exposure while supporting secure day-to-day backup and restore operations.

- Enable encryption: Use backup encryption for data at rest and in transit.
- Secure credentials: Store Veeam credentials in a password manager.
- IAM policies: Apply least-privilege access to {{site.data.keyword.cos_short}} buckets.
- Network isolation: Use security groups to restrict VBR server access.
- Regular updates: Keep Veeam software updated with the latest patches.
- Audit logs: Enable and review Veeam audit logs regularly.

### Required IAM access policies
{: #veeam-vbr-vpc-access-policies}

Ensure that you have the following IAM access policies:

- VPC Infrastructure Services: Editor or Administrator role.
- {{site.data.keyword.cos_short}}: Writer or Manager role.
- Permission to create and manage the following resources:
   - Virtual servers
   - Block storage volumes
   - Security groups
   - Floating IPs

## Architecture overview
{: #veeam-vbr-vpc-architecture}

The solution architecture includes:

- Veeam Backup & Replication (VBR) server: A Windows virtual server running Veeam Backup & Replication.
- Local repository: Resilient File System (ReFS)-formatted block storage for fast backups.
- Cloud repository: {{site.data.keyword.cos_full_notm}} for long-term retention.
- Scale-out backup repository (SOBR): A scale-out backup repository that combines local and cloud tiers.

## Provisioning the VBR server instance
{: #veeam-vbr-vpc-provision-vsi}
{: step}

Complete the following steps to create a Windows virtual server to host the Veeam Backup & Replication server.

1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
2. Go to the VPC Infrastructure section. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
3. Click **Create** to create a new virtual server instance.
4. In the **New virtual server for VPC** form, specify the following details.
    1. In the **Location** section:
       - For **Geography**, select your preferred region, such as **North America**.
       - For **Region**, select your region, such as **Dallas**.
       - For **Zone**, select an availability zone, such as **us-south-1**.
    2. In the **Details** section, configure the instance details.
       - For **Name**, enter `veeam-vbr-server`.
       - For **Resource group**, select your resource group.
       - For **Tags**, optionally add tags such as `veeam` and `backup`.
    3. In the **Image** section, select the operating system.
       - For **Operating system**, select **Windows**.
       - For **Version**, select **Windows Server 2022 Standard Edition** or later.
    4. In the **Profile** section, select a profile based on your workload.
       - For **Profile**, select **Balanced** > `bx2-4x16` (4 vCPUs, 16 GB RAM) or a higher configuration based on your workload.
            - For small environments (< 10 VMs): `bx2-4x16`.
            - For medium environments (10-50 VMs): `bx2-8x32`.
            - For large environments (> 50 VMs): `bx2-16x64` or higher.
    5. In the **SSH keys** section, select or create an SSH key.
    6. In the **Data volumes** section, click **Create**.
       1. In the **Create block storage volume** form, enter `veeam-backup-volume` for **Name**.
       2. For **Profile**, select `10 input/output operations per second (IOPS)/GB` or higher.
       3. For **Size**, enter the size based on your backup requirements, such as `500 GB` for small environments or `2000 GB` for larger environments.
       4. Click **Save**.
       5. Click **Create** again to add a second volume.
       6. In the **Create block storage volume** form, enter `veeam-cache-volume` for **Name**.
       7. For **Profile**, select `5 IOPS/GB`.
       8. For **Size**, enter `100 GB`.
       9. Click **Save**.
    7. In the **Networking** section, for **Virtual private cloud**, either choose an existing VPC or create a new one.
    8. Click **Create virtual server**.
5. Wait for the virtual server instance to be provisioned and reach the **Running** status. This process takes approximately 5 - 10 minutes.
6. Reserve a floating IP for remote access.
   1. On the virtual server details page, in the **Networking** section, click the primary interface.
   2. Click **Attach** next to **Floating IP**.
   3. In the **Reserve new floating IP** form, enter `veeam-vbr-server-fip` for **Name**
   4. Click **Reserve**.
   5. Note the floating IP address for RDP access.
7. From the **Navigation menu**, click **Infrastructure** > **Network** > **Security groups**.
8. Click **Create**.
   1. Select **North America** for **Geography**.
   2. Select **Dallas (us-south)** for **Region**.
   3. Enter `veeam-vbr-sg` for **Name**.
   4. Choose your resource group.
   5. Choose your VPC.
   6. For **Security group rules**, add the following rules:
       - **Inbound**
          - **TCP** from your IP address to port `3389` for RDP.
          - **TCP** from the target virtual server instance IP addresses that you created earlier to ports `2500-3300` and `6160-6163`.
       - **Outbound**
          - **All** traffic to any port.
   7. For **Attaching virtual server interfaces**, choose the interface for the server `veeam-vbr-server`.
   8. Click **Create security group**.

## Connecting to the VBR server
{: #veeam-vbr-vpc-connect-vsi}
{: step}

Complete the following steps to connect to your Windows virtual server and retrieve the initial administrator password.

1. Get the initial administrator password.
   1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
   2. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   3. Click the `veeam-vbr-server` virtual server.
   4. Follow the [instructions](/docs/vpc?topic=vpc-vsi_is_connecting_windows) to retrieve the password.
2. Connect through Remote Desktop Protocol (RDP).
   1. Open your Remote Desktop Protocol (RDP) client (Remote Desktop Connection on Windows, Microsoft® Remote Desktop on macOS).
   2. For **Computer**, enter the floating IP address.
   3. For **Username**, enter `Administrator`.
   4. For **Password**, paste the password you retrieved.
   5. Click **Connect**, and then accept any certificate warnings.
3. After you connect, change the administrator password.
   1. Press **Ctrl+Alt+End** (or **Ctrl+Alt+Delete** if you are using Windows RDP client).
   2. Click **Change a password**.
   3. Enter the current password, and then set a new secure password.
   4. Click **OK**.

Open the required ports on the Windows virtual server. For more information, see the [Veeam documentation](https://helpcenter.veeam.com/docs/vbr/userguide/used_ports.html){: external} to determine which ports must be open. In this example, the Windows firewall is disabled.
{: note}

## Preparing the data volumes
{: #veeam-vbr-vpc-prepare-volumes}
{: step}

Complete the following steps to initialize and format the attached data volumes with ReFS for optimal Veeam performance.

1. Right-click the **Start** button.
2. Click **Disk Management**.
3. Initialize the first data disk for the backup volume.
   1. In the **Initialize Disk** dialog that appears, select **Disk 1**.
   2. For **Partition style**, select **GPT (GUID Partition Table)**.
   3. Click **OK**.
   4. If the dialog box does not appear automatically, right-click **Disk 1** when it shows **Unknown** and **Not Initialized**.
   5. Click **Initialize Disk**, and then complete the same selections.
4. Create a volume on the first data disk.
   1. Find the 500GB drive that was added.
   2. Right-click on the `Disk x` label (e.g., `Disk 2`) in the left pane and choose **Initialize**.
   3. In the **Initialize Disk** dialog, select **Disk x**, for **Partition style**, select **GPT (GUID Partition Table)**, and then click **OK**.
   4. Right-click the **Unallocated**  space on the disk and choose **New Simple Volume**.
   5. In the **New Simple Volume Wizard**:
      1. Click **Next**.
      2. For **Volume size**, accept the default volume size (maximum size).
      3. Click **Next**.
      4. For **Assign drive letter**, select **E**.
      5. Click **Next**.
      6. For **Format this volume with the following settings**, specify the following details:

          | Setting | Value |
          | -------- | -------- |
          | File system | ReFS |
          | Allocation unit size | `64K` |
          | Volume label | **VeeamBackup** |
          | Quick format | Select **Perform a quick format** |
          {: caption="Backup volume format settings" caption-side="bottom"}

      7. Click **Next**.
      8. Click **Finish**.
5. Initialize the second data disk for the cache volume.
   1. Locate the 100 GB drive.
   2. Right-click the `Disk x` label, such as `Disk 3`, in the left pane and select **Initialize Disk**.
   3. In the **Initialize Disk** dialog, select **Disk x**.
   4. For **Partition style**, select **GPT (GUID Partition Table)**.
   5. Click **OK**.
   6. Right-click the **Unallocated** space on **Disk x** and choose **New Simple Volume**.
   7. In the **New Simple Volume Wizard**:
      1. Click **Next**.
      2. Accept the default volume size.
      3. Click **Next**.
      4. For **Assign drive letter**, select **F**.
      5. Click **Next**.
      6. For **Format this volume**, specify:

         | Setting | Value |
         | -------- | -------- |
         | File system | ReFS |
         | Allocation unit size | `64K` |
         | Volume label | **VeeamCache** |
         | Quick format | Select **Perform a quick format** |
         {: caption="Cache volume format settings" caption-side="bottom"}

      7. Click **Next**.
      8. Click **Finish**.
6. Verify the volumes are formatted correctly.
   1. Open **File Explorer**.
   2. Verify that drives **E:** (VeeamBackup) and **F:** (VeeamCache) are visible.
   3. Check that both show **ReFS** as the file system.

## Downloading Veeam Backup & Replication
{: #veeam-vbr-vpc-download-veeam}
{: step}

Complete the following steps to download the installation files for Veeam Backup & Replication.

1. On the VBR server, open **Microsoft Edge** or **Internet Explorer**.
2. Go to the [Veeam Downloads page](https://www.veeam.com/products/downloads.html){: external}.
3. Download Veeam Backup & Replication.
   1. Scroll to **Veeam Backup & Replication**.
   2. Click **Download** for the latest version.
   3. If prompted, create a free Veeam account or log in.
   4. Select **Download** for the ISO file.
   5. Save the ISO file to `C:\Veeam\VeeamBackupReplication.iso` (create the folder if needed).
4. Download the Veeam license file.
   - If you have a purchased license:
      1. Log in to the Veeam account portal.
      2. Download the license file.
      3. Save the license file to `C:\Veeam\license.lic`.
   - If you are using a trial license:
      1. Request a trial license from the Veeam website.
      2. Check your email for the license file.
      3. Save the license file to `C:\Veeam\license.lic`.

## Installing Veeam Backup & Replication
{: #veeam-vbr-vpc-install-veeam}
{: step}

Complete the following steps to install Veeam Backup & Replication on the Windows virtual server.

1. Mount the Veeam ISO file.
   1. Open **File Explorer**.
   2. Go to **Downloads**.
   3. Right-click **VeeamBackupReplication.iso**.
   4. Click **Mount**.
   5. Note the assigned drive letter, such as `D:`.
2. Run the Veeam installer.
   1. Open the mounted ISO drive in File Explorer.
   2. Double-click **Setup.exe**.
   3. If prompted by User Account Control, click **Yes**.
3. Complete the installation wizard.
   1. On the **Veeam Backup & Replication** installer, click **Install**.
   2. On the **License Agreement** page, read and accept the license agreement, and then click **I Accept**.
   3. On the **Program Features** page, ensure that all features are selected, and then click **Next**.
   4. On the **License** page, click **Browse to local license file**.
      1. Go to `C:\Veeam\license.lic`.
      2. Click **Open**, and then click **Next**.
   5. On the **Service Account** page, select the Windows user account that runs the service, and then click **Next**.
   6. On the **System Configuration Check** page, review any warnings or errors.
       1. If a restart is needed, click **Reboot**, and then click **Yes**.
       2. Wait for the virtual server to restart, and then restart the Veeam installer.
       3. If all checks pass, click **Next**.
   7. On the **Ready to Install** page, click **Customize Settings**.
      1. In **SQL Server Settings**, accept the default settings, and then click **Next**.
      2. In the **Reporting Database**, accept the default settings, and then click **Next**.
      3. In **Database Names**, accept the default settings, and then click **Next**.
      4. For the **Data Locations**:
         1. For the **Installation Path**, choose the **C:\\\\** drive.
         2. For the **Path for cache and catalog files**, choose the **VeeamCache** drive.
         3. Click **Next**.
         4. In **Port Configuration**, accept the default settings, and then click **Next**.
         5. In **Certification Selection**, accept the default settings, and then click **Next**.
      5. Click **Install**.
   8. Wait for the installation to finish. This process might take some time to complete.
   9. Click **Finish**.
      - If a restart is needed, click **Yes**, and then wait for the virtual server to restart.
      - If a restart is not needed, click **Close**.
4. Verify that the Veeam Backup & Replication Console starts automatically.

## Creating an IBM Cloud Object Storage instance
{: #veeam-vbr-vpc-create-cos}
{: step}

Complete the following steps to create {{site.data.keyword.cos_full_notm}} instance for long-term backup retention.

1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
2. From the **Navigation menu**, click **Infrastructure** > **Storage** > **Object storage**.
3. Click **Create an instance**.
4. On the **Create** page, configure the {{site.data.keyword.cos_full_notm}} instance settings.
   1. For **Select a pricing plan**, select **Standard** (pay-as-you-go).
   2. For **Service name**, enter `veeam-backup-cos`.
   3. For **Resource group**, select your resource group.
   4. For **Tags**, optionally add tags such as `veeam` and `backup`.
   5. Click **Create**.
5. Create a bucket for Veeam backups. From the Cloud Object Storage instance page, click **Buckets** in the left navigation
   1. Click **Create bucket**.
   2. Click **Create a Custom bucket**.
   3. In the **Create a bucket** form, configure the following bucket settings.
       1. For **Bucket name**, enter `veeam-vpc-backups`. The name must be globally unique.
       2. For **Resiliency**, select **Regional**.
       3. For **Location**, select the same region as your VBR server, such as **us-south**.
       4. For **Storage class**, select **Standard**.
       5. Click **Create bucket**.
6. Generate HMAC credentials for Veeam. From the Cloud Object Storage instance page, click **Service credentials** in the left navigation.
   1. Click **New credential**.
   2. In the **Create Credentials** form, configure the credential settings.
       1. For **Control by Secrets Manager**, select **Off**.
       2. For **Name**, enter `veeam-hmac-credentials`.
       3. For **Role**, select **Writer**.
       4. Click **Advanced options**.
       5. For **Include HMAC Credential**, set the toggle to **On**.
   3. Click **Add**.
   4. On the newly created credential, click **View credentials**.
   5. Copy and securely save the following values:
       - `access_key_id`
       - `secret_access_key`
       - Endpoint URL from the bucket details, such as `s3.us-south.cloud-object-storage.appdomain.cloud`
       - Bucket name: `veeam-vpc-backups`

       Alternatively, you can use a direct endpoint such as `s3.direct.us-south.cloud-object-storage.appdomain.cloud`. This option requires a VPE.
       {: note}

## Configuring IBM Cloud Object Storage in Veeam
{: #veeam-vbr-vpc-configure-cos-repo}
{: step}

Complete the following steps to add {{site.data.keyword.cos_full_notm}} as a backup repository in Veeam.

1. Open the Veeam Backup & Replication Console if it is not already open.
   1. From the **Start** menu, search for **Veeam Backup & Replication Console** and click it to open.
   2. Click **Connect** to localhost.
   3. Click **Yes** to **Trust this server?**
   4. Sign in by using the current user account.
   5. In the left navigation pane, click **Backup Infrastructure**.
2. Add a new object storage repository. In the **Backup Infrastructure** view, right-click **Backup Repositories**.
   1. Click **Add Backup Repository**.
   2. On the **Backup Repository** page, click **Object storage**.
   3. On the **Object Storage** page, select **S3 Compatible**.
   4. On the **S3 Compatible** page, select **S3 Compatible**.
   5. On the **Name** page, configure the repository details.
      1. For **Name**, enter `IBM-COS-Repository`.
      2. For **Description**, enter `{{site.data.keyword.cos_full_notm}} for long-term backup retention`.
      3. Click **Next**.
   6. On the **Account** page, enter the account connection details.
      1. For **Service point**, enter the {{site.data.keyword.cos_full_notm}} endpoint, such as `s3.us-south.cloud-object-storage.appdomain.cloud`.
      2. For **Region**, keep the field blank or enter the region code, such as `us-south`.
      3. Click **Add** next to **Credentials**.
      4. In the **S3 Compatible Account** form:
         1. For **Access key**, paste the `access_key_id` from your HMAC credentials.
         2. For **Secret key**, paste the `secret_access_key` from your HMAC credentials.
         3. For **Description**, enter `IBM COS HMAC Credentials`.
         4. Click **OK**.
      5. Click **Next**.
   7. On the **Bucket** page, click **Browse**.
      1. Select `veeam-vpc-backups` from the list.
      2. Click **OK**.
      3. For **Folder**, click **Browse**.
      4. Click **New Folder**.
      5. Enter `backups` as the folder name.
      6. Select the new folder and click **OK**.
      7. Click **Next**.
   8. On the **Mount Server** page, select the VBR server from the list, and then click **Next**.
   9. On the **Review** page, review the configuration, and then click **Apply**.
   10. Wait until the repository is created.
   11. Click **Finish**.

## Adding a local disk repository
{: #veeam-vbr-vpc-add-local-repo}
{: step}

Complete the following steps to configure the local ReFS volume as a backup repository for fast backups and restores.

1. In the Veeam Backup & Replication Console, go to **Backup Infrastructure**.
2. Add a new backup repository. Right-click **Backup Repositories**.
   1. Click **Add Backup Repository**.
   2. In the **New Backup Repository** wizard. On the **Backup Repository** page, click **Direct attached storage**.
   3. On the **Direct Attached Storage** page, select **Microsoft Windows**.
   4. On the **Name** page, configure the repository details.
      1. For **Name**, enter `Local-Backup-Repository`.
      2. For **Description**, enter `Local ReFS repository for fast backups`.
      3. Click **Next**.
   5. On the **Server** page, select the VBR server from the list.
      1. Click **Populate**.
      2. Select the 500 GB drive that you added to the VBR server.
      3. Click **Next**.
   6. On the **Repository** page, click **Browse**.
      1. Browse to `E:\VeeamBackup`.
      2. Create the folder if it does not exist.
      3. Click **OK**.
      4. In **Load control**, set **Limit concurrent tasks** to `4`. Adjust this value based on your server resources.
      5. Click **Customize repository settings...**.
      6. In the **Storage Compatibility Settings** form:
         1. For **Decompress backup data blocks before storing**, select **Enabled** (improves performance with ReFS).
         2. For **Align backup file data blocks**, select **Enabled**.
         3. Click **OK**.
      7. Click **Next**.
   7. On the **Mount Server** page, select the VBR server, and then click **Next**.
   8. On the **Review** page, review the configuration, and then click **Apply**.
   9. Wait until the repository is created.
   10. Click **Finish**.

## Creating a Scale-Out Backup Repository (SOBR)
{: #veeam-vbr-vpc-create-sobr}
{: step}

Complete the following steps to create a Scale-Out Backup Repository (SOBR) that combines local and cloud storage tiers.

1. In the Veeam Backup & Replication Console, go to **Backup Infrastructure**.
2. Right-click **Scale-out Repositories**.
3. Click **Add Scale-out Backup Repository**.
4. In the **New Scale-out Backup Repository** wizard:
   1. On the **Name** page, enter `SOBR-VPC-Backups` for **Name**.
      1. For **Description**, enter `Scale-out repository with local performance tier and cloud capacity tier`
      2. Click **Next**.
   2. On the **Performance Tier** page, click **Add**.
      1. Select `Local-Backup-Repository`.
      2. Click **OK**.
      3. Click **Next**.
   3. On the **Placement policy** page, select **Data locality** to keep backups on the same extent, and then click **Next**.
   4. On the **Capacity Tier** page, select **Extend scale-out backup repository capacity with object storage**.
      1. Click **Choose**.
      2. Select `IBM-COS-Repository`.
      3. Click **OK**.
      4. Keep **Copy backups to object storage as soon as they are created** cleared unless you want an immediate offsite copy.
      5. For **Move backups to object storage**, set **Operational restores** to `7 days`. Adjust the value if needed to match your recovery time objective (RTO) requirements.
      6. Click **Next**.
   5. On the **Archive Tier** page, keep the option cleared unless you are using S3 Glacier, and then click **Apply**.
   6. On the **Summary** page, review the SOBR configuration, and then click **Finish**.
   7. Wait until the SOBR is created.

## Support resources
{: #veeam-vbr-vpc-support-resources}

Use the following resources for additional information when you work with Veeam Backup & Replication and {{site.data.keyword.cloud_notm}} services.

- [Veeam Help Center](https://helpcenter.veeam.com/){: external}
- [{{site.data.keyword.vpc_short}} documentation](/docs/vpc)
- [{{site.data.keyword.cos_short}} documentation](/docs/cloud-object-storage)
- [Veeam Community Forums](https://forums.veeam.com/){: external}
- [Veeam Best Practices](https://bp.veeam.com/vbr/){: external}

## Next steps
{: #veeam-vbr-vpc-next-steps}

After you complete this tutorial, consider the following next steps:

- Perform backups: Protect workloads by creating backup jobs.
- Configure replication: Set up Veeam replication for additional protection.
- Implement monitoring: Integrate with monitoring tools (Veeam ONE, Grafana, and so on).
- Automate operations: Create PowerShell scripts for common tasks.
- Plan disaster recovery: Document and test DR procedures.
- Optimize costs: Review and adjust retention policies and SOBR settings.

## Related links
{: #veeam-vbr-vpc-related-content}

Now that you understand how to configure Veeam Backup & Replication for {{site.data.keyword.vpc_short}} virtual servers, explore the following related topics:

- [Troubleshooting Configuring VBR with VSI](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-vbr-vpc-vsi-integration-configuration)
- [{{site.data.keyword.vpc_short}} overview](/docs/vpc?topic=vpc-about-vpc)
- [{{site.data.keyword.cos_short}} overview](/docs/cloud-object-storage?topic=cloud-object-storage-about-cloud-object-storage)
- [Veeam Backup & Replication User Guide](https://helpcenter.veeam.com/docs/vbr/userguide/overview.html){: external}
