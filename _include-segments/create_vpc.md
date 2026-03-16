## Creating a VPC
{: #virt-sol-vpc-migration-tutorial-create-vpc}
{: step}

Use the following steps to create and set up a VPC environment. The VPC hosts the migrated virtual servers and associated resources such as subnets, gateways, and security groups. The following information also includes resources that are used later in the migration process.

Before you begin, make sure that you're logged in to the IBM Cloud console.
{: tip}

   1. From the **Navigation menu**, click **Infrastructure > Network > VPCs**.
   2. Click **Create** and specify the following information.

   3. In the **Location** section, specify the following information
      1. For **Geography**, select **North America**.
      2. For **Region**, select **Dallas (us-south)**.
   4. In the **Details** section, specify the following information
      1. For **Name**, enter `vpc-migration`.
   5. In the **Subnets** section, specify the following information
      1. Delete all but the subnet in the zone **us-south-1**.
      2. Click the **edit** icon of the subnet that remains. The Edit Subnet form appears. Within the form:
         1. In the **Details** section, for **Name**, enter `vpc-migration-sn-1`.
         2. Click **Save**.
      3. Click **Create a virtual private cloud**.

## Creating a public gateway
{: #virt-sol-vpc-migration-tutorial-create-public-gateway}
{: step}

Use the following steps to create a public gateway.

1. Create the Public Gateway to allow access to the internet from the VPC
   1. From the **Navigation menu**, click **Infrastructure > Network > Subnets**.
   2. From the list of available resources, select **vpc-migration-sn-1**.
   3. In the **Public Gateway** section, click the **toggle switch**. Within the **Attach public gateway** form, click **Attach**.
2. Bind a Floating IP to the Bastion virtual server by using the following steps.
   1. From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
   2. From the list of available resources, select **vpc-migration-vsi-bastion**.
   3. Click **Networking**.
   4. Click the **Menu** icon of the attached Network Interface.
   5. Click **Edit floating IPs**. Within the **Edit floating IPs** form, specify the following information:
      1. Click **Attach**. Within the **Attach a Floating IP** form, specify the following information:
      2. Click **Reserve a new floating IP**. Within the **Reserve a Floating IP** form, specify the following information.
      3. In the **Details** section, for **Name**, enter `vpc-migration-fip`.
      4. Click **Reserve**.
3. Click **Close**.
