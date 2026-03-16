## Starting the RHEL virtual server
{: #virt-sol-vpc-migration-tutorial-start-rhel-vm}

1. Log in to the IBM Cloud console.
2. Create an RHEL virtual server from the attached boot volume
    1. From the **Navigation menu**, click **Infrastructure > Storage > Block storage volumes**.
    2. Find and click **vpc-migration-vsi-rhel9-boot-volume** from the list of available resources.
    3. In the **Attached virtual server** section, click the **detach** icon next to the name of the worker virtual server.
    4. Wait for the worker virtual server to detach.
    5. In the **Attached virtual server** section, click **Attach**. Within the **Attach to the virtual server** form, specify the following information:
       1. Click **Create server**
       2. Click **Attach as boot volume**.
       3. In the **Location** section, specify the following information:
          1. For **Geography**, select **North America**.
          2. For **Region**, select **Dallas (us-south)**.
          3. For **Zone**, select **us-south-1**.
       4. In the **Details** section, specify the following information:
          1. For **Name**, enter `vpc-migration-vsi-rhel9`
       5. In the **Server configuration** section, specify the following information:
          1. For **SSH Keys**, select **vpc-migration-ssh-key**.
       6. Click **Create a virtual server**.
3. Get the IP of the RHEL virtual server by specifying the following information:
    1. From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
    2. Search for **vpc-migration-vsi-rhel9**.
    3. From the list of results, copy the reserved IP of the RHEL virtual server.
4. Log in to the new RHEL virtual server
    1. SSH into the RHEL virtual server by running the following command:

    `ssh -J root@<BASTION_VSI_IP> root@<RHEL_VSI_IP>`

    Where

    - `<BASTION_VSI_IP>` is the IP of the Bastion virtual server that you copied previously.
    - `<RHEL_VSI_IP>` is the IP of the RHEL virtual server that you copied previously.
    
