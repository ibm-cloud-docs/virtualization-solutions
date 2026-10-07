## Creating a RHEL virtual server from a boot volume in IBM Cloud
{: #virt-sol-vpc-migration-tutorial-start-rhel-vm}
{: step}

After you transfer the RHEL virtual server disk to IBM Cloud, create a new virtual server instance by using the migrated boot volume as the startup disk.
{: shortdesc}

1. Log in to the IBM Cloud console.
1. From the **Navigation menu**, click **Infrastructure** > **Storage** > **Block storage volumes**.
1. From the list of available resources, find and click **vpc-migration-vsi-rhel9-boot-volume**.
1. In the **Attached virtual server** section, click the **Detach** icon next to the name of the worker virtual server. Wait for the worker virtual server to detach.
1. In the **Attached virtual server** section, click **Attach**. Within the **Attach to the virtual server** form, specify the following information:
   1. Click **Create server**.
   1. Click **Attach as boot volume**.
   1. In the **Location** section, specify the following information:
      - For **Geography**, select **North America**.
      - For **Region**, select **Dallas (us-south)**.
      - For **Zone**, select **us-south-1**.
   1. In the **Details** section, for **Name**, enter `vpc-migration-vsi-rhel9`.
   1. In the **Server configuration** section, for **SSH Keys**, select **vpc-migration-ssh-key**.
   1. Click **Create a virtual server**.
1. Get the IP address of the RHEL virtual server by completing the following steps:
   1. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   1. Search for **vpc-migration-vsi-rhel9**.
   1. From the list of results, copy the reserved IP address of the RHEL virtual server.
1. Log in to the RHEL virtual server by running the following SSH command:

   ```sh
   ssh -J root@<BASTION_VSI_IP> root@<RHEL_VSI_IP>
   ```
   {: pre}

   Where

   `<BASTION_VSI_IP>`
   :   The IP address of the bastion virtual server that you copied previously.

   `<RHEL_VSI_IP>`
   :   The IP address of the RHEL virtual server that you copied previously.
