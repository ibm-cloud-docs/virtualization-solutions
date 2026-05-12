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
   4. Deselect the **Create a default prefix for each zone** checkbox. The default prefix remains the same across VPCs and can introduce routing conflicts when multiple VPCs are connected to the same {{site.data.keyword.cloud_notm}} classic infrastructure account.
   5. Click **Create a virtual private cloud**.
   6. Click the name of the VPC that you created.
   7. Select the **Address prefixes** tab.
   8. Create an address prefix of your choice for the Location **us-south-1**. Ensure that this address prefix does not overlap with prefixes that are used by other VPCs that are connected through {{site.data.keyword.cloud_notm}} classic infrastructure using {{site.data.keyword.cloud_notm}} Transit Gateway.
   9. From the **Navigation menu**, go to **Infrastructure** > **Network** > **Subnets**. Then, click **Create**.
   10. In the **Location** section, specify the following information:
       - For **Geography**, select **North America**.
       - For **Region**, select **Dallas (us-south)**.
       - For **Zone**, select **us-south-1**.
   11. In the **Details** section, for **Name**, enter `vpc-migration-sn-1`, and select the new VPC for the **Virtual private cloud**.
   12. In the **IP range selection** section, select the address prefix that you created earlier. You can use the entire prefix for your subnet.
   13. Select the toggle to **Attach** a public gateway to your subnet. This enables your workloads to access the internet from the VPC.
   14. Click **Create subnet**.
