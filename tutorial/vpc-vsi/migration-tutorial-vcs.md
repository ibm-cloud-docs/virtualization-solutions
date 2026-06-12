---

copyright:
  years: 2026
lastupdated: "2026-06-12"

keywords: Red Hat OpenShift Virtualization, virtual servers, Red Hat OpenShift Kubernetes Service, VSI, File Storage, Backup, Kasten, Veeam, volumes, VMware VCS to VPC migration, virtual machine disk transfer, virt-v2v conversion tool, transit gateway VCS VPC, virtIO drivers Windows RHEL, NSX firewall configuration, Ubuntu live boot migration, netcat disk transfer, cloud-init configuration, VMware workload migration


subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 60m

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from VMware Cloud Foundation for Classic to IBM Cloud Virtual Servers for VPC
{: #virt-sol-vcs-vpc-migration-tutorial-overview}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}

Migrate VMs from VMware Cloud Foundation to IBM Cloud VPC: configure transit gateway, transfer disks with netcat, convert with virt-v2v, and deploy on VPC.
{: shortdesc}

{{./../../_include-segments/objective.md}}

## Before you begin
{: #virt-sol-vpc-migration-tutorial-prerequisites}

This tutorial requires the following prerequisites.

* An available VMware&reg; vCenter Server&reg; (VCS) instance.
* Virtual Private Network (VPN) access to your VCS instance configured by using the [instructions](/docs/vmwaresolutions?topic=vmwaresolutions-trbl_timeout_vc_console).
* The correct access policies to manage VCS environments and the ability to create the following VCS resources. For more information, see [Managing Identity and Access Management (IAM) access for VCS](/docs/vmwaresolutions?topic=vmwaresolutions-iam&interface=ui).
   - Virtual machines
   - Networks
   - Firewall rules
   - Network Address Translation (NAT) rules
   - Transit gateway connection groups


{{./../../_include-segments/access.md}}

{{./../../_include-segments/create_vpc.md}}

{{./../../_include-segments/create_virtual_server.md}}

{{./../../_include-segments/create_bastion.md}}

{{./../../_include-segments/create_security_group.md}}


## Setting up networking on a VCS instance
{: #virt-sol-vpc-migration-tutorial-setup-VCS}
{: step}

Use the following steps to configure networking in your {{site.data.keyword.vmwaresolutions_full}} environment to enable outbound internet access to install the required packages on the virtual servers.

1. Find the credentials to NSX&reg;.
   1. Log in to the {{site.data.keyword.cloud}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **NSX Manager**, copy **http** credentials and **Fully Qualified Domain Name (FQDN)**.
2. Find the available Internet Protocol (IP) address for private Source Network Address Translation (SNAT).
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Infrastructure**.
   5. Scroll down to the **Network interface** and click **Private virtual local area network (VLAN)**.
   6. Find the subnet labeled `Portable private subnet for customer workload edge`.
       1. Note the IP addresses that are `Reserved`.
       2. Click the subnet and find an IP address that is `Usable` but does not exist in the `Reserved` list on the previous page.
       3. Copy this IP as it is used multiple times in the following steps.
3. Create the NAT rules in the NSX User Interface (UI).
    1. Log in to the NSX UI.
    2. Go to the **Networking** tab.
    3. Click **NAT** under **Network Services**.
    4. For **Gateway**, select `T1-workload-vcs-mf | Tier-1`.
    5. Verify that you see an SNAT rule that is named `T1-public-snat-workload-*` that exists and is enabled.
       - Copy the IP address under **Translated IP | Port**. This IP address is used in later steps.
    6. Click **ADD NAT RULE**.
       1. For **Name**, enter `T1-private-snat-workload`.
       2. For **Action**, choose **SNAT**.
       3. For **Source IP**, enter `192.168.0.0/16`.
       4. For **Translated IP | Port**, enter the IP that you chose in the previous step.
       5. For **Enabled**, choose **No**.
       6. Click **SAVE**.
4. Create gateway firewall rules for T1 in NSX UI.
   1. Log in to the NSX UI.
   2. Go to the **Security** tab.
   3. Click **Gateway Firewall** under **Policy management**.
   4. For **Gateway**, select `T1-workload-vcs-mf | Tier-1`.
   5. Click the checkbox for **Wkld_Drop_All_Policy_T1** and click **+ ADD RULE**.
      1. For **Name**, enter `allow 192`.
      2. For **Sources**:
         1. Click the edit icon.
         2. Click **IP addresses**.
         3. Enter `192.168.0.0/16`.
         4. Click **APPLY**.
      3. Click **PUBLISH**.
5. Create gateway firewall rules for T0 in NSX UI.
   1. Log in to the NSX UI.
   2. Go to the **Security** tab.
   3. Click **Gateway Firewall** under **Policy Management**.
   4. For **Gateway**, select `T0-workload-vcs-mf | Tier-0`.
   5. Click the checkbox for **Wkld_Drop_All_Policy_T0** and click **+ ADD RULE**.
      1. For **Name**, enter `allow 192`.
      2. For **Sources**:
         1. Click the edit icon.
         2. Click **IP addresses**.
         3. Enter the two converted IP addresses that are saved from before.
         4. Click **APPLY**.
      3. Click **PUBLISH**.

## Uploading the Ubuntu Server ISO into the VCS instance
{: #virt-sol-vpc-migration-tutorial-upload-ubuntu-iso}
{: step}

Use the following steps to create a catalog in VCS and upload an Ubuntu&reg; Server International Organization for Standardization (ISO) into it. This ISO is used later to start the virtual servers into a shell for disk migration.

1. Go to [Get Ubuntu Server](https://ubuntu.com/download/server){: external} and download the ISO of the most recent Long-Term Support (LTS) version of Ubuntu Server.
2. Log in to the {{site.data.keyword.cloud_notm}} console.
3. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
4. Log in to vCenter and upload the ISO image.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Click the data store icon.
   3. Click **management-share** or **vsanDatastore** depending on your VCS instance type.
   4. Make sure that the root folder is highlighted and click **UPLOAD FILES**.
   5. Choose the Ubuntu ISO.
   6. Click **Open**.
   7. Wait for the operation to finish.
   8. If you see a certificate error:
      1. Click the host in the error message and accept the certificate.

## Creating the Windows and RHEL virtual servers
{: #virt-sol-vpc-migration-tutorial-create-vms}
{: step}

Use the following steps to create Microsoft Windows&reg; and Red Hat&reg; Enterprise Linux&reg; (RHEL) virtual servers in VCS. These servers are migrated to the Virtual Private Cloud (VPC).

1. Find the credentials to vCenter&reg;.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
2. Log in to vCenter and deploy your Windows virtual server that is named `vm-win22` in your VCS instance.
   1. Make sure your Network Interface Card (NIC) is connected to the 192.168.0.0/16 NSX segment.
3. Open a Web Console window to the Windows virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-win22**.
   3. Click **Launch Web Console** and in the **Web Console** window, log in with Administrator credentials.
4. Try to reach the internet from the Windows virtual server.
   1. From the Web Console window, do the following actions:
       1. Open Microsoft Edge&reg;.
       2. Go to the Google home page.
       3. Verify that the page loads.
   2. Close the Web Console window.
5. Log in to vCenter and deploy your RHEL virtual server that is named `vm-rhel9` in your VCS instance.
   - Make sure that your NIC is connected to the 192.168.0.0/16 NSX segment.
6. Open a Web Console window to the RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-rhel9**.
   3. Click **Launch Web Console** and in the **Web Console** window, log in with root credentials.
7. Try to reach the internet from the RHEL virtual server.
   1. From the Web Console window, verify that you can resolve Google's domain name by running the following command:

      ```sh
      nslookup www.google.com
      ```
      {: pre}

   2. Close the Web Console window.

## Getting the ISO for the Windows virtIO drivers into the Windows virtual server
{: #virt-sol-vpc-migration-tutorial-load-virtio-iso}
{: step}

Use the following steps to obtain and transfer the ISO that contains the Windows virtIO drivers. These drivers help ensure compatibility with {{site.data.keyword.vpc_short}}. This ISO file on the Windows virtual server is used to fix the Windows recovery image in a later step.

1. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
2. Log in to vCenter and deploy your RHEL virtual server that is named `vm-rhel9-tmp` in your VCS instance.
   - Make sure that your NIC is connected to the 192.168.0.0/16 NSX segment.
3. Open a Web Console window to the temporary RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-rhel9-tmp**.
   3. Click **Launch Web Console** and in the **Web Console** window, log in with root credentials.
   4. Make sure that your RHEL is registered.
4. Install the `virtio-win` package on the RHEL virtual server.
   1. From the Web Console window, run the following command to install the package:

      ```sh
      dnf install -y virtio-win
      ```
      {: pre}

5. Get the IP address of the temporary RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-rhel9-tmp**.
   3. Under **virtual machine Details**, look for **IP addresses** and copy the 192.x.x.x address.
6. Copy the virtIO drivers ISO into the Windows virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-win22**.
   3. Click **Launch Web Console** and in the **Web Console** window, log in with Administrator credentials.
   4. Open the command prompt and copy the ISO from the RHEL virtual server by running the following command:

      ```sh
      scp root@<TEMP_RHEL_VM_IP>:/usr/share/virtio-win/virtio-win.iso C:\virtio-win.iso
      ```
      {: pre}

      Where `<TEMP_RHEL_VM_IP>` is the IP address of the temporary RHEL virtual server that you previously copied.

      When prompted whether you trust the remote server, enter **y** and enter the password of the temporary RHEL virtual server.

7. Keep the temporary RHEL virtual server and the virtio-win.iso file together, as they will be used later to transfer the ISO to the worker virtual server after VPC connectivity is established.

## Getting the cloud-init installer for the Windows virtual server
{: #virt-sol-vpc-migration-tutorial-load-cloud-init-installer}
{: step}

The Windows virtual server also needs Cloud-init installed on it to run as a virtual server in your VPC. Use the following steps to obtain the installer from the internet.

1. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
2. Open a Web Console window to the Windows virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-win22**.
   3. Click **Launch Web Console** and in the **Web Console** window, log in with Administrator credentials.
3. Download the cloud-init installer.
   1. From the Web Console window:
      1. Open Microsoft Edge.
      2. Go to [the download page for the installer](https://www.cloudbase.it/downloads/CloudbaseInitSetup_1_1_4_x64.msi){: external}. The download starts automatically.
   2. Close the Web Console window.

## Setting up a transit gateway (private connection) between the VCS instance and VPC
{: #virt-sol-vpc-migration-tutorial-setup-tgw}
{: step}

Use the following steps to create a transit gateway to securely connect your VCS instance and VPC over the {{site.data.keyword.cloud_notm}} private network.

1. Log in to the {{site.data.keyword.cloud_notm}} console.
2. Create a transit gateway.
   1. From the **Navigation menu**, click **Infrastructure > Network > Transit gateway**.
   2. Click **Create**.
   3. In the **Configuration** section, specify the following information:
      1. For **Name**, enter `vpc-migration-tgw`.
   4. In the **Location** section, specify the following information:
      1. For **Routing option**, select **Local routing**.
      2. For **Location**, select **Dallas (us-south)**.
      3. Click **Create**.
3. Connect the VPC to the transit gateway.
   1. From the **Navigation menu**, click **Infrastructure > Network > Transit gateway**.
   2. From the list of available resources, select **vpc-migration-tgw**.
   3. Click **Add connection** and within the form, specify the following information:
      1. For **Network connection**, select **VPC**.
      2. For **Connection reach**, select **Add a new connection to this account**.
      3. For **Region**, select **Dallas (us-south)**.
      4. For **Available connections**, select **vpc-migration**.
      5. Click **Add**.
4. Connect the VCS instance to the transit gateway.
   1. Find the subnet for the VCS instance.
      1. Log in to the {{site.data.keyword.cloud_notm}} console.
      2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
      3. From the list of available resources, select your VCS instance.
      4. Click the **Infrastructure** tab.
      5. Click the cluster where the virtual machine that you are migrating is located in the list of clusters.
      6. Scroll down to **Network interface** and find the subnet labeled **Portable private subnet for customer workload edge**.
      7. Record the Classless Inter-Domain Routing (CIDR) notation. For this document, it is referred to as `<CIDR>`.
   2. Update transit gateway with the VCS subnet.
      1. Log in to the {{site.data.keyword.cloud_notm}} console.
      2. From the **Navigation menu**, click **Infrastructure > Network > Transit gateway**.
      3. From the list of available resources, select **vpc-migration-tgw**.
      4. Click **Add connection** and within the form, specify the following information:
         1. For **Network connection**, select **Classic infrastructure**.
         2. For **Connection reach**, select **Add a new connection to this account**.
         3. Click **Prefix filtering (optional)**.
         4. Click **Create a prefix filter**.
            1. For **Action**, choose **Permit**.
            2. For **Network**, enter `<CIDR>`. `<CIDR>` is the name of the subnet that you found previously.
            3. Click **Save**.
            4. Click **Add**.
5. Create a security group to allow traffic between the virtual server on the VPC environment and the VCS instance.
   1. From the **Navigation menu**, click **Infrastructure > Network > Security groups**.
   2. Click **Create**.
   3. In the **Location** section, specify the following information:
      1. For **Geography**, select **North America**.
      2. For **Region**, select **Dallas (us-south)**.
   4. In the **Details** section, specify the following information:
      1. For **Name**, enter `vpc-migration-sg-tgw`.
      2. For **Virtual private cloud**, select **vpc-migration**.
   5. In the **Inbound rules** section, specify the following information:
      1. Click **Create** and within the form, specify the following information:
         1. For **Protocol**, select **Any protocol**.
         2. For **Source type**, select **IP or CIDR**.
         3. For **Source**, enter `<CIDR>`. `<CIDR>` is the name of the subnet that you found previously.
         4. Click **Create**.
   6. In the **Attaching virtual server interfaces** section, select the interface of the worker virtual server and click **Create a security group**.

## Getting the ISO for the Windows virtIO drivers into the worker virtual server
{: #virt-sol-vpc-migration-tutorial-load-virtio-iso-worker}
{: step}

Use the following steps to obtain and transfer the ISO that contains the Windows virtIO drivers onto the worker virtual server. These drivers help ensure compatibility with {{site.data.keyword.vpc_short}}. This ISO file on the worker virtual server is used to facilitate disk image conversion in a later step.

1. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
2. Get the IP address of the temporary RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-rhel9-tmp**.
   3. Under **virtual machine details**, look for **IP addresses** and copy the 192.x.x.x address.
3. Copy the virtIO drivers ISO into the worker virtual server.

   1. Secure Shell (SSH) into the worker virtual server.

      ```bash
      ssh -J root@<BASTION_VSI_IP> root@<WORKER_VSI_IP>
      ```
      {: codeblock}

   2. SCP over the virtio drivers ISO file.

      ```bash
      scp root@<TEMP_RHEL_VM_IP>:/usr/share/virtio-win/virtio-win.iso /root/virtio-win.iso
      ```
      {: codeblock}

      Where `<TEMP_RHEL_VM_IP>` is the IP address of the temporary RHEL virtual server that you previously copied.

      When prompted whether you trust the remote server, enter **yes** and enter the password of the temporary RHEL virtual server.

## Preparing the Windows virtual server for migration
{: #virt-sol-vpc-migration-tutorial-prep-windows-vm}
{: step}

Before you can migrate the virtual server, you must install the necessary drivers and configure Cloud-init on the Windows virtual server. Use the following steps to prepare the Windows virtual server for migration.

1. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
2. Open a Web Console window to the Windows virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-win22**.
   3. Click **Launch Web Console** and in the **Web Console** window, log in with Administrator credentials.
3. Install the virtIO drivers on the recovery image of the Windows virtual server by following the steps that are in [Making the virtio-win drivers available in the recovery image](/docs/vpc?topic=vpc-create-windows-custom-image#virtio-win-drivers-windows-recovery-image). Use the command prompt, not PowerShell to run the commands noted in the linked documentation. The driver files added to the recovery image come from mounting the `C:\virtio-win.iso` file that was copied over to the Windows virtual server in an earlier step.

   Keep in mind that if your virtual server has a GPT partition table, you must set the partition IDs to UUIDs, not numbers. Get the correct IDs by displaying the details of the System and Recovery partitions while `diskpart` is running. For more information, see the [documentation on the detail partition command](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/detail-partition){: external}.

   If your virtual server does not show any files in the recovery drive after you assign a drive letter to the recovery partition, check the C:\Windows\System32\Recovery folder. The Winre.wim file only appears after you disable `reagentc`. The file is still hidden and can only be viewed by using the `dir /a` command.

4. Install and configure Cloud-init on the Windows virtual server.
   1. From the Web Console window, open the **File Explorer**.
   2. Click **Downloads**.
   3. Double-click the Cloud-init installer file that you downloaded previously and follow the prompts to install the package. Make sure that you use `Administrator` for the name of the Cloud-init user, not Admin.
   4. After the installation finishes, in the installation wizard, do not check any boxes to run sysprep. Click finish.
   5. Modify the Cloud-init configuration files by following the steps that are in [Customizing a virtual server](/docs/vpc?topic=vpc-create-windows-custom-image#customize-virtual-machine). **Do not** run the `sysprep` steps, stop after you modify the files.
5. Shut down the Windows virtual server and close the Web Console.

## Preparing the RHEL virtual server for migration
{: #virt-sol-vpc-migration-tutorial-prep-rhel-vm}
{: step}

You need to configure the RHEL virtual server with virtIO drivers and Cloud-init to prepare it for migration. The installation of the RHEL virtIO drivers depends on whether they are included in the kernel or whether they need to load as modules. These steps assume that the drivers need to be loaded and that they aren't loaded.

Use the following information to prepare the RHEL virtual server for migration.

1. Log in to the {{site.data.keyword.cloud_notm}} console.
2. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**
   5. Under **vCenter**, copy **ADMIN** credentials.
3. Log in to vCenter and open a Web Console window to the virtual server named `vm-rhel9`.
4. Install Cloud-init on the RHEL virtual server.
   1. From the Web Console window, install `cloud-init` by running the following command:

      ```sh
      dnf install -y cloud-init
      ```
      {: pre}

5. Delete the existing network configuration on the RHEL virtual server to help ensure that it gets configured correctly when it starts.
   1. From the Web Console window:
      1. Delete the files that are in `/etc/sysconfig/network-scripts` by running the following command:

         ```sh
         rm /etc/sysconfig/network-scripts/*
         ```
         {: pre}

      2. Delete the files that are in `/etc/NetworkManager/system-connections` by running the following command:

         ```sh
         rm /etc/NetworkManager/system-connections/*
         ```
         {: pre}

      3. Open `/etc/sysconfig/network` and delete any lines where `GATEWAYDEV` is set.

6. Delete the existing Red Hat Subscription Manager configuration to help ensure that the virtual server gets reregistered when it starts.
   1. From the Web Console window, delete `/etc/rhsm/facts/uuid_override.facts` by running the following command:

      ```sh
      rm /etc/rhsm/facts/uuid_override.facts
      ```
      {: pre}

   2. Close the Web Console window.

7. Power off the RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find `vm-rhel9`, right-click, and choose **Power** > **Power Off**.

{{./../../_include-segments/start_netcat_server.md}}

## Allowing VCS instance onto the private network
{: #virt-sol-vpc-migration-tutorial-prep-windows-vm}
{: step}

1. Enable SNAT rule to allow from VCS instance to VPC.
   1. Find the credentials to NSX.
      1. Log in to the {{site.data.keyword.cloud_notm}} console.
      2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
      3. From the list of available resources, select your VCS instance.
      4. Go to **Access information**.
      5. Under **NSX Manager**, copy **http** credentials and **FQDN (Fully Qualified Domain Name)**.
   2. Disable public SNAT rule.
      1. Log in to the NSX UI.
      2. Go to the **Networking** tab.
      3. Click **NAT** under **Network Services**.
      4. For **Gateway**, select `T1-workload-vcs-mf | Tier-1`.
      5. Verify that you see an SNAT rule that is named `T1-public-snat-workload-*` that exists and is disabled.
   3. Enable private SNAT rule.
      1. Log in to the NSX UI.
      2. Go to the **Networking** tab.
      3. Click **NAT** under **Network Services**.
      4. For **Gateway**, select `T1-workload-vcs-mf | Tier-1`.
      5. Verify that you see an SNAT rule that is named `T1-private-snat-workload-*` that exists and is enabled.
2. Get the IP address range of the VPC subnet.
   1. From the **Navigation menu**, click **Infrastructure > Network > Subnets**.
   2. From the list of available resources, select **vpc-migration-sn-1**.
   3. Copy the value of the **IP range**.
3. Open a Web Console window to the RHEL virtual server.
4. Try to reach an IP address in the VPC subnet from the VCS instance.
   1. From the Web Console window, verify that you can ping the worker virtual server by running the following command:

      ```sh
      ping <WORKER_VSI_IP>
      ```
      {: pre}

      Where `<WORKER_VSI_IP>` is the IP address of the worker virtual server that you copied previously.

   2. Close the Web Console window.

5. Try to reach an IP address in the routed VCS network from the VPC.
   1. SSH into the worker virtual server by running the command that you used previously.
   2. Verify that you can ping the gateway of the routed VCS network.
   3. Find the gateway of `<CIDR>` and try to ping the IP address. `<CIDR>` is the name of the subnet that you found previously.

## Migrating the Windows virtual server
{: #virt-sol-vpc-migration-tutorial-migrate-windows-vm}
{: step}

1. Find the credentials to vCenter.
   1. Log in to the {{site.data.keyword.cloud_notm}} console.
   2. From the **Navigation menu**, click **VMware > Resources > VCF for Classic**.
   3. From the list of available resources, select your VCS instance.
   4. Go to **Access information**.
   5. Under **vCenter**, copy **ADMIN** credentials.
2. Get the IP address of the Windows virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-win22**.
   3. Under **virtual machine details**, look for **IP addresses** and copy the 192.x.x.x address.

3. Start into an Ubuntu Server shell from the Windows virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client&reg;** and choose **inventory**.
   2. Find **vm-win22**, right-click, and choose **Edit Settings**.
      1. Expand **CD/DVD drive 1**.
         1. For **CD/DVD drive 1**, click the dropdown and choose **data store ISO File**.
            1. For the **Select File** screen, select **management-share** or **vsanDatastore** depending on your VCS instance type.
            2. Choose the Ubuntu ISO that was downloaded earlier.
         2. For **Status**, click the checkbox for **Connect At Power On**.
         3. Click **OK**.
      2. Click **OK**.
   3. From the vCenter UI, find `vm-win22`, right-click, and choose **Power** > **Power On**.
   4. Click **Launch Web Console**. Within the **Web Console** window, specify the following information:
      1. Choose **Extensible Firmware Interface (EFI) VMware Virtual Integrated Drive Electronics (IDE) CD-ROM Drive**.
      2. Choose **Try or Install Ubuntu Server**.
      3. Wait for the language selection screen to load.
      4. Choose **Help**.
      5. Choose **enter shell**.
4. Configure networking on the Ubuntu Server that is running on the Windows virtual server.
   1. Within the Web Console window, add an IP address to the primary network interface by running the following command:

      ```sh
      ip addr add <WINDOWS_VM_IP>/24 dev ens192
      ```
      {: pre}

      Where `<WINDOWS_VM_IP>` is the IP address of the Windows virtual server that you copied previously.

   2. Add a default route through the routed Virtual Data Center (VDC) network by running the following command:

      ```sh
      ip route add default via 192.168.0.1
      ```
      {: pre}

5. Transfer the disk of the Windows virtual server to the worker virtual server.

   1. Within the Web Console window, start the transfer of the Windows virtual server disk by running the following command:

      ```sh
      dd if=/dev/sda bs=16M status=progress | gzip | nc -v <WORKER_VSI_IP> 31337
      ```
      {: pre}

      Where `<WORKER_VSI_IP>` is the IP address of the worker virtual server that you copied previously.

   2. Wait for the transfer to complete. You can monitor the progress of the transfer from either the Ubuntu Server shell or the worker virtual server.

      Both the Netcat client and server commands might not stop after the transfer completes. If they don't stop, you need to manually interrupt them.
      {: note}

   3. Close the Web Console window.

6. On the worker virtual server, fix the partition table on the attached boot volume by moving the partition table to the correct position on the disk by running the following command:

   ```sh
   sgdisk --move-second-header /dev/<DEV_NAME>
   ```
   {: pre}

   Where `<DEV_NAME>` is the name of the block device that you found previously.

7. Flush the buffers of the attached boot volume by running the following command:

   ```sh
   blockdev --flushbufs /dev/<DEV_NAME>
   ```
   {: pre}

   Where `<DEV_NAME>` is the name of the block device that you found previously.

8. Install utilities to use the virt-v2v-in-place tool.

   ```sh
   apt-get install -y virt-v2v
   apt-get install -y rhsrvany
   ```
   {: pre}

9. Make a symlink to the mounted Windows boot volume. The virtual server name here corresponds to the example Windows virtual server name that was created in this document. If a virtual server of a different name is being migrated, use the name of that virtual server instead for the VM_NAME variable.

   ```bash
   VM_NAME=vm-win22
   TARGET_DEV=/dev/<DEV_NAME>
   SYMLINK="/tmp/${VM_NAME}-sda"
   ln -fs "${TARGET_DEV}" "${SYMLINK}"
   ls -l "${SYMLINK}"
   ```
   {: codeblock}

   Where `<DEV_NAME>` is the name of the block device that you found previously.

10. Run `virt-v2v-in-place`. The virtio-win.iso file that is copied over from an earlier step is referenced with the `VIRTIO_WIN` variable here.

    ```bash
    export LIBGUESTFS_BACKEND=direct
    export VIRTIO_WIN=/root/virtio-win.iso
    virt-v2v-in-place -i disk "${SYMLINK}" --block-driver virtio-scsi -v
    ```
    {: codeblock}

11. Make sure that the conversion was successful.

   Check that the return code is 0. The following command displays 0 if the conversion was successful:

      echo $?
   {: pre}

    If the `virt-v2v-in-place` command was not successful, the image on the mounted boot volume is in an indeterminate state. Check the output for any error messages. Any issue needs to be resolved first. Afterwards, the data from the source Windows virtual server needs to be re-transferred to the mounted boot volume on the worker virtual server before retrying the `virt-v2v-in-place` command.
12. Log in to the {{site.data.keyword.cloud_notm}} console.
13. Create a Windows virtual server from the attached boot volume.
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
          1. For **Name**, enter `vpc-migration-vsi-win22`.
       5. In the **Server configuration** section, for **SSH Keys**, select **vpc-migration-ssh-key**.
       6. Click **Create a virtual server**.
14. Get the IP address of the Windows virtual server.
    1. From the **Navigation menu**, click **Infrastructure > Compute > Virtual server instances**.
    2. Search for **vpc-migration-vsi-win22**.
    3. Copy the reserved IP address of the Windows virtual server from the list of results.
15. Log in to the Windows virtual server.
    1. Display the Remote Desktop Protocol (RDP) port of the Windows virtual server through the bastion virtual server by running the following command:

       ```sh
       ssh -L 3389:<WINDOWS_VSI_IP>:3389 <BASTION_VSI_IP> -l root -N
       ```
       {: pre}

       Where `<WINDOWS_VSI_IP>` is the IP address of the Windows virtual server that you copied previously and `<BASTION_VSI_IP>` is the IP address of the bastion virtual server that you copied previously.

    2. Use your preferred Remote Desktop client to connect to the Windows virtual server. Use `localhost` as the IP address and log in as Administrator with the password of the Windows virtual server.

{{./../../_include-segments/create_boot_disk.md}}

## Understanding Block Storage for VPC volumes
{: #virt-sol-vpc-migration-tutorial-block-storage}
{: step}

Before migrating the RHEL virtual server, it's important to understand the Block Storage for VPC volumes that are being used in this migration:

- **Boot volumes**: The volumes created in this tutorial are boot volumes that contain the operating system. Boot volumes are automatically attached during instance creation and can range from 10 GB to 32,000 GB depending on the profile used.
- **Volume profiles**: This tutorial uses Block Storage for VPC volumes. You can choose between traditional tiered profiles (3iops-tier, 5iops-tier, 10iops-tier) or the SDP (Defined Performance) profile for custom IOPS and throughput.
- **Volume encryption**: All volumes are encrypted by default with IBM-managed encryption. You can optionally use customer-managed encryption with your own root keys.
- **Volume limits**: Each account can create up to 300 volumes per region, and each virtual server instance can have up to 12 data volumes attached (plus one boot volume).
- **Performance**: Each volume has dedicated performance allocation, eliminating "noisy neighbor" issues common in shared storage environments.

For more information, see [About Block Storage for VPC](/docs/vpc?topic=vpc-block-storage-about).

## Migrating the RHEL virtual server
{: #virt-sol-vpc-migration-tutorial-migrate-rhel-vm}
{: step}

1. Go to the vCenter portal.
2. Get the IP address of the RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find and click **vm-rhel9**.
   3. Under **virtual machine Details**, look for **IP addresses** and copy the 192.x.x.x address.
3. Start into an Ubuntu Server shell from the RHEL virtual server.
   1. From the vCenter UI, click the icon in the upper left next to **vSphere Client** and choose **inventory**.
   2. Find **vm-rhel9**, right-click, and choose **Edit Settings**.
      1. Expand **CD/DVD drive 1**.
         1. For **CD/DVD drive 1**, click the dropdown and choose **data store ISO file**.
            1. For the **Select File** screen, select **management-share** or **vsanDatastore** depending on your VCS instance type.
            2. Choose the Ubuntu ISO that was downloaded earlier.
         2. For **Status**, click the checkbox for **Connect At Power On**.
         3. Click **OK**.
      2. Click **OK**.
   3. From the vCenter UI, find `vm-rhel9`, right-click, and choose **Power** > **Power On**.
   4. Click **Launch Web Console**. Within the **Web Console** window, specify the following information:
      1. Choose **EFI VMware Virtual IDE CD-ROM Drive** (Extensible Firmware Interface VMware Virtual Integrated Drive Electronics CD-ROM Drive).
      2. Choose **Try or Install Ubuntu Server**.
      3. Wait for the language selection screen to load.
      4. Choose **Help**.
      5. Choose **enter shell**.
4. Configure networking on the Ubuntu Server that is running on the RHEL virtual server.
   1. Within the Web Console window, add an IP address to the primary network interface by running the following command:

      ```sh
      ip addr add <RHEL_VM_IP>/24 dev ens192
      ```
      {: pre}

      Where `<RHEL_VM_IP>` is the IP address of the RHEL virtual server that you copied previously.

   2. Add a default route through the routed VCS network by running the following command:

      ```sh
      ip route add default via 192.168.0.1
      ```
      {: pre}

5. Transfer the disk of the RHEL virtual server to the worker virtual server.
   1. Within the Web Console window, start the transfer of the RHEL virtual server disk by running the following command:

      ```sh
      dd if=/dev/sda bs=16M status=progress | gzip | nc -v <WORKER_VSI_IP> 31337
      ```
      {: pre}

      Where `<WORKER_VSI_IP>` is the IP address of the worker virtual server that you copied previously.

   2. Wait for the transfer to complete. You can monitor the progress of the transfer from either the Ubuntu Server shell or the worker virtual server.

      Both the Netcat client and server commands might not stop after the transfer completes. If they don't stop, you need to manually interrupt them.
      {: note}

   3. Close the Web Console window.

6. On the worker virtual server, fix the partition table on the attached boot volume by moving the partition table to the correct position on the disk by running the following command:

   `sgdisk --move-second-header /dev/<DEV_NAME>`

   Where

   `<DEV_NAME>` is the name of the block device that you found previously.

7. Flush the buffers of the attached boot volume by running the following command:

   `blockdev --flushbufs /dev/<DEV_NAME>`

   Where

   `<DEV_NAME>` is the name of the block device that you found previously.

{{./../../_include-segments/create_vsi_rhel.md}}

{{./../../_include-segments/next_step.md}}
