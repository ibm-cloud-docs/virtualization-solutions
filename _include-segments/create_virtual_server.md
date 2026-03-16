## Creating a virtual server
{: #virt-sol-vpc-migration-tutorial-create-virtual-server}
{: step}

Use the following steps to create a worker virtual server from which to perform the migrations.

1. From the **Navigation menu**, click **Infrastructure > Network > Subnets**.
2. From the list of available resources, select **vpc-migration-sn-1**.
3. Click **Attached Resources**.
4. In the **Attached Instances** section, click **Create**.
5. In the **Location** section, specify the following information
   1. For **Geography**, select **North America**.
   2. For **Region**, select **Dallas (us-south)**.
   3. For **Zone**, select **us-south-1**.
6. In the **Details** section, for **Name**, enter `vpc-migration-vsi-worker`.
7. In the **Server configuration** section, specify the following information
      1. For **Image**, click **Change Image**. The Select an Image form appears. Within the form:
         1. Search for **Ubuntu**.
         2. Select **ibm-ubuntu-24-04-3-minimal-amd64-4** from the list of results.
         3. Click **Save**.
      2. For **SSH keys**, click **Create an SSH key**. The Create an SSH key form appears. Within the form:
         1. In the **Details** section, specify the following information
            1. For **Name**, enter `vpc-migration-ssh-key`.
            2. For **Select SSH key input method**, select **Generate a key pair for me**.
         2. Click **Create**. The SSH private key is downloaded automatically.
8. Click **Create a virtual server**.
