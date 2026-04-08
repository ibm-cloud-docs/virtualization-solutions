## Starting the RHEL virtual server
{: #virt-sol-vpc-migration-tutorial-start-rhel-vm}

Use the following steps to create a RHEL virtual server from the attached boot volume.

1. Log in to the IBM Cloud console.
1. From the **Navigation menu**, click **Infrastructure** > **Storage** > **Block storage volumes**.
2. From the list of available resources, find and click **vpc-migration-vsi-rhel9-boot-volume**.
3. In the **Attached virtual server** section, click the **Detach** icon next to the name of the worker virtual server. wait for the worker virtual server to detach.
5. In the **Attached virtual server** section, click **Attach**. Within the **Attach to the virtual server** form, specify the following information:
   1. Click **Create server**
   2. Click **Attach as boot volume**.
   3. In the **Location** section, specify the following information:
      - For **Geography**, select **North America**.
      - For **Region**, select **Dallas (us-south)**.
      - For **Zone**, select **us-south-1**.
   4. In the **Details** section, for **Name**, enter `vpc-migration-vsi-rhel9`.
   5. In the **Server configuration** section, for **SSH Keys**, select **vpc-migration-ssh-key**.
   6. Click **Create a virtual server**.
6. Get the IP of the RHEL virtual server by specifying the following information:
   1. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   2. Search for **vpc-migration-vsi-rhel9**.
   3. From the list of results, copy the reserved IP of the RHEL virtual server.
7. Log in to the new RHEL virtual server
    - SSH into the RHEL virtual server by running the following command:

    `ssh -J root@<BASTION_VSI_IP> root@<RHEL_VSI_IP>`

    Where

    - `<BASTION_VSI_IP>` is the IP of the Bastion virtual server that you copied previously.
    - `<RHEL_VSI_IP>` is the IP of the RHEL virtual server that you copied previously.
