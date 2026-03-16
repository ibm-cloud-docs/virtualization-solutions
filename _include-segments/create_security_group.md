## Creating a security group
{: #virt-sol-vpc-migration-tutorial-create-security-group}
{: step}

Use the following steps to create a security group. Security groups allow SSH connections to the Bastion virtual server.

To improve the security of your VPC, the inbound rules in this security group must set an IP or CIDR as the _Source type_ to restrict the sources of traffic into its attached resources.
{: note}

1. From the **Navigation menu**, click **Infrastructure > Network security groups**.
2. Click **Create**.
3. In the **Location** section, specify the following information:
   - For **Geography**, select **North America**.
   - For **Region**, select **Dallas (us-south)**.
      - For **Zone**, select **us-south-1**.
   - In the **Details** section, specify the following information:
      - For **Name**, enter `vpc-migration-sg-bastion`
      - For **Virtual private cloud**, select **vpc-migration**.
4. In the **Inbound rules** section, specify the following information:
      1. Click **Create**. In the **Create inbound rule** form, specify the following information
         - For **Protocol**, select **TCP**.
         - For **Port**, select the **Port range**.
         - For **Port min**, enter `22`.
         - For **Port max**, enter `22`.
         - Click **Create**.
      2. Click **Create**. In the **Create inbound rule** form, specify the following information
         - For **Protocol**, select **ICMP**.
         - For **Value**, select **Type and code**.
         - For **Type**, enter `8`.
         - For **Code**, keep empty.
         - Click **Create**.
5. In the **Attaching virtual server interfaces** section, specify the following information
   - Select the interface of the Bastion virtual server.
   - Click **Create a security group**.
6. Get the IP of the Bastion virtual server
   - From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
   - Search for **vpc-migration-vsi-bastion**.
   - Copy the floating IP of the Bastion virtual server from the list of results.
7. Get the IP of the worker virtual server
   - From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
   - Search for **vpc-migration-vsi-worker**.
   - Copy the reserved IP of the worker virtual server from the list of results.
8. Copy the downloaded SSH private key to its default location.

    - On Windows, it's `C:\Users\<YourUsername>\.ssh\id_rsa`.
    - On Linux or macOS, it's `~/.ssh/id_rsa`.

9. Try to connect to the internet from the worker virtual server
   - Use Secure Shell to log in to the worker virtual server by running the following command:

   `ssh -J root@<BASTION_VSI_IP> root@<WORKER_VSI_IP>`

   Where

   - `<BASTION_VSI_IP>` is the IP of the Bastion virtual server that you copied previously.
   - `<WORKER_VSI_IP>` is the IP of the worker virtual server that you copied previously.

   - Verify that you can reach Google by its domain name by running the following command:

   `nslookup www.google.com`
