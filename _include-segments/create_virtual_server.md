## Creating a virtual server
{: #virt-sol-vpc-migration-tutorial-create-virtual-server}
{: step}

Use these steps to create a worker virtual server that is used to run migrations.

1. From the **Navigation menu**, click **Infrastructure** > **Network** > **Subnets**.
2. From the list of available resources, select **vpc-migration-sn-1**.
3. Click **Attached resources**.
4. In the **Attached instances** section, click **Create**.
5. In the **Location** section, specify the following information:
   - For **Geography**, select **North America**.
   - For **Region**, select **Dallas (us-south)**.
   - For **Zone**, select **us-south-1**.
6. In the **Details** section, for **Name**, enter `vpc-migration-vsi-worker`.
7. In the **Server configuration** section, specify the following information:
   - For **Image**, click **Change image**.
   - The **Select an image** form appears where you take the following actions:
       1. Search for **Ubuntu**.
       2. Select **ibm-ubuntu-24-04-3-minimal-amd64-4** from the list of results and click **Save**.
   - For **SSH keys**, click **Create an SSH key** and in the Create an SSH key form, take the following actions:
       1. In the **Details** section, specify the following information:
          1. For **Name**, enter `vpc-migration-ssh-key`.
          2. For **Select SSH key input method**, select **Generate a key pair for me**.
       2. Click **Create**. The SSH private key is downloaded automatically.
8. Click **Create a virtual server**.
