## Creating the boot disk for the RHEL virtual server
{: #virt-sol-vpc-migration-tutorial-create-rhel-boot-disk}
{: step}

Use the following information to transfer a RHEL virtual server disk to your VPC and create a virtual server from the migrated disk.

When the virtual server starts, Cloud-init runs and the root password resets.
{: note}

1. Log in to the IBM Cloud console.
2. Create a detached RHEL boot volume by specifying the following information:
   1. From the **Navigation menu**, click **Infrastructure** > **Network** > **Subnets**.
   2. From the list of available resources, select **vpc-migration-sn-1**.
   3. Click **Attached resources**
   4. In the **Attached instances** section, click **Create**.
   5. In the **Location** section, specify the following information:
      - For **Geography**, select **North America**.
      - For **Region**, select **Dallas (us-south)**.
      - For **Zone**, select **us-south-1**.
   6. In the **Details** section, for **Name**, enter `vpc-migration-vsi-rhel9`.
   7. In the **Server configuration** section, specify the following information:
      1. For **Image**, click **Change image**.
      2. From the **Select an image** form, specify the following information:
         1. Search for **Red Hat**.
         2. Select **ibm-redhat-9-6-minimal-amd64-5** from the list of results and click **Save**.
      3. For **SSH keys**, select **vpc-migration-ssh-key**.
   8. In the **Storage** section, click the **Edit** icon that is next to the item for the boot volume.
   9. Within the **Edit boot volume** form, specify the following information:
       1. In the **Details** section, specify the following information:
          1. For **Name**, enter `vpc-migration-vsi-rhel9-boot-volume`
          2. Set **Auto-delete** to **Disabled**
       2. In the **Profiles** section, select **General purpose**.
       3. In the **Storage capacity** section, for **Storage size**, enter a value that is greater than or equal to the storage size of the RHEL virtual server and click **Save**.
   10. Click **Create a virtual server**.
   11. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   12. From the list of available resources, select **vpc-migration-vsi-rhel9**.
   13. Click **Delete**. In the **Delete virtual server instance** form,
       1. Enter **Delete** into the input.
       2. Click **Delete**.
3. Attach the boot volume to the worker virtual server by specifying the following information:
   1. From the **Navigation menu**, click **Infrastructure** > **Storage** > **Block storage volumes**.
   2. In the list of available resources, click **vpc-migration-vsi-rhel9-boot-volume**.
   3. In the **Attached virtual server** section, click **Attach**.
   4. Within the **Attach to a virtual server** form, from the list of virtual servers, select **vpc-migration-vsi-worker**, and click **Attach**.
4. SSH into the worker virtual server by running the command that you used previously.
5. Get the name of the block device for the attached boot volume by running the following command:

    `lsblk -n -d -o NAME | tail -n 1`

6. Start the Netcat server, wait for an incoming disk transfer, then write it to the attached boot volume by running the following command:

    `nc -l 31337 | gunzip | dd of=/dev/<DEV_NAME> bs=16M status=progress`

    Where `<DEV_NAME>` is the name of the block device that you found previously.
