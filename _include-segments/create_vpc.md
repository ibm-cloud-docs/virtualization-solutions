## Creating a VPC
{: #virt-sol-vpc-migration-tutorial-create-vpc}
{: step}

Use the following steps to create and set up a VPC environment. The VPC hosts the migrated virtual servers and associated resources such as subnets, gateways, and security groups. The following information also includes resources that are used later in the migration process.

Before you begin, make sure that you're logged in to the IBM Cloud console.
{: tip}

   1. From the **Navigation menu**, click **Infrastructure** > **Network** > **VPCs**, and click **Create**.
   2. In the **Location** section, specify the following information:
      - For **Geography**, select **North America**.
      - For **Region**, select **Dallas (us-south)**.
   3. In the **Details** section, for **Name**, enter `vpc-migration`.
   4. In the **Subnets** section, specify the following information:
      1. Delete all but the subnet in the zone **us-south-1**.
      2. Click the **edit** icon of the remaining subnet. In the Edit Subnet form that appears specify the following information:
         - In the **Details** section, for **Name**, enter `vpc-migration-sn-1` and click **Save**.
      3. Click **Create a virtual private cloud**.

## Creating a public gateway
{: #virt-sol-vpc-migration-tutorial-create-public-gateway}
{: step}

You need to create a public gateway to access to the internet from the VPC.

Use the following steps to create a public gateway.

1. From the **Navigation menu**, click **Infrastructure** > **Network** > **Subnets**.
2. From the list of available resources, select **vpc-migration-sn-1**.
3. In the **Public gateway** section, click the toggle button.
4. Within the **Attach public gateway** form, click **Attach**.
5. Bind a floating IP to the Bastion virtual server by using the following steps.
   1. From the **Navigation menu**, click **Infrastructure** > **Compute** > **Virtual server instances**.
   2. From the list of available resources, select **vpc-migration-vsi-bastion**.
   3. Click **Networking**.
   4. Click the **Menu** icon of the attached network interface.
      1. Click **Edit floating IPs**. Within the **Edit floating IPs** form, click **Attach**
      2. Within the **Attach a floating IP** form, click **Reserve a new floating IP**.
      3. Within the **Reserve a floating IP** form, in the **Details** section, for **Name**, enter `vpc-migration-fip` and click **Reserve**.
6. Click **Close** to close the menu.
