---

copyright:
  years: 2026
lastupdated: "2026-05-20"

keywords: Veeam Backup & Replication, VBR, restore, recovery, IBM Cloud VPC, VSI, volume restore, disaster recovery

subcollection: virtualization-solutions

content-type: tutorial
services: vpc
account-plan: paid
completion-time: 90m

---

{{site.data.keyword.attribute-definition-list}}

# Restoring IBM Cloud VPC Virtual Server Instances with Veeam Backup & Replication
{: #veeam-vbr-vpc-vsi-restore}
{: toc-content-type="tutorial"}
{: toc-services="vpc"}
{: toc-completion-time="90m"}

This tutorial provides step-by-step instructions for restoring {{site.data.keyword.vpc_full}} virtual server instances by using Veeam Backup & Replication (VBR) 13 with agents.

It simulates a disaster recovery scenario where the original virtual server instance is deleted, its volumes are preserved, and a temporary virtual server instance is used to restore the data to the original volumes.
{: shortdesc}

## Objectives
{: #veeam-vbr-vpc-restore-objectives}

This tutorial covers the main recovery tasks that are required to restore preserved volumes and rebuild a virtual server instance after a failure. You also learn how the temporary worker virtual server instance supports the restore workflow.

In this tutorial, you learn how to:

- Set up a temporary worker virtual server instance for restore operations
- Perform volume-level restores for Windows and Linux virtual server instances
- Restore data to preserved volumes by using a temporary virtual server instance
- Restore boot and data volumes from Veeam backups
- Recreate the original virtual server instance with restored volumes
- Reconfigure Veeam agents after restore

By the end of this tutorial, you complete a full disaster recovery scenario that includes deleting a virtual server instance, restoring its volumes, and bringing it back online with its data intact.

## Before you begin
{: #veeam-vbr-vpc-restore-prerequisites}

Before you begin, verify that you meet the following prerequisites:

- A working VBR 13 installation with configured repositories (Scale-Out Backup Repository or SOBR using local disk and {{site.data.keyword.cos_full}}).
- At least one successful backup of a Windows or Linux virtual server instance.
- A temporary virtual server instance with a Veeam agent installed (generic agent, not preinstalled).
- Appropriate Identity and Access Management (IAM) permissions to create and manage Virtual Private Cloud (VPC) resources.
- Network connectivity between the Veeam server and target virtual server instances.
- Familiarity with the [Veeam VBR configuration tutorial](/docs/virtualization-solutions?topic=virtualization-solutions-veeam-vbr-vpc-vsi-configuration).

## Best practices
{: #veeam-vbr-vpc-restore-best-practices}

Follow these practices to make restore operations more predictable and reduce the risk of delays during recovery. They help you validate backups, preserve access, and confirm that restored systems are ready for production use.

1. Save credentials (passwords, SSH keys) before you delete the original virtual server instance.
2. Test restores regularly to verify your backup strategy works.
3. Use generic agents for templates and restore scenarios because they are more flexible than preinstalled agents.
4. Document your restore procedures that are specific to your environment.
5. Automate where possible to reduce human error and restore time.
6. Monitor restore progress to identify issues early.
7. Verify restored data before you decommission the temporary virtual server instance.
8. Keep the temporary virtual server instances until you confirm the restored virtual server instance is fully functional.
9. Update the Veeam agent configuration immediately after restore to verify future backups work correctly.

### Required access policies
{: #veeam-vbr-vpc-restore-access-policies}

Verify that you have the following access policies:

- VPC Infrastructure Services: Editor or Administrator role
- Ability to manage:
    - Virtual server instances
    - Block storage volumes
    - Security groups
    - Floating IP addresses

## Restore scenario overview
{: #veeam-vbr-vpc-restore-scenario}

The section outlines the restore workflow that is completed in this tutorial. It shows how the temporary worker virtual server instance, preserved volumes, and Veeam restore operations work together in a disaster recovery scenario.

1. Verify that the original virtual server instance contains boot and data volumes that are backed up to Veeam.
2. Delete the original virtual server instance while you preserve its volumes.
3. Deploy a temporary virtual server instance with the Veeam agent installed.
4. Attach the original volumes to the temporary virtual server instance.
5. Restore volumes by using Veeam agent on a temporary virtual server instance.
6. Detach restored volumes from the temporary virtual server instance.
7. Recreate the original virtual server instance by using the restored volumes.
8. Reconfigure the Veeam agent on the restored virtual server instance.

This approach enables data restoration without losing the original storage volumes, making it useful for disaster recovery scenarios.

### Creating a Windows worker virtual server instance
{: #veeam-vbr-vpc-restore-temp-vsi-windows}

Open the required ports on the Windows virtual server instance. For more information, see [Veeam's documentation](https://helpcenter.veeam.com/docs/vbr/userguide/used_ports.html){: external}. In this example, the Windows Firewall is disabled.
{: note}

1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
2. From the **Navigation menu**, go to **Infrastructure** > **Compute** > **Virtual server instances**.
3. Create a new virtual server instance.
   1. Click **Create**.
   2. In the **New virtual server for VPC** form, specify the following information:
      1. In the **Location** section:
         - For **Geography**, select your preferred region (for example, **North America**).
         - For **Region**, select your region (for example, **Dallas**).
         - For **Zone**, select an availability zone (for example, **us-south-1**).
      2. In the **Details** section:
         - For **Name**, enter `windows-worker-server`.
         - For **Resource group**, select your resource group.
         - For **Tags**, optionally add tags such as `veeam`, `backup`.
      3. In the **Image** section:
         - For **Operating system**, select **Windows**.
         - For **Version**, select **Windows Server 2022 Standard Edition** or a later version.
      4. In the **Profile** section:
         - For **Profile**, select **Flex** > **bxf-2x8** (2 vCPUs, 8 GB RAM) or a higher profile size based on your workload requirements.
      5. In the **SSH keys** section, select or create an SSH key.
      6. In the **Networking** section:
         1. For **Virtual private cloud**, choose the VPC that you want to restore from.
         2. Click edit on the `eth0`.
            1. Click **Next**.
            2. In the **Network** panel, under **Security Groups** select `veeam-vbr-sg`.
            3. Click **Next**.
            4. Click **Next**.
            5. Click **Save**.
      7. Click **Create virtual server**.
4. Wait for the virtual server instance to reach the **Running** status. This process usually takes 5 - 10 minutes.

## Creating a boot volume for restored Windows virtual server instance
{: #veeam-vbr-vpc-restore-windows}
{: step}

1. Delete the original Windows virtual server instance.
2. Deploy a virtual server instance to create the boot volume.
   1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
   2. From the **Navigation menu**, go to **Infrastructure** > **Compute** > **Virtual server instances**.
   3. Create a new virtual server instance.
      1. Click **Create**.
      2. In the **New virtual server for VPC** form, specify the following information:
         1. In the **Location** section:
            - For **Geography**, select your preferred region (for example, **North America**).
            - For **Region**, select your region (for example, **Dallas**).
            - For **Zone**, select an availability zone (for example, **us-south-1**).
         2. In the **Details** section:
            - For **Name**, enter `windows-temp-server`.
            - For **Resource group**, select your resource group.
            - For **Tags**, optionally add tags such as `veeam`, `backup`.
         3. In the **Image** section:
            - For **Operating system**, select **windows**.
            - For **Version**, select **ibm-windows-server-2022-full-standard-amd64-34** or a later version.
         4. In the **Profile** section:
            - For **Profile**, select **Flex** > **bxf-2x8** (2 vCPUs, 8 GB RAM) or a higher profile size based on your workload requirements.
         5. In the **SSH keys** section, select or create an SSH key.
         6. In the **Storage** section:
            1. Under **Boot Volume**, click **edit**.
               1. For **Name**, enter `windows-recovery-volume`.
               2. For **Auto-delete**, click **Disabled**.
               3. For **Storage Size**, select the same size as the volume that you are restoring.
               4. Click **Save**.
         7. In the **Networking** section:
            1. For **Virtual private cloud**, select the VPC that contains the resources that you want to restore.
         8. Click **Create virtual server**.
   4. Delete the new virtual server instance.
   5. Attach the volume to the worker virtual server instance.
      1. Go to the virtual server instance named `windows-worker-server`.
      2. In the **Storage** tab, click **Actions** > **Attach**, and then select **windows-recovery-volume**.
      3. Click **Save**.

## Formatting the volume on the Windows worker virtual server instance
{: #veeam-vbr-vpc-connect-vsi}
{: step}

Follow these steps to connect to your Windows worker virtual server instance and retrieve the initial administrator password.

1. Get the initial administrator password.
   1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
   2. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   3. Click the virtual server instance named **windows-worker-server**.
   4. Follow the [instructions](/docs/vpc?topic=vpc-vsi_is_connecting_windows) to retrieve the password.
2. Connect by using Remote Desktop Protocol (RDP).
   1. Open your RDP client (Remote Desktop Connection on Windows, Microsoft Remote Desktop on macOS).
   2. For **Computer**, enter the floating IP address.
   3. For **Username**, enter `Administrator`.
   4. For **Password**, paste the password you retrieved.
   5. Click **Connect**, and then accept any certificate warnings.
3. When connected, change the administrator password.
   1. Press **Ctrl+Alt+End** (or **Ctrl+Alt+Delete** if you are using Windows RDP client).
   2. Click **Change a password**.
   3. Enter the current password, and then set a new secure password.
   4. Click **OK**.
4. Open the Windows command prompt to format the attached volume.
   1. Enter `diskpart`.
   2. Enter `list disk`.
      - Identify the new disk. It is typically the last disk that you added, and its size matches the attached block storage that is configured in the previous steps.
   3. Enter `select  disk X`. For example, `select disk 3`.
   4. Enter `list part`.
      - Identify the partition with type=system
      - Identify the partition with type=recovery
      - Identify the next two steps for both types.
   5. Enter `select part X`, where X is the partition number that is identified in the previous step.
   6. Enter `delete part override` to remove the selected partition.
        You can delete all partitions if needed.
        {: note}

5. Initialize the new attached disk.

Open the required ports on the Windows virtual server instance. For more information, see [Veeam's documentation](https://helpcenter.veeam.com/docs/vbr/userguide/used_ports.html){: external}. In this example, the Windows Firewall is disabled.
{: note}

6. Record the private address of the Windows worker virtual server instance.

## Connecting the Windows worker virtual server instance to VBR
{: #veeam-vbr-vpc-connect-windows-vbr}
{: step}

1. In the Veeam Backup & Replication console, go to **Inventory**.
2. Create a new protection group for Windows servers.
   1. In the left navigation pane, right-click **Physical & Cloud Infrastructure**.
   2. Click **Add protection group**.
   3. Choose **Individual computers**.
   4. In the **New Protection Group** wizard, specify the following details:
      1. On the **Name** page:
         1. For **Name**, enter `Windows-Worker-Restore-Group`.
         2. For **Description**, enter `Windows worker VSIs in {{site.data.keyword.cloud_notm}} VPC for restore`.
         3. Click **Next**.
      2. On the **Computers** page:
         1. Click **Add**.
         2. In the **Add Computer** form:
            1. For **DNS name or IP address**, enter the private IP address of your Windows worker virtual server instance.
            2. For **Connect using admin credentials**, click **Add**.
               1. For **Username**, enter `Administrator` or the domain account username.
               2. For **Password**, enter the administrator password.
               3. For **Description**, enter `Windows VSI Admin Credentials`.
               4. Click **OK**.
            3. Click **OK**.
         3. Click **Next**.
      3. On the **Options** page:
         1. Select the checkbox for **Install backup agent**.
         2. Select the checkbox for **Install change block tracking driver**.
         3. Select the checkbox for **Enable auto-update for installed components**.
         4. Select the checkbox for **Perform reboot automatically if required**.
         5. Click **Next**.
      4. On the **Review** page:
         1. Review the computer details.
         2. Click **Apply**, if needed.
         3. Repeat steps 1-4 to add more Windows virtual server instances to the protection group.
         4. Click **Next**.
      5. On the **Apply** page (if you want to apply any changes):
          - Click **Next**.
      6. On the **Summary** page, click **Finish**.
3. Wait for **Machine rescan** to finish.
4. Verify that the server was added:
   1. In the **Inventory** pane, expand **Physical & Cloud Infrastructure** > **Protection Groups**.
   2. Expand **Windows-Worker-Restore-Group**.
   3. Verify that the Windows worker virtual server instance is listed with a green checkmark.

## Restoring Windows volumes by using VBR
{: #veeam-vbr-vpc-restore-windows-volumes}

1. In the Veeam Backup & Replication console, click **Backups**.
2. Right-click the Windows virtual server instance, and then select **Restore Volumes...**
   1. In the **Restore point** page, select the restore point.
   2. Click **Next**.
   3. In the **Disk Mapping** page:
      1. For **Destination host**, select the Windows worker instance under the protection group **Windows-Worker-Restore-Group**.
      2. Click **OK**.
      3. Click **Customize disk mapping**.
         1. Map the disk to the same disk that you configured earlier.
            - Right-click a partition, and then select **remove**.
            - Right-click the same partition and choose the **restore** > **Disk 0**.
         2. Click **OK**.
      4. Click **Next >**.
   4. In the **Secure Restore**, click **Next >**.
   5. In the **Reason**, click **Next**.
   6. In the **Summary**, click **Finish**.
3. Wait for the task to finish.

## Restoring the Windows virtual server instance
{: #veeam-agent-vpc-restore-windows-volumes}

1. Log in to the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com){: external}.
2. Remove the attached block storage volume.
   1. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   2. Locate the Windows virtual server instance named **windows-worker-server**.
   3. Go to the **Storage** tab.
   4. Locate the **windows-recovery-volume** storage volume, and then select **Detach Volume**.
   5. Click **Detach**.
   6. Wait until the volume is detached.
3. Deploy a new virtual server instance with the restored volume:
   1. From the **Navigation menu**, click **Infrastructure** > **Storage** > **Block storage volumes**.
   2. Locate the **windows-recovery-volume** storage volume, and then click **Attach +**.
   3. In the **Attach to virtual server**, click **Create server** > **Attach as boot volume**.
   4. In the **Virtual server for VPC** panel:
      1. For **Name**, enter `windows-recovered-server`.
      2. For **Resource group**, select your resource group.
      3. For **SSH keys**, select your SSH key.
      4. For **Virtual private cloud**, select your VPC.
      5. Click **Create virtual server**.
   5. Wait for the virtual server instance deployment to complete and for the status to change to **Running**.
   6. Log in to the Windows virtual server instance by using the password from the original virtual server instance.
4. Register the new virtual server instance in VBR.

## Linux-based restore
{: #veeam-vbr-vpc-vsi-restore-linux}

The Linux restore process is similar to the Windows restore process but has some differences:

1. The Linux worker uses the stand-alone agent. Follow these [instructions](https://helpcenter.veeam.com/docs/agentforlinux/userguide/installation_process.html){: external} to install the agent on the {{site.data.keyword.redhat_full}} worker virtual server instance.
2. Perform a volume-based restore of the boot volume on the Linux worker virtual server instance. Follow steps 3 - 9 in this [document](https://helpcenter.veeam.com/docs/agentforlinux/userguide/baremetal_volume_restore.html){: external}.

## Next steps
{: #veeam-vbr-vpc-restore-next-steps}

After you complete this tutorial, review the following related resources to strengthen your backup and recovery strategy in {{site.data.keyword.vpc_short}}. The following resources help you refine your Veeam configuration, operational practices, and validation approach.

- [Troubleshooting Restoring VBR with VSI](/docs/virtualization-solutions?topic=virtualization-solutions-troubleshooting-vbr-vpc-vsi-integration-restore).
- Review the [Veeam VBR configuration tutorial](/docs/vpc?topic=vpc-vsi_best_practices).
- Explore [Veeam Backup & Replication 13 documentation](https://helpcenter.veeam.com/docs/vbr/userguide/overview.html?ver=13){: external}.
- Learn about [{{site.data.keyword.cloud_notm}} VPC best practices](/docs/vpc?topic=vpc-vsi_best_practices)
- Set up [automated backup testing and validation](https://helpcenter.veeam.com/docs/vbr/userguide/surebackup_hiw.html?ver=13){: external}
