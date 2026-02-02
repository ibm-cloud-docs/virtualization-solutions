---

copyright:
  years: 2026, 2026
lastupdated: "2026-02-02"

keywords: Red Hat OpenShift Virtualization, virtual servers, ROKS, VSI, File Storage, Backup, Kasten, Veeam, volumes

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 60m

---
{{site.data.keyword.attribute-definition-list}}

# Migrating your VMware workloads to IBM Cloud Virtual Servers for VPC

{: #virt-sol-vpc-migration-tutorial-overview}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}

The following tutorial describes how to migrate your workloads from a {{site.data.keyword.vmware-service_full}} (VCFaaS) environment to {{site.data.keyword.vpc_full}} (VPC). You can also use this tutorial with a self‑managed VMware environment.

## Objective

{: #virt-sol-vpc-migration-tutorial-overview-objectives}

The following objectives are covered in the following tutorial.

- Create a VPC and the resources that it needs
- Set up a VCFaaS instance with a Microsoft Windows&reg; virtual server and an RHEL virtual server to use as migration examples
- Build a private connection between VCFaaS and VPC through a Transit Gateway (TGW)
- Move your virtual server disks into VPC Block Storage volumes
- Create virtual servers from the migrated disks

Some VMware configuration steps, such as connecting your environment to VPC through a Transit Gateway, are different.
{: note}

The virtual servers that are part of this tutorial have the following constraints:

- They have a single disk size of less than 250 GB
- They are not Bring Your Own License (BYOL)
- They access the internet to download the necessary packages to perform the migration

The following diagram illustrates what you build as part of this tutorial.

![Virtualization Solutions VPC Migration Tutorial High-Level Architecture](../../images/vpc-vsi/vpc-vsi-migration-tutorial-high-level-diagram.svg){: caption="Virtualization Solutions VPC Migration Tutorial High Level Architecture" caption-side="bottom"}

## Before you begin

{: #virt-sol-vpc-migration-tutorial-prerequisites}

This tutorial requires the following prerequisites.

- An available VCFaaS instance
- Have the correct access policies to manage VCFaaS environments and make sure that you can create the following VCFaaS resources. For more information, see [Managing IAM access for VCFaaS](/docs/vmware-service?topic=vmware-service-vmaas-iam&interface=ui).
  - VMs
  - Networks
  - Firewall Rules
  - NAT rules
  - TGW Connection Group
- Have the correct access policies to manage VPCs and their resources and make sure that you can create the following VPC resources. For more information, see [Managing IAM access for VPC Infrastructure Services](/docs/vpc?topic=vpc-iam-getting-started&interface=ui).
  - VPCs
  - Virtual servers
  - Security groups
  - Public gateways
  - Floating IPs
- Have the correct access policies to manage TGWs and make sure that you can create the following TGW resources. For more information, see [Using IAM permissions with IBM Cloud Transit Gateway](/docs/transit-gateway?topic=transit-gateway-iam).
  - TGWs (Transit gateways)
  - TGW connections

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

## Creating a Bastion virtual server

{: #virt-sol-vpc-migration-tutorial-create-bastion-virtual-server}
{: step}

Create a Bastion virtual server from which to securely access the resources in the VPC.

1. From the **Navigation menu**, click **Infrastructure > Network > Subnets**.
2. From the list of available resources, select **vpc-migration-sn-1**.
3. Click **Attached Resources**.
4. In the **Attached Instances** section, click **Create**.
5. In the **Location** section, specify the following information
   - For **Geography**, select **North America**.
   - For **Region**, select **Dallas (us-south)**.
   - For **Zone**, select **us-south-1**.
6. In the **Details** section, for **Name**, enter `vpc-migration-vsi-bastion`.
7. In the **Server configuration** section, specify the following information
   - For **Image**, click **Change Image**. The **Select an image** form appears. Within the form, do the following actions:
      1. Search for `ubuntu`.
      2. Select **ibm-ubuntu-24-04-3-minimal-amd64-4** from the list of results.
      3. Click **Save**.
8. For **SSH keys**, select **vpc-migration-ssh-key**.
9. Click **Create a virtual server**.

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
   - From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.**
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

## Setting up networking on a VCFaaS instance

{: #virt-sol-vpc-migration-tutorial-setup-vcfaas}
{: step}

Use the following steps to configure networking in your VMware Cloud Director (VCFaaS) environment to enable outbound internet access to install the required packages on the virtual servers.

1. Log in to the IBM Cloud console.
2. Go to the VCFaaS tenant portal.
   1. From the **Navigation menu**, click **VMware > Resources > VCF as a Service**.
   2. From the list of available resources, select your Director site and click **VMware console**.
   3. Click **Sign in with OIDC** and follow the prompts to log in with your IBM Cloud account.
3. Create a routed Virtual Data Center (VDC) Network.
   1. In the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Networking** section, click **Networks**.
   4. Click **New**. In the **New organization VDC network** wizard, specify the following information:
      1. In the **Scope** section, specify the following information:
         1. Select **Current organization virtual data center**.
         2. Click **Next**.
      2. In the **Network type** section, specify the following information:
         1. Select **Routed**.
         2. Click **Next**.
      3. In the **Edge connection** section, specify the following information:
         1. Select your Edge Gateway.
         2. Click **Next**.
      4. In the **General** section, specify the following information:
         1. For **Name**, enter `192.168.0.1/24`.
         2. For **Gateway CIDR**, enter `192.168.0.1/24`.
         3. Click **Next**.
      5. In the **Static IP pools** section, specify the following information:
         1. For **IP range**, enter `192.168.0.2-192.168.0.254`.
         2. Click **Add**
         3. Click **Next**
      6. In the **DNS** section, specify the following information:
         1. For **Primary DNS**, enter `161.26.0.10`.
         2. For **Secondary DNS**, enter `8.8.8.8`.
         3. Click **Next**.
      7. In the **Segment profile template** section, click **Next**.
      8. In the **Ready To complete** section, click **Finish**.
4. Create a firewall rule to allow outbound traffic from the routed VDC Network.
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Networking** section, click **Edges**.
   4. Click your Edge gateway.
   5. In the **Services** section, click **Firewall**.
   6. Click **New**. Within the **New rule** form, specify the following information:
      1. For **Name**, enter `outbound-rule`
      2. For **Source**, click **Edit**. In the **Select source firewall groups** form, specify the following information:
          1. Click **Firewall IP addresses**.
          2. Click **Add**.
          3. For the new list item that appears, enter `192.168.0.1/24`.
          4. Click **Keep**.
      3. For **Destination**, click **Edit**. In the **Select destination firewall groups** form, specify the following information:
         1. Set **Any destination** to **enabled**.
         2. Click **Keep**.
      4. Click **Save**.
5. Create a NAT Rule to allow access to the internet from the routed VDC Network
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Networking** section, click **Edges**.
   4. Click your Edge Gateway. In the **Services** section, click **NAT**.
   5. Click **New**. Within the **Add NAT rule** form, specify the following information:
      1. For **Name**, enter `outbound-nat`.
      2. For **NAT action**, enter `SNAT`.
      3. For **External IP**, select an IP from the list of available IPs, or request a new one.
      4. For **Internal IP**, enter `192.168.0.0/24`.
      5. Click **Save**.

## Uploading the Ubuntu Server ISO into the VCFaaS instance

{: #virt-sol-vpc-migration-tutorial-upload-ubuntu-iso}
{: step}

Use the following steps to create a catalog in VCFaaS and upload an Ubuntu Server ISO into it. This ISO is used later to boot the virtual servers into a shell for disk migration.

1. Go to [Get Ubuntu Server](https://ubuntu.com/download/server) and download the ISO of the most recent LTS version of Ubuntu Server.
2. Log in to the IBM Cloud console.
3. Go to the VCFaaS tenant portal.
4. Create a Catalog
   1. From the side window, click **Content hub**.
   2. Click **Catalogs**.
   3. Click **New**. In the **Create catalog** form, specify the following information:
      1. For **Name**, enter `vpc-migration-catalog`.
      2. Click **OK**.
5. Upload the Ubuntu Server ISO to the catalog.
   1. From the side window, click **Content hub**.
   2. Click **Catalogs**.
   3. From the list of available catalogs, select **vpc-migration-catalog**.
   4. Click **Media**.
   5. Click **Add**. In the **Upload media** form, specify the following information:
      1. Click the **Upload** icon.
         1. In the file browser, click the Ubuntu Server ISO that you previously downloaded.
         2. Click **Open**.
      2. Click **OK** and wait for the ISO to upload.

## Creating the Windows and RHEL virtual servers

{: #virt-sol-vpc-migration-tutorial-create-vms}
{: step}

Use the following steps to create Windows and RHEL virtual servers in VCFaaS. These servers are migrated to the VPC.

1. Log in to the IBM Cloud console.
2. Go to the VCFaaS tenant portal.
3. Create a Windows virtual server.
   1. From the side window, click **Data centers**
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Click **New VM** and specify the following information:
      1. For **Name**, enter `vm-win22`.
      2. For **Templates**, select **Windows-22-Template-Official**.
      3. From the **NICs**:
         1. For **IP Mode**, select **Static - IP pool**.
      4. Click **OK** and wait for the virtual server to power on.
4. Get the password of the Windows virtual server
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-win22** from the list of available virtual servers.
   5. Click **Details** > **Guest OS Customization** > **Edit**. In the **Edit Guest Properties** form, specify the following information:
      1. Copy your password.
      2. Click **Discard**.
5. Open a web console window to the Windows virtual server.
   1. From the side window, click **Data centers**.
   2. Click your VDC from the list of available data centers.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-win22** from the list click **Details** > **Launch Web Console** and in the **Web Console** window, log in as Administrator with the password that you copied.
6. Try to reach the internet from the Windows virtual server.
   1. From the Web Console window, do the following actions:
       1. Open Microsoft Edge.
       2. Go to the Google home page.
       3. Verify that the page loads.
   2. Close the Web Console window.
7. Create a RHEL virtual server.
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Click **New VM**. In the **New VM** form, specify the following information:
      1. For **Name**, enter `vm-rhel9*`
      2. For **Templates**, select **RHEL-9-Template-Official**
      3. In the **NICs** section, specify the following information:
         1. For **IP Mode**, select **Static - IP Pool**.
      4. Click **OK** and wait for the virtual server to power on.
8. Get the password of the RHEL.
   1. From the side window, click **Data centers**.
   2. Click your VDC from the list of available data centers.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9** from the list of available virtual servers.
   5. Click **Details** > **Guest OS Customization** > **Edit** and in the **Edit Guest Properties** form
      1. Copy your password.
      2. Click **Discard**.
9. Open a Web Console window to the RHEL virtual server.
   1. From the side window, click **Data centers**
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9** from the list of available virtual servers.
   5. Click **Details**.
   6. Click **Launch web console** and in the **Web console window**
       1. Log in as root with the password that you copied.
10. Try to reach the internet from the RHEL virtual server.
    1. From the Web Console window, verify that you can resolve Google's domain name by running the following command:

         `nslookup www.google.com`

    2. Close the Web Console window.

## Getting the ISO for the Windows virtIO drivers into the Windows virtual server

{: #virt-sol-vpc-migration-tutorial-load-virtio-iso}
{: step}

Use the following steps to obtain and transfer the ISO that contains the Windows virtIO drivers. These drivers help ensure compatibility with IBM Cloud VPC.

1. Log in to the IBM Cloud console.
2. Go to the VCFaaS tenant portal.
3. Create a temporary RHEL virtual server.
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Click **New VM** and specify the following information:
      1. For **Name**, enter `vm-rhel9-tmp`.
      2. For **Templates**, select **RHEL-9-Template-Official**.
      3. In the **NICs** section, specify the following information
         1. For **IP Mode**, select **Static - IP Pool**.
      4. Click **OK** and wait for the virtual server to power on.
4. Get the password of the temporary RHEL virtual server.
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9-tmp** from the list of available virtual servers.
   5. Click **Details**.
   6. Click **Guest OS customization**.
   7. Click **Edit** and in the **Edit guest properties** form,
      1. Copy your password.
      2. Click **Discard**.
5. Open a Web Console window to the temporary RHEL virtual server
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. From the list of avaible virtual servers, find **vm-rhel9-tmp**.
   5. Click **Details** > **Launch Web Console** and in the **Web Console window**,
6. Register the temporary RHEL virtual server with the Red Hat Subscription Manager by following the steps that are in the [Operating VMware Cloud Director guide](/docs/vmware-service?topic=vmware-service-vcd-ops-guide#vcd-ops-guide-public-cat-rhel).
7. Install the `virtio-win` package on the RHEL virtual server.
   1. From the Web Console window,
      1. Run `dnf install -y virtio-win` to install the package.
8. Get the IP of the temporary RHEL virtual server
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9-tmp** from the list of available virtual servers.
   5. Click **Details**.
   6. In the **Hardware** section, click **NICs**.
   7. Copy the IP address of the primary network interface.
9. Get the password of the Windows virtual server.
10. Copy the virtIO drivers ISO into the Windows virtual server.
    1. From the side window, click **Data centers**.
    2. From the list of available data centers, select your VDC.
    3. In the **Compute** section, click **Virtual machines**.
    4. Find **vm-win22** from the list of available virtual servers.
    5. Click **Details** > **Launch Web Console** and in the **Web Console** window,
        1. Log in as Administrator with the password of the Windows virtual server.
        2. Open the command prompt and copy the ISO from the RHEL virtual server by running the following command:
        `scp root@<TEMP_RHEL_VM_IP>:/usr/share/virtio-win/virtio-win.iso C:\virtio-win.iso`

            Where

            `<TEMP_RHEL_VM_IP>` is the IP of the temporary RHEL virtual server that you previously copied.

            When prompted whether you trust the remote server, enter **y** and enter the password of the temporary RHEL virtual server.

## Getting the cloud-init installer for the Windows virtual server

{: #virt-sol-vpc-migration-tutorial-load-cloud-init-installer}
{: step}

The Windows virtual server also needs Cloud-init installed on it to run as a virtual server in your VPC. Use the following steps to obtain the installer from the internet.

1. Log in to the IBM Cloud console.
2. Go to the VCFaaS tenant portal.
3. Get the password of the Windows virtual server.
4. Open a Web Console window to the Windows virtual server.
5. Download the cloud-init installer
   1. From the Web Console window:
      1. Open Microsoft Edge.
      2. Go to [the download page for the installer](https://www.cloudbase.it/downloads/CloudbaseInitSetup_1_1_4_x64.msi). The download starts automatically.
   2. Close the Web Console window.

## Setting up a transit gateway (private connection) between the VCFaaS instance and VPC

{: #virt-sol-vpc-migration-tutorial-setup-tgw}
{: step}

Use the following steps to create a transit gateway to securely connect your VCFaaS instance and VPC over the {{site.data.keyword.cloud_notm}} private network.

1. Log in to the IBM Cloud console.
2. Create a transit gateway.
   1. From the **Navigation menu**, click **Infrastructure > Network > Transit gateway**.
   2. Click **Create**
   3. In the **Configuration** section, specify the following information
      1. For **Name**, enter `vpc-migration-tgw`.
   4. In the **Location** section, specify the following information
      1. For **Routing option**, select **Local routing**
      2. For **Location**, select **Dallas (us-south)**.
      3. Click **Create**
3. Connect the VPC to the transit gateway.
   1. From the **Navigation menu**, click **Infrastructure > Network > Transit gateway**.
   2. From the list of available resources, select **vpc-migration-tgw**.
   3. Click **Add connection** and within the form, specify the following information:
      1. For **Network connection**, select **VPC**.
      2. For **Connection reach**, select **Add a new connection to this account**.
      3. For **Region**, select **Dallas (us-south)**.
      4. For **Available connections**, select **vpc-migration**.
      5. Click **Add**.
4. Connect the VCFaaS instance to the transit gateway.
   1. Follow the steps that are in [Using transit gateway to interconnect VCF as a Service with IBM Cloud services](https://cloud.ibm.com/docs/vmware-service?topic=vmware-service-tgw-adding-connections) to connect the VCFaaS instance to the transit gateway.
5. Create a security group to allow traffic between the virtual server on the VPC environment and the VCFaaS instance.
   1. From the **Navigation menu**, click **Infrastructure > Network > Security groups**.
   2. Click **Create**.
   3. In the **Location** section, specify the following information
      1. For **Geography**, select **North America**.
      2. For **Region**, select **Dallas (us-south)**.
   4. In the **Details** section, specify the following information
      1. For **Name**, enter `vpc-migration-sg-tgw`.
      2. For **Virtual private cloud**, select **vpc-migration**.
   5. In the **Inbound rules** section, specify the following information
      1. Click **Create** and within the form, specify the following information:
         1. For **Protocol**, select **Any protocol**.
         2. For **Source type**, select **IP or CIDR**.
         3. For **Source**, enter `192.168.0.0/24`.
         4. Click **Create**.
   6. In the **Attaching virtual server interfaces** section, select the interface of the worker virtual server and click **Create a security group**.
6. Get the IP range of the VPC subnet
   1. From the **Navigation menu**, click > **Infrastructure > Network > Subnets**.
   2. From the list of available resources, select **vpc-migration-sn-1**.
   3. Copy the value of the **IP range**.
7. Go to the VCFaaS tenant portal.
8. Create firewall rules to allow traffic between the VCFaaS instance and the virtual serverI on a VPC environment.
   1. From the side window, click **Data centers**.
   2. From the list of available data centers, select your VDC.
   3. In the **Networking** section, click **Edges**
   4. Click your edge gateway.
   5. In the **Services** section, click **Firewall**.
   6. Click **New** and within the form, specify the following information:
      1. For **Name**, enter `tgw-outbound-rule`.
      2. For **Source**, click **Edit** and within the form, specify the following information:
          1. Click **Firewall IP addresses**.
          2. Click **Add** and enter `192.168.0.0/24`.
          3. Click **Keep**.
      3. For **Destination**, click **Edit** and within the form, specify the following information:
          1. Click **Firewall IP addresses**.
          2. Click **Add**.
          3. For the new item that appears, enter the IP range of the VPC subnet.
          4. Click **Keep**.
      4. Click **Save**.
   7. Click **New** and within the form, specify the following information:
       1. For **Name**, enter `tgw-inbound-rule`.
       2. For **Source**, click **Edit** and within the form, specify the following information:
          1. Click **Firewall IP addresses**.
          2. Click **Add**.
          3. For the new item that appears, enter the IP range of the VPC subnet.
          4. Click **Keep**.
       3. For **Destination**, click **Edit** and within the form, specify the following information:
          1. Click **Firewall IP addresses**.
          2. Click **Add** and enter `192.168.0.0/24`.
          3. Click **Keep**.
       4. Click **Save**.
9. Disable the NAT rule to allow access to the internet from the network.
   1. From the side window, click **Data centers**
   2. Click your VDC from the list of available data centers.
   3. In the **Networking** section, click **Edges**.
   4. Click your edge gateway.
   5. In the **Services** section, select **NAT**.
   6. Select **outbound-nat**.
   7. Click **Edit** and within the form, specify the following information:
      1. Expand **Advanced settings**.
      2. Set **State** to the **disabled**.
      3. Click **Save**.
10. Open a Web Console window to the RHEL virtual server.
11. Try to reach an IP in the VPC subnet from the VCFaaS instance.
    1. From the Web Console window, verify that you can ping the worker virtual server by running the following command:

       `ping <WORKER_VSI_IP>`

        Where

        `<WORKER_VSI_IP>` is the IP of the worker virtual server that you copied previously.

    2. Close the Web Console window.
12. Try to reach an IP in the routed VDC network from the VPC.
    1. SSH into the worker virtual server by running the command that you used previously.
    2. Verify that you can ping the gateway of the routed VDC Network by running the following command:

          `ping 192.168.0.1`

## Preparing the Windows virtual server for migration

{: #virt-sol-vpc-migration-tutorial-prep-windows-vm}
{: step}

Before you can migrate the virtual server, you must install the necessary drivers and configure Cloud-init on the Windows virtual server. Use the following steps to prepare the Windows virtual server for migration.

1. Log in to the IBM Cloud console.
2. Go to the VCFaaS tenant portal.
3. Get the password of the Windows virtual server.
4. Open a Web Console window to the Windows virtual server.
5. Install the virtIO drivers on the Windows virtual server.
   1. From the Web Console window:
      1. Open the File Explorer.
      2. Go to `C:\`.
      3. Double-click `C:\virtio-win.iso`.
      4. Double-click `virtio-win-gt-x64` and follow the prompts.
      5. Double-click `virtio-win-guest-tools` and follow the prompts.
6. Install the virtIO drivers on the recovery image of the Windows virtual server by following the steps that are in [Making the virtio-win drivers available in the recovery image](https://cloud.ibm.com/docs/vpc?topic=vpc-create-windows-custom-image#virtio-win-drivers-windows-recovery-image).

    Keep in mind that if your virtual server has a GPT partition table, you must set the partition IDs to UUIDs, not numbers. Get the correct IDs by displaying the details of the System and Recovery partitions while `diskpart` is running. For more information, see the [documentation on the detail partition command](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/detail-partition){: external}.

7. Install and configure Cloud-init on the Windows virtual server
    1. From the Web Console window, open the **File Explorer**.
    2. Click **Downloads**.
    3. Double-click the Cloud-init that you downloaded previously and follow the prompts to install the package. Make sure that you use `Administrator` for the name of the Cloud-init user, not Admin.
    4. Modify the Cloud-init configuration files by following the steps that are in [Customizing a virtual server](/docs/vpc?topic=vpc-create-windows-custom-image#customize-virtual-machine).
8. Run `sysprep` on the Windows virtual server to generalize it immediately before you migrate it.
   1. From the Web Console window, open the command prompt.
   2. Generalize the virtual server by running the following command:
   `C:\Windows\System32\Sysprep\Sysprep.exe /oobe /generalize /shutdown "/unattend:C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\Unattend.xml"`

   After you run sysprep, the virtual server stops.
   2. Close the Web Console.

## Migrating the Windows virtual server

{: #virt-sol-vpc-migration-tutorial-migrate-windows-vm}
{: step}

Use the following steps to migrate the Windows virtual server by transferring its boot disk to a block storage volume in VPC and create a virtual server from it.

When the virtual server starts, Cloud-init runs and the administrator password resets.
{: note}

1. Log in to the IBM Cloud console.
2. Create a detached Windows boot volume
   1. From the **Navigation menu**, click **Infrastructure > Network > Subnets**.
   2. From the list of available resources, select **vpc-migration-sn-1**.
   3. Click **Attached resources**
   4. In the **Attached instances** section, click **Create**.
   5. In the **Location** section, specify the following information:
      1. For **Geography**, select **North America**.
      2. For **Region**, select **Dallas (us-south)**.
      3. For **Zone**, select **us-south-1**.
   6. In the **Details** section, specify the following information:
      1. For **Name**, enter `vpc-migration-vsi-win22`.
   7. IN the **Server configuration** section, specify the following information:
      1. For **Image**, click **Change image**. Within the **Select an image** form, specify the following information:
         1. Search for **windows**.
         2. From the list of results, select **ibm-windows-server-2022-full-standard-amd64-32**.
         3. Click **Save**.
      2. For **SSH keys**, select **vpc-migration-ssh-key**.
   8. In the **Storage** section, click the **edit** icon that is next to the item for the boot volume. Within the **Edit boot volume** form, specify the following information:
       1. In the **Details** section, specify the following information:
          1. For **Name**, enter `vpc-migration-vsi-win22-boot-volume`
          2. Set **Auto-delete** to **disabled**
       2. In the **Profiles** section, specify the following information:
          1. Select **general purpose**
       3. In the **Storage capacity** section, specify the following information:
          1. For **Storage size**, enter a value that is greater than or equal to the storage size of the Windows virtual server.
       4. Click **Save**.
   9. Click **Create a virtual server**.
   10. From the **Navigation menu**, click > **Infrastructure > Compute > Virtual server instances**.
   11. From the list of available resources, select **vpc-migration-vsi-win22**.
   12. Click **Delete**. Within the **Delete virtual server instance** form, specify the following information:
       1. Enter **Delete** into the input.
       2. Click **Delete**.
3. Attach the boot volume to the worker virtual server by specifying the following information:
   1. From the **Navigation menu**, click **Infrastructure > Storage > Block storage volumes**.
   2. From the list of available resources, select **vpc-migration-vsi-win22-boot-volume**.
   3. In the **Attached virtual server** section, click **Attach**. Within the **Attach to a virtual server** form, specify the following information:
      1. From the list of virtual servers, select **vpc-migration-vsi-worker**.
      2. Click **Attach**.
4. SSH into the worker virtual server by running the command that you used previously.
5. Get the name of the block device for the attached boot volume by running the following command:

    `lsblk -n -d -o NAME | tail -n 1`

6. Start the Netcat server, wait for an incoming disk transfer, and write it to the attached boot volume by running the following command:

    `nc -l 31337 | gunzip | dd of=/dev/<DEV_NAME> bs=16M status=progress`

    Where

    `<DEV_NAME>` is the name of the block device that you found previously.

7. Go to the VCFaaS tenant portal.
8. Get the IP of the Windows virtual server by specifying the following information:
   1. From the side window, click **Data centers**.
   2. Click your VDC from the list of available data centers.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-win22** from the list of available VMs.
   5. Click **Details**.
   6. In the **Hardware** section, click **NICs**.
   7. Copy the IP address of the primary network interface.
9. Boot into an Ubuntu Server shell from the Windows virtual server by specifying the following information:
   1. From the side window, click **Data centers**
   2. Click your VDC from the list of available Data Centers.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-win22** from the list of available virtual servers
   5. Click **Details**.
   6. Click **General**.
   7. Click **Edit**.
   8. In the **Boot options** section, specify the following information:
      1. Set **Enter boot setup** to **enabled**
      2. Click **Save**
   9. Click **All actions**:
   10. In the **Media** section, click **Insert media**. Within the **Insert CD** form, specify the following information:
       1. Select the Ubuntu Server ISO.
       2. Click **Insert**.
   11. Click **Power on** and wait for the virtual server to start.
   12. Click **Launch the web console**. Within the **Web Console**, specify the following information:
       1. Choose **EFI VMware Virtual IDE CDROM Drive**.
       2. Choose **Try or install Ubuntu Server**.
       3. Wait for the language selection screen to load.
       4. Choose **Help**.
       5. Choose **Enter shell**.
10. Configure networking on the Ubuntu Server that is running on the Windows virtual server by specifying the following information:
    1. Within the Web Console window:
       1. Add an IP to the primary network interface by running the following command:

            `ip addr add <WINDOWS_VM_IP>/24 dev ens192`

            Where

            `<WINDOWS_VM_IP>` is the IP of the Windows virtual server that you copied previously.

       2. Add a default route through the routed VDC Network by running the following command:

            `ip route add default via 192.168.0.1`

11. Transfer the disk of the Windows virtual server to the worker virtual server by specifying the following information:

    1. Within the Web Console window:
       1. Start the transfer of the Windows virtual server disk by running the following command:

          `dd if=/dev/sda bs=16M status=progress | gzip | nc -v <WORKER_VSI_IP> 31337`

          Where

          `<WORKER_VSI_IP>` is the IP of the worker virtual server that you copied previously.

    2. Wait for the transfer to complete. You can monitor the progress of the transfer from either the Ubuntu Server shell or the worker virtual server.

          Both the Netcat client and server commands might not stop after the transfer completes. If they don't stop, you need to manually interrupt them.
      {: tip}

    3. Close the Web Console window.
12. Fix the partition table on the attached boot volume b moving the partition table to the correct position on the disk by running the following command:

    `sgdisk --move-second-header /dev/<DEV_NAME>`

    Where

    `<DEV_NAME>` is the name of the block device that you found previously.

13. Flush the buffers of the attached boot volume by running the following command:

    `blockdev --flushbufs /dev/<DEV_NAME>`

    Where

    `<DEV_NAME>` is the name of the block device that you found previously.

14. Log in to the IBM Cloud console.
15. Create a Windows virtual server from the attached boot volume by specifying the following information:
    1. From the **Navigation menu**, click **Infrastructure > Storage > Block storage volumes**.
    2. From the list of available resources, select **vpc-migration-vsi-win22-boot-volume**.
    3. In the **Attached virtual server** section, click the **Detach** icon next to the name of the worker virtual server.
    4. Wait for the worker virtual server to detach.
    5. In the **Attached virtual server** section, click **Attach**. Within the **Attach to the virtual server** form, specify the following information:
        1. Click **Create server**.
        2. Click **Attach as boot volume**.
        3. In the **Location** section, specify the following information:
            1. For **Geography**, select **North America**.
            2. For **Region**, select **Dallas (us-south)**.
            3. For **Zone**, select **us-south-1**.
            4. In the **Details** section, specify the following information:
               1. For **Name**, enter `vpc-migration-vsi-win22`
               2. In the **Server configuration** section, specify the following information:
        4. For **SSH Keys**, select **vpc-migration-ssh-key**.
        5. Click **Create a virtual server**.
16. Get the IP of the Windows virtual server by specifying the following information:
    1. From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
    2. Search for **vpc-migration-vsi-win22**.
    3. Copy the reserved IP of the Windows virtual server from the list of results.
17. Get the password of the Windows virtual server.
    1. Follow the steps that are in [Connecting to your Windows instance](/docs/vpc?topic=vpc-vsi_is_connecting_windows#vsi_connecting_windows_instance) to get the password of the Administrator user.
18. Log in to the Windows virtual server.
    1. Display the RDP port of the Windows virtual server through the Bastion virtual server by running the following command:

          `ssh -L 3389:<WINDOWS_VSI_IP>:3389 <BASTION_VSI_IP> -l root -N`

          Where

          `<WINDOWS_VSI_IP>` is the IP of the Windows virtual server that you copied previously.
  
          `<BASTION_VSI_IP>` is the IP of the Bastion virtual server that you copied previously.

    2. Use your preferred Remote Desktop client to connect to the Windows virtual server. Use `localhost` as the IP and log in as Administrator with the password of the Windows virtual server.

## Preparing the RHEL virtual server for migration

{: #virt-sol-vpc-migration-tutorial-prep-rhel-vm}
{: step}

You need to configure the RHEL virtual server with virtIO drivers and Cloud-init to prepare it for migration. The installation of the RHEL virtIO drivers depends on whether they are included in the kernel or whether they need to load as modules. These steps assume that the drivers need to be loaded and that they aren't loaded.

Use the following information to prepare the RHEL virtual server for migration.

1. Log in to the IBM Cloud console.
2. Go to the VCFaaS tenant portal.
3. Get the password of the RHEL virtual server.
4. Open a Web Console window to the RHEL virtual server.
5. Configure the virtio drivers to load on boot by specifying the following information:
   1. From the Web Console window:
      1. Create the configuration file to load the drivers on boot by running the following command:

          `echo -e 'virtio_blk\nvirtio_net' > /etc/modules-load.d/virtio-drivers.conf`

      2. Regenerate the `initramfs` image by running the following command:

          `dracut -f`

      3. Restart the virtual server by using the `reboot` command.
      4. Log in as root with the password that you copied previously.
      5. Verify that the drivers are loaded by running the following command:

      `lsmod | grep virtio`

       Look for virtio_blk and virtio_net in the output.
6. Delete the existing network configuration on the RHEL virtual server to help make sure that it gets configured correctly when it starts.
   1. From the Web Console window, specify the following information:
      1. Delete the files that are in `/etc/sysconfig/network-scripts` by running the following command:

          `rm /etc/sysconfig/network-scripts/*`

      2. Delete the files that re in `/etc/NetworkManager/system-connections` by running the following command:

          `m /etc/NetworkManager/system-connections/*`

      3. Open `/etc/sysconfig/network` and delete any lines where _GATEWAYDEV_ is set.
7. Delete the existing Red Hat Subscription Manager configuration to help make sure that the virtual server gets reregistered when it starts.
   1. From the Web Console Window,
      1. Delete `/etc/rhsm/facts/uuid_override.facts` by running the following command:

      `rm /etc/rhsm/facts/uuid_override.facts`

8. Install Cloud-init on the RHEL virtual server
   1. From the Web Console window:
      1. Install `cloud-init` by running the following command:

      `dnf install -y cloud-init`

   2. Close the Web Console window.
9. Power off the RHEL virtual server by specifying the following information:
   1. From the side window, click **Data centers**
   2. From the list of available data centers, select your VDC.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9** from the list of available virtual servers.
   5. Click **Power Off**

## Migrating the RHEL virtual server

{: #virt-sol-vpc-migration-tutorial-migrate-rhel-vm}
{: step}

Use the following information to transfer the RHEL virtual server disk to your VPC and create a virtual server from the migrated disk.

When the virtual server starts, Cloud-init runs and the root password resets.
{: note}

1. Log in to the IBM Cloud console.
2. Create a detached RHEL boot volume by specifying the following information:
   1. From the **Navigation menu**, click **Infrastructure > Network > Subnets**.
   2. From the list of available resources, select **vpc-migration-sn-1**.
   3. Click **Attached resources**
   4. In the **Attached instances** section, click **Create**.
   5. In the **Location** section, specify the following information:
      1. For **Geography**, select **North America**.
      2. For **Region**, select **Dallas (us-south)**.
      3. For **Zone**, select **us-south-1**.
   6. In the **Details** section, specify the following information:
      1. For **Name**, enter `vpc-migration-vsi-rhel9`
   7. In the **Server configuration** section, specify the following information:
      1. For **Image**, click **Change image**. From the **Select an image** form, specify the following information:
         1. Search for **Red Hat**.
         2. Select **ibm-redhat-9-6-minimal-amd64-5** from the list of results.
         3. Click **Save**.
      2. For **SSH keys**, select **vpc-migration-ssh-key**.
   8. In the **Storage** section, click the **Edit** icon that is next to the item for the boot volume. With in the **Edit boot volume** form, specify the following information:
       1. In the **Details** section, specify the following information:
          1. For **Name**, enter `vpc-migration-vsi-rhel9-boot-volume`
          2. Set **Auto-delete** to **Disabled**
       2. In the **Profiles** section, select **General purpose**.
       3. In the **Storage capacity** section, specify the following information:
          1. For **Storage size**, enter a value that is greater than or equal to the storage size of the RHEL virtual server.
       4. Click **Save**.
   9. Click **Create a virtual server**.
   10. From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
   11. From the list of available resources, select **vpc-migration-vsi-rhel9**.
   12. Click **Delete**. In the **Delete Virtual server instance** form,
       1. Enter **Delete**.
       2. Click **Delete**.
3. Attach the boot volume to the worker virtual server by specifying the following information:
   1. From the **Navigation menu**, click **Infrastructure > Storage > Block storage volumes**.
   2. In the list of available resources, click **vpc-migration-vsi-rhel9-boot-volume**.
   3. In the **Attached virtual server** section, click **Attach**. Within the **Attach to a virtual server** form, specify the following information:
      1. From the list of virtual server, select **vpc-migration-vsi-worker**.
      2. Click **Attach**.
4. SSH into the worker virtual server by running the command that you used previously.

5. Get the name of the block device for the attached boot volume by running the following command:

    `lsblk -n -d -o NAME | tail -n 1`

6. Start the Netcat server, wait for an incoming disk transfer, and write it to the attached boot volume by running the following command:

    `nc -l 31337 | gunzip | dd of=/dev/<DEV_NAME> bs=16M status=progress`

    Where

    `<DEV_NAME>` is the name of the block device that you found previously.

7. Go to the VCFaaS tenant portal.
8. Get the IP of the RHEL virtual server by specifying the following information:
   1. From the side window, click **Data centers**.
   2. Click your VDC from the list of available data centers.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9** from the list of available virtual servers.
   5. Click **Details**.
   6. In the **Hardware** section, click **NICs**.
   7. Copy the IP address of the primary network interface.
9. Start into a Ubuntu Server shell from the RHEL virtual server by specifying the following information:
   1. From the side window, click **Data centers**
   2. Click your VDC from the list of available data centers.
   3. In the **Compute** section, click **Virtual machines**.
   4. Find **vm-rhel9** from the list of available virtual servers.
   5. Click **Details**
   6. Click **General**
   7. Click **Edit**
   8. In the **Boot options** section, specify the following information:
      1. Set **Enter boot setup** to **Enabled**.
      2. Click **Save**.
   9. Click **All actions**.
   10. In the **Media** section, click **Insert media** and select the Ubuntu Server ISO. Then, click **Insert**.
   11. Click **Power on** and wait for the virtual server to start.
   12. Click **Launch Web Console**. Within the **Web Console** window, specify the following information:
       1. Choose **EFI VMware Virtual IDE CDROM Drive**.
       2. Choose **Try or Install Ubuntu Server**.
       3. Wait for the language selection screen to load.
       4. Choose **Help**.
       5. Choose **Enter shell**.
10. Configure networking on the Ubuntu Server that is running on the RHEL virtual server
    1. Within the Web Console window, specify the following information:
       1. Add an IP to the primary network interface by running the following command:

           `ip addr add <RHEL_VM_IP>/24 dev ens192`

            Where

            `<RHEL_VM_IP>` is the IP of the RHEL virtual server that you copied previously.

       2. Add a default route through the routed VDC Network by running the following command:

           `ip route add default via 192.168.0.1`

11. Transfer the disk of the RHEL virtual server to the worker virtual server by specifying the following information:
    1. Within the Web Console window:
       1. Start the transfer of the RHEL virtual server disk by running the following command:

            `dd if=/dev/sda bs=16M status=progress | gzip | nc -v <WORKER_VSI_IP> 31337`

            Where

            `<WORKER_VSI_IP>` is the IP of the worker virtual server that you copied previously.

       2. Wait for the transfer to complete. You can monitor the progress of the transfer from either the Ubuntu Server shell or the worker virtual server.

       Both the Netcat client and server commands might not stop after the transfer completes. If they don't stop, you need to manually interrupt them.
       {: note}

    2. Close the Web Console window.
12. Fix the partition table on the attached boot volume by moving the partition table to the correct position on the disk by running the following command:

    `sgdisk --move-second-header /dev/<DEV_NAME>`

    Where

    `<DEV_NAME>` is the name of the block device that you found previously.

13. Flush the buffers of the attached boot volume by running the following command:

    `blockdev --flushbufs /dev/<DEV_NAME>`

    Where

    `<DEV_NAME>` is the name of the block device that you found previously.

14. Log in to the IBM Cloud console.
15. Create an RHEL virtual server from the attached boot volume
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
16. Get the IP of the RHEL virtual server, specify the following information:
    1. From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
    2. Search for **vpc-migration-vsi-rhel9**.
    3. From the list of results, copy the reserved IP of the RHEL virtual server.
17. Log in to the new RHEL virtual server
    1. SSH into the RHEL virtual server by running the following command:

    `ssh -J root@<BASTION_VSI_IP> root@<RHEL_VSI_IP>`

    Where

    `<BASTION_VSI_IP>` is the IP of the Bastion virtual server that you copied previously.
    `<RHEL_VSI_IP>` is the IP of the RHEL virtual server that you copied previously.

## You completed the tutorial

{: #virt-sol-vpc-migration-tutorial-congratulations}

You successfully migrated VCFaaS instances into a VPC. You can continue to develop your VPC by migrating or adding more virtual servers and other resources.
