## Creating a VPC
{: #virt-sol-vpc-migration-tutorial-create-vpc}
{: step}

Use the following steps to create and set up a VPC environment. The VPC hosts the migrated virtual servers and associated resources such as subnets, gateways, and security groups. The following information also includes resources that are used later in the migration process.

Before you begin, make sure that you're logged in to the IBM Cloud console.
{: tip}

   1. From the **Navigation menu**, go to **Infrastructure** > **Network** > **VPCs**, and then click **Create**.
   2. In the **Location** section, specify the following information:
      - For **Geography**, select **North America**.
      - For **Region**, select **Dallas (us-south)**.
   3. In the **Details** section, for **Name**, enter `vpc-migration`.
