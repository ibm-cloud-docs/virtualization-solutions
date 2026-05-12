## Creating a Bastion virtual server
{: #virt-sol-vpc-migration-tutorial-create-bastion-virtual-server}
{: step}

Create a Bastion virtual server to securely access the resources in the VPC.

1. From the **Navigation menu**, click **Infrastructure** > **Network** > **Subnets**.
2. From the list of available resources, select **vpc-migration-sn-1**.
3. Click **Attached resources**.
4. In the attached instances section, click **Create**.
5. In the **Location** section, specify the following information:
   - For **Geography**, select **North America**.
   - For **Region**, select **Dallas (us-south)**.
   - For **Zone**, select **us-south-1**.
6. In the **Details** section, for **Name**, enter `vpc-migration-vsi-bastion`.
7. In the server configuration section, specify the following information:
   - For **Image**, click **Change image**.
   - The **Select an image** form appears where you take the following actions:
      1. Search for `ubuntu`.
      2. Select **ibm-ubuntu-24-04-3-minimal-amd64-4** from the list of results and click **Save**.
8. For **SSH keys**, select **vpc-migration-ssh-key**.
9. Click **Create a virtual server**.

## Creating a floating IP
{: #virt-sol-vpc-migration-tutorial-create-floating-ip}
{: step}

To access your bastion server from the internet, create a floating IP by completing the following steps.

1. From the **Navigation menu**, go to **Infrastructure** > **Compute** > **Virtual server instances**.
2. From the list of available resources, select **vpc-migration-vsi-bastion**.
3. Click **Networking**.
4. Click the **Menu** icon for the attached network interface.
   1. Click **Edit floating IPs**.
   2. In the **Edit floating IPs** form, click **Attach**
   3. In the **Attach a floating IP** form, click **Reserve a new floating IP**.
   4. In the **Reserve a floating IP** form, in the **Details** section:
       1. For **Name**, enter `vpc-migration-fip`.
       2. Click **Reserve**.
5. Click **Close** to exit the menu.
