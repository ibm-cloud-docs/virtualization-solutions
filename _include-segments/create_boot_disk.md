## Migrating a RHEL virtual server disk to IBM Cloud VPC
{: #virt-sol-vpc-migration-tutorial-migrate-rhel-boot-disk}
{: step}

Transfer a RHEL virtual server disk to an IBM Cloud VPC block storage volume and attach it to the worker virtual server to receive the migrated boot disk.
{: shortdesc}

When the virtual server starts, Cloud-init runs and the root password resets.
{: note}

1. Log in to the IBM Cloud console.
1. Create a detached RHEL boot volume by completing the following steps:
   1. From the **Navigation menu**, click **Infrastructure** > **Network** > **Subnets**.
   1. From the list of available resources, select **vpc-migration-sn-1**.
   1. Click **Attached resources**.
   1. In the **Attached instances** section, click **Create**.
   1. In the **Location** section, specify the following information:
      - For **Geography**, select **North America**.
      - For **Region**, select **Dallas (us-south)**.
      - For **Zone**, select **us-south-1**.
   1. In the **Details** section, for **Name**, enter `vpc-migration-vsi-rhel9`.
   1. In the **Server configuration** section, specify the following information:
      1. For **Image**, click **Change image**.
      1. From the **Select an image** form, specify the following information:
         1. Search for **Red Hat**.
         1. Select **ibm-redhat-9-6-minimal-amd64-5** from the list of results and click **Save**.
      1. For **SSH keys**, select **vpc-migration-ssh-key**.
   1. In the **Storage** section, click the **Edit** icon that is next to the item for the boot volume.
   1. Within the **Edit boot volume** form, specify the following information:
       1. In the **Details** section, specify the following information:
          1. For **Name**, enter `vpc-migration-vsi-rhel9-boot-volume`.
          1. Set **Auto-delete** to **Disabled**.
       1. In the **Profiles** section, select **General purpose**.
       1. In the **Storage capacity** section, for **Storage size**, enter a value that is greater than or equal to the storage size of the RHEL virtual server and click **Save**.
   1. Click **Create a virtual server**.
   1. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   1. From the list of available resources, select **vpc-migration-vsi-rhel9**.
   1. Click **Delete**. In the **Delete virtual server instance** form, complete the following steps:
       1. Enter **Delete** into the input.
       1. Click **Delete**.
1. Attach the boot volume to the worker virtual server by completing the following steps:
   1. From the **Navigation menu**, click **Infrastructure** > **Storage** > **Block storage volumes**.
   1. In the list of available resources, click **vpc-migration-vsi-rhel9-boot-volume**.
   1. In the **Attached virtual server** section, click **Attach**.
   1. Within the **Attach to a virtual server** form, from the list of virtual servers, select **vpc-migration-vsi-worker**, and click **Attach**.
1. SSH into the worker virtual server by running the command that you used previously.
1. Get the name of the block device for the attached boot volume by running the following command:

   ```sh
   lsblk -n -d -o NAME | tail -n 1
   ```
   {: pre}

1. Start the Netcat server, wait for an incoming disk transfer, and then write it to the attached boot volume by running the following command:

   ```sh
   nc -l 31337 | gunzip | dd of=/dev/<DEV_NAME> bs=16M status=progress
   ```
   {: pre}

   Where

   `<DEV_NAME>`
   :   The name of the block device that you found previously.
