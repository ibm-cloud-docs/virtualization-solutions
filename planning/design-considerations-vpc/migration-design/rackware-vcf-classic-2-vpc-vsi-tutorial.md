---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-06"

keywords: VSI, File Storage, Block Storage, Encryption, Migration, RackWare, RMM

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 60m
use-case: ApplicationModernization
industry: SoftwareAndPlatformApplications
compliance: HIPPA

---

{{site.data.keyword.attribute-definition-list}}

# Migrating from IBM Cloud VMware VCF-Automated to VPC VSI with RackWare RMM Tutorial
{: #virt-sol-vpc-migration-design-rmm-tutorial}
{: #tutorial-migration-toolkit}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}
{: toc-use-case="ApplicationModernization"}
{: toc-industry="Software and platform applications"}
{: toc-compliance="HIPPA"}

This tutorial focuses on using RackWare RMM (RackWare Management Module) to migrate IBM Cloud VCF-Automated Virtual Machines (VMs) to IBM Cloud VPC Virtual Server Instances. There is an associated Technical Guide that describes the technologies used.
{: shortdesc}

While RMM has an auto-provision feature to automatically provision an IBM Cloud VPC virtual server instance, appropriately sized, this feature was not used in this tutorial. Therefore, in this guide we manually provision the target virtual server instances and associated data disk.

RackWare's RMM Server Migration solution provides an easy, automated, and simplified process to migrate existing IBM Cloud VMware VCF-Automated virtual machines (VM) from their current location to IBM Cloud VPC virtual server instances.

The RackWare Management Module (RMM) migration solution provides a seamless virtual-to-virtual re-platforming for these VMware virtual machines to IBM Cloud virtual server instance migration. Its intuitive GUI allows you to move the OS, application, and data from VMware ESXi to IBM Cloud VPC virtual server instance.

1. Create Source virtual machines.
1. RMM Install:
      1. Create a VPC.
      1. Create subnet for the RMM server.
      1. Create a SSH key.
      1. Deploy the RMM server from the IBM Cloud catalog.
1. Obtain Licensing from RackWare.
1. Create a Transit Gateway with connections to Classic and VPC.
1. Order a portable private subnet.
1. Order a static private subnet.
1. Deploy a virtual machine for use as a bridge server.
1. Configure bridge server.
1. Generate the RMM’s ssh keys
1. Gather information about the source virtual machines.
1. Order the target virtual server instances.
1. Prepare the source virtual machine.
1. Use the RMM GUI to configure the migrations and perform the initial migration.
1. Verification.
1. Perform delta syncs.
1. Cut-over.

## Before you begin
{: #virt-sol-vpc-migration-design-rmm-before-you-begin}

In this tutorial we migrate an Ubuntu virtual machine and an a Microsoft Windows 2019 virtual machine. This tutorial assumes that you have:

1. Read [Migrating from IBM Cloud VMware VCF-Automated to VPC VSI with RackWare RMM Technical Guide](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-rmm-guide).
1. Have existing IBM Cloud VMware-Automated instance and hosting virtual machines on NSX overlay segments. These overlay segments do not have native access to the IBM Cloud Classic private network.
1. Created one or more resource groups.
1. Created a VPC with prefixes that cover your required networks.
1. Created a subnet for the RMM server.
1. Created an SSH key and uploaded the public key to VPC SSH keys.
1. Provisioned an RMM server from the IBM catalog.

## Create Source virtual machines
{: #virt-sol-vpc-migration-design-rmm-tutorial-create-vm}
{: step}

First, you need to set up your source virtual machine. This section has examples for the following:

- [Ubuntu source virtual machine](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-rmm-tutorial#virt-sol-vpc-migration-design-rmm-tutorial-ubuntu-vm).
- [Windows source virtual machine](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-rmm-tutorial#virt-sol-vpc-migration-design-rmm-tutorial-step1-windows).

### Ubuntu Source virtual machine
{: #virt-sol-vpc-migration-design-rmm-tutorial-ubuntu-vm}
{: step}

Deploy a virtual machine on the IBM Cloud VCF-Automated instance with the following specification:

- CPU: 2
- Memory: 1GB
- Hard disk 1: 10GB
- Hard disk 2: 100GB
- Network adapter 1: T1-192-168-10-0-workload
- Firmware: EFI
- OS: Ubuntu 22.04
- IP address: 192.168.10.11

The second disk was formatted and a test file written using the following commands:

```bash
# Format entire disk (no partition table)
sudo mkfs.ext4 /dev/sdb

# Mount it
sudo mkdir -p /mnt/sdb
sudo mount /dev/sdb /mnt/sdb

# Write file
echo "Test content" | sudo tee /mnt/sdb/testfile.txt
```
{: codeblock}

### Windows Source virtual machine
{: #virt-sol-vpc-migration-design-rmm-tutorial-step1-windows}

A virtual machine was deployed on the IBM Cloud VCF-Automated instance with the following specification:

- CPU: 2
- Memory: 4GB
- Hard disk 1: 90GB
- Network adapter 1: T1-192-168-10-0-workload
- Firmware: EFI
- OS: Windows 2019
- IP address: 192.168.10.12

No second disk was provisioned for this virtual machine.

## RMM Install
{: #virt-sol-vpc-migration-design-rmm-tutorial-rmm-install}
{: step}

Prior to installing RMM, you first set up the VPC infrastructure. At a bare minimum, you must set up a VPC, subnets, and the corresponding virtual server instances that you are planning to migrate. The new target virtual server instance profile (vCPU and vMemory) does not need to match the source. However, for the storage, the target storage needs to be the same or greater than the used space on the source VM's filesystem in size. The RMM's right-sizing option should be used to facilitate this change.

Use the VPC product documentation to:

1. [Create a VPC](/docs/vpc?topic=vpc-creating-vpc-resources-with-cli-and-api&interface=cli#create-a-vpc-cli).
1. [Create subnet](/docs/vpc?topic=vpc-creating-vpc-resources-with-cli-and-api&interface=cli#create-a-subnet-cli) for the RMM server.
1. [Create a SSH key](/docs/vpc?topic=vpc-creating-vpc-resources-with-cli-and-api&interface=cli#add-ssh-key-cli).
1. Deploy the RMM server from the IBM Cloud catalog.

   The RMM server has a public IP address for connectivity and a default login.

1. After you deploy, log in to the RMM server.
1. In the RMM server, change the default password, create users, and create an SSH key.
1. Upload the SSH key to IBM Cloud VPC.

You may need to update the RMM version, if so via email to RackWare, request access to their FTP repository and instructions to upgrade

## Obtain Licensing from RackWare
{: #virt-sol-vpc-migration-design-rmm-tutorial-rackware-license}
{: step}

1. Obtain licenses from RackWare by emailing the generated preinstall file to RackWare licensing.
1. To generate a preinstall file in `/etc/rackware`, run the following command using your public key name and IP address.

   ```bash
   ssh -i ~/.ssh/sno3 root@161.156.171.81
   rwadm relicense
   ```
   {: codeblock}

   Example output is shown below, please note that RackWare recommend installing RMM on RHEL or Rocky 8.x servers:

   ```text
   CentOS Linux release 7.9.2009 (Core)
   Found supported RedHat/CentOS release.
   CentOS Linux release 7.9.2009 (Core)

   WARNING: This command will generate a new preinstall file, but will also INVALIDATE the existing license on next RMM restart.
   If you wish to continue using RMM till you get the new license, DO NOT STOP RMM after running this command.
   Do you wish to continue? (Y/N)  [N]: Y
   PreInstall file generated at /etc/rackware/rwlicense_preinstall_1765474883. Please email this file to licensing@rackwareinc.com to get the license.
   ```
   {: screen}

1. Copy the file from `/etc/rackware` and send it to `licensing@rackwareinc.com` as an attachment.

   ```bash
   scp -i ~/.ssh/sno3 root@161.156.171.81:'/etc/rackware/rwlicense_preinstall_*' /Work/2025/RMM/
   ```
   {: pre}

1. After receiving a valid license, download the license file to `/etc/rackware` and restart the services to apply the license by running the following command, note your filename will be different:

   ```bash
   scp -i ~/.ssh/sno3 /Work/2025/RMM/rwlicense_1765474883_uk_ibm_POC_Mig root@161.156.171.81:/etc/rackware/
   ssh -i ~/.ssh/sno3 root@161.156.171.81
   rwadm restart
   ```
   {: codeblock}

1. Verify the license by running the following command and return the output to `licensing@rackwareinc.com`:

   ```bash
   rw rmm show
   ```
   {: pre}

## Create a Transit Gateway with connections to Classic and VPC
{: #virt-sol-vpc-migration-design-rmm-tutorial-transit-gateway}
{: step}

An IBM Cloud Transit Gateway is a fully managed hub-and-spoke networking service that provides centralized, private connectivity between IBM Cloud infrastructure environments including IBM Cloud VPCs and IBM Cloud Classic infrastructure.

It acts as a routing hub where multiple VPCs and Classic connections attach, allowing traffic to flow securely without requiring complex VPN meshes or manual route management.

Connections to VPCs use native VPC attachments that automatically exchange routes, while Classic connectivity is established via a Classic Infrastructure connection that integrates with the IBM Cloud backbone, enabling seamless private IP communication between VPC subnets and Classic VLANs.

Using the IBM Cloud documentation create a local transit gateway and connect your VPC and your IBM Cloud Classic network. For more information, see [Creating a transit gateway](/docs/transit-gateway?topic=transit-gateway-ordering-transit-gateway&interface=cli).

## Order a portable private subnet
{: #virt-sol-vpc-migration-design-rmm-tutorial-order-subnet}
{: step}

An IBM Cloud Classic portable private subnet is a block of private IP addresses that can be assigned to your Classic resources. While you already have a number of subnets associated with your IBM Cloud VMware-Automated instance, the IP addresses on these subnets are assigned by the automation and you should not manually assign IP addresses from these subnets.

It is good practice to order a new portable private subnet and assign IP addresses to virtual machines as needed. This subnet will be used to host the outside interfaces of the one or more RMM bridge servers, therefore, only a small subnet is required.

Using the IBM Cloud documentation order a new portable private subnet in the IBM Cloud Private VLAN named `Private management VLAN`. For more information, see [Customer-owned subnets for Classic](/docs/subnets?topic=subnets-customer-owned-subnets).

Assign an IP address for the RMM Bridge Server deployed later in this tutorial.

## Order a static private subnet
{: #virt-sol-vpc-migration-design-rmm-tutorial-order-static-subnet}
{: step}

An IBM Cloud Classic Static Subnet provides a block of IP addresses that are permanently routed to a specific endpoint, in our use case the RMM Bridge Server.

All IP addresses in the subnet are useable e.g. a /30 subnet has 4 useable IP addresses or a /29 subnet has 8 usable IP addresses.

Using the IBM Cloud documentation order a new static private subnet in the IBM Cloud Private VLAN named `Private management VLAN` and targeting the IP address in the portable private subnet that you will use for the RMM Bridge Server. For more information, see [Customer-owned subnets for Classic](/docs/subnets?topic=subnets-customer-owned-subnets).

## Deploy a virtual machine for use as a bridge server
{: #virt-sol-vpc-migration-design-rmm-tutorial-deploy-bridge-server}
{: step}

A virtual machine was deployed on the IBM Cloud VCF-Automated instance with the following specification:

- CPU: 2
- Memory: 1GB
- Hard disk 1: 10GB
- Network adapter 1: T1-192-168-10-0-workload
- Network adapter 2: mgmt-dpg-mgt
- Firmware: EFI
- OS: Ubuntu 22.04
- IP address 1: 192.168.10.254
- IP address 2: 10.134.54.62

1. To access your Ubuntu virtual machine via SSH you may need to configure SSH:

   1. Use the following command to configure SSH.

      ```bash
      sudo nano /etc/ssh/sshd_config
      ```
      {: pre}

   1. Make sure these lines are set:

      ```bash
      PasswordAuthentication yes
      PubkeyAuthentication yes
      ChallengeResponseAuthentication no
      UsePAM yes
      KbdInteractiveAuthentication yes
      ```
      {: codeblock}

   1. Using **Netplan** (default on Ubuntu 18.04+), edit your network config:

      ```bash
      sudo nano /etc/netplan/50-cloud-init.yaml
      ```
      {: pre}

      Example configuration with two NICs:

      ```yaml
      network:
        version: 2
        renderer: networkd
        ethernets:
          ens192:  # NIC 1 - Inside network
            addresses:
              - 192.168.10.254/24
            routes:
              - to: default
                via: 192.168.10.1
            nameservers:
              addresses: [8.8.8.8, 8.8.4.4]

          ens224:  # NIC 2 - Outside network
            addresses:
              - 10.134.54.62/26
            routes:
              - to: 10.0.0.0/8
                via: 10.134.54.1
      ```
      {: codeblock}

   1. Apply the configuration:

      ```bash
      sudo netplan apply
      ```
      {: pre}

   1. Verify interfaces:

      ```bash
      ip addr show
      ip route show
      ```
      {: codeblock}

## Configure bridge server
{: #virt-sol-vpc-migration-design-rmm-tutorial-bridge-server}
{: step}

This example uses an Ubuntu virtual machine hosted on the IBM Cloud VCF-Automated instance. The virtual machine is deployed with two interfaces.

1. Connect to the virtual machine via SSH:
   1. Update the system with the following command

      ```bash
      sudo apt update && sudo apt upgrade -y
      ```
      {: pre}

   1. Configure IP Forwarding using the following command.

      ```bash
      sudo sysctl -w net.ipv4.ip_forward=1
      sudo sysctl -p
      ```
      {: codeblock}

   1. Verify the connection using the following command

      ```bash
      cat /proc/sys/net/ipv4/ip_forward
      ```
      {: pre}

   This should return a 1.
   1. Identify Your Network Interfaces using the following command.

      ```bash
      ip addr show
      ```
      {: pre}

      Here is an example of what you might see with that command

      ```text
      ens192: Source VM network (192.168.10.0/24) - "inside" interface
      ens224: RMM network (10.134.54.0/26) - "outside" interface
      ```
      {: screen}

   1. Configure Static NAT (1-to-1 mapping), such as the following example using `VM1: NAT IP: 10.194.177.82, Real IP: 192.168.10.11`.

      ```bash
      # Install iptables-persistent
      sudo apt install iptables-persistent -y

      # Clear existing rules first
      sudo iptables -F
      sudo iptables -t nat -F

      # Enable forwarding
      sudo iptables -P FORWARD ACCEPT

      # Static NAT for VM1 (192.168.10.11 <-> 10.194.177.82)
      # Destination NAT (DNAT): Incoming traffic to 10.194.177.82 goes to 192.168.10.11
      sudo iptables -t nat -A PREROUTING -i ens224 -d 10.194.177.82 -j DNAT --to-destination 192.168.10.11

      # Source NAT (SNAT): Outgoing traffic from 192.168.10.11 appears as 10.194.177.82
      sudo iptables -t nat -A POSTROUTING -o ens224 -s 192.168.10.11 -j SNAT --to-source 10.194.177.82

      # Static NAT for VM1 (192.168.10.12 <-> 10.194.177.83)
      # Destination NAT (DNAT): Incoming traffic to 10.194.177.83 goes to 192.168.10.12
      sudo iptables -t nat -A PREROUTING -i ens224 -d 10.194.177.83 -j DNAT --to-destination 192.168.10.12

      # Source NAT (SNAT): Outgoing traffic from 192.168.10.12 appears as 10.194.177.83
      sudo iptables -t nat -A POSTROUTING -o ens224 -s 192.168.10.12 -j SNAT --to-source 10.194.177.83

      # Allow forwarding between interfaces for these specific IPs
      sudo iptables -A FORWARD -s 192.168.10.11 -j ACCEPT
      sudo iptables -A FORWARD -d 192.168.10.11 -j ACCEPT
      sudo iptables -A FORWARD -s 192.168.10.12 -j ACCEPT
      sudo iptables -A FORWARD -d 192.168.10.12 -j ACCEPT

      # Allow established connections
      sudo iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT

      # Add IP aliases for NAT addresses on ens224
      # The bridge server owns these IPs directly, so it responds to ARP requests
      sudo ip addr add 10.194.177.82/32 dev ens224
      sudo ip addr add 10.194.177.83/32 dev ens224
      ```
      {: codeblock}

## Generate the RMM’s ssh keys
{: #virt-sol-vpc-migration-design-rmm-tutorial-ssh-keys}
{: step}

1. SSH to the RMM Server

   ```bash
   ssh -i ~/.ssh/sno3 root@161.156.171.81
   ```
   {: pre}

1. From the RMM’s console window generate an SSH key pair.

   ```bash
   # Generate RSA key (4096-bit) without prompts
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N "" -C "rackware-rmm-server"
   ```
   {: codeblock}

   Parameters:

   -t rsa: Key type (rsa, ed25519, ecdsa, dsa)
   -b 4096: Key size in bits
   -f ~/.ssh/id_rsa: Output file path
   -N "": Empty passphrase
   -C "comment": Comment (usually email)

1. Record the public key `cat ~/.ssh/id_rsa.pub`:

   ```bash
   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDXA2L3ppb3YQxYQYC2eZon0I7J3Xm/zFMb5PuDHcxKTyFzZzyW6FZlZKM8iQMuO+4bkRVs1nLeqSl3bZR1ubWjaiYyj6cb4BY1/QVR+LNomzfW+hHfohb/MvDsEYNVkhJMHk1vdrZgIjQ5ADRjv2+1D/lUR1j8kHWdM+Bf0ilcuELdZI79iPV+T0umvYPyjtPO1zgkbMgfxh7qAjYkkrdDhbPAY3qvRyyuCrwzrFkmg7zc4QVj370uYDBekWpR9q+6GtGjt9f1Onf0k7CJPKQ1AOG/s+Yho58bK0ogxQJzf45jgfKH/GbOOKlwPzSVbOloajy+lQcepp4lqTKGt354fywwG+ePXFuID9iNJ3w9gsLu94xTjJ5GsB0FPq7V78fUFpKHnuCF5mRMR+mHevKb5oz0E6ANicoKr7yj15vUW0++AtM/qrEp0CbP3giIzQk0TBJsZrodr9vSIfHVPCd3z3/nAWnGhNftnZjw3/LcvUa5KXkuTvw9P5AA5EAXrd9SO77mhsH7+jVkooTI9nQxRnZkTEpPPQFZhZkXhSF/zOndQsX2gQd8/9d5Y+gq3ADo8ykrjgkYcO+J1MwZHlR54MKww+HbZqHDS3ePhphsUHYewldF/n82Ft9/dxUllVAKc9TfDMhyemdmh3yXJhrK8tlDoLB2/qzr4o2o40v9ew== rackware-rmm-server
   ```
   {: codeblock}

## Gather Information about the Source virtual machines
{: #virt-sol-vpc-migration-design-rmm-tutorial-source-vm-info}
{: step}

While RMM allows auto-provisioning, where the RMM server will create the target servers in the IBM Cloud VPC with the CPU, RAM, and disk that matches the source server specifications, we are manually creating the target virtual server instance in the IBM Cloud VPC in this tutorial. Collect the source virtual machine specifications and especially the IP address and assigned NAT IP address. For example:

| Hostname | OS | IP Address | NAT IP Address |
| --- | --- | --- | --- |
| VM1 | Ubuntu | 192.168.10.11 | 10.194.177.82 |
| VM2 | Windows | 192.168.10.12 | 10.194.177.83 |
{: caption="Source virtual machine specifications" caption-side="bottom"}

## Order the target virtual server instances
{: #virt-sol-vpc-migration-design-rmm-tutorial-step11}
{: step}

1. Follow the IBM Cloud instructions to create the required subnet in the VPC, the CIDR should match the CIDR of the source network.
1. Follow the IBM Cloud instructions to create the required security group in the VPC for the target virtual server instances. Ensure that SSH is allowed from the RMM to the target virtual server instances. For the Windows virtual server instance we will also need to include RDP so that we can connect to the virtual server instance to configure it ready for use by RMM
1. Follow the IBM Cloud instructions to create the required SSH keys in the VPC for the target virtual server instances. As we are provisioning the Ubuntu Target virtual server instance manually, we will need to upload the RMM public key we created earlier and include this when we provision the virtual server instance. This enables RMM to use password-less SSH. For the Windows virtual server instance, this key is used to encrypt the passowrd
1. With the information gathered in the previous step and using the IBM Cloud documentation order the IBM Cloud VPC virtual server instance servers and associated data volumes.

### Ubuntu Target virtual server instance
{: #virt-sol-vpc-migration-design-rmm-tutorial-step11-ubuntu}

As we are provisioning the Ubuntu Target virtual server instance manually, ensure you have uploaded the RMM public key we created earlier and include this when we provision the virtual server instance. This enables RMM to use password-less SSH.

### Windows Target virtual server instance
{: #virt-sol-vpc-migration-design-rmm-tutorial-step11-windows}

The RMM replicates and syncs Windows servers without requiring a windows password. This is referred to as SSH-Only. To properly configure SSH on Windows the RMM provides a small MSI that needs to be installed on the source virtual machine and target virtual server instance.
The RMM should ssh to the Windows servers as the `SYSTEM` user, this is the user most commonly used by RackWare customers, and should be considered the default choice

The RMM SSHD msi installer package can be run either from a web browser or directly on the Windows virtual server instance. After adding the route to the RMM server, we can use `https://<your-RMM-IP-or-FQDN-Address>/windows/RWSSHDService_x64.msi` e.g. `https://10.68.70.11/windows/RWSSHDService_x64.msi`.

For the Target virtual server instance, you will need to get the password after it has been provisioned to install the SSHD. Use the CLI command after logging in, `ibmcloud is in-init <target_VSI_name> --private-key @<private_key_location>` e.g. `ibmcloud is in-init rmm-source2-tgt --private-key @~/.ssh/sno3`

Allow SSH through the Windows Firewall by using the following in a PowerShell Window:

```ps1
New-NetFirewallRule `
  -DisplayName "Allow SSH (TCP 22)" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -Action Allow `
  -Profile Any
```
{: codeblock}

To install Rackware SSHD Service in the target virtual server instance:

1. Once the “Welcome to the Rackware SSHD Service Setup Wizard” page is displayed, press Next to begin the installation.
1. Read and accept the “License Agreement”, press Next, and the Installation Folder window will be shown.
1. Then press the ‘Next’ button. The SSHD Configuration window will be shown.
1. On the SSHD Configuration window:
1. As RMM will be accessing the Windows host as the SYSTEM user, then the username field should show SYSTEM.
1. Enter the RMM’s public SSH ke, which is the contents of the file /root/.ssh/id_rsa.pub on the RMM server.
1. Then press Next and the Confirm Installation screen will be shown.
1. Press Next to begin the installation.
1. After the installation completes, you will see the Installation Complete window.
1. Press the Close button.

Alternatively, if you want to use the command line, in a CMD window in the target virtual server instance:

```bash
# Create a directory
mkdir C:\<temp_dir>

# Download the RackWare SSHD MSI from the RMM server
curl.exe -k https://<your-RMM-IP-or-FQDN-Address>/windows/RWSSHDService_x64.msi -o C:\<temp_dir>\RWSSHDService_x64.msi

# Install the RackWare SSHD
msiexec.exe /i "C:\<temp_dir>\RWSSHDService_x64.msi" /passive /L*v C:\<temp_dir>\rwsshd.log TARGETDIR="C:\Program Files" SVCUSERNAME="" PASSWORD="" PORT="22" RMMSSHKEY="<RMM_PUBLIC_SSH KEY>"
```
{: codeblock}

For example:

```bash
# Create a directory
mkdir C:\Temp

# Download the RackWare SSHD MSI from the RMM server
curl.exe -k https://10.68.70.11/windows/RWSSHDService_x64.msi -o C:\Temp\RWSSHDService_x64.msi

# Install the RackWare SSHD
msiexec.exe /i "C:\Temp\RWSSHDService_x64.msi" /passive /L*v C:\Temp\rwsshd.log TARGETDIR="C:\Program Files" SVCUSERNAME="" PASSWORD="" PORT="22" RMMSSHKEY="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDXA2L3ppb3YQxYQYC2eZon0I7J3Xm/zFMb5PuDHcxKTyFzZzyW6FZlZKM8iQMuO+4bkRVs1nLeqSl3bZR1ubWjaiYyj6cb4BY1/QVR+LNomzfW+hHfohb/MvDsEYNVkhJMHk1vdrZgIjQ5ADRjv2+1D/lUR1j8kHWdM+Bf0ilcuELdZI79iPV+T0umvYPyjtPO1zgkbMgfxh7qAjYkkrdDhbPAY3qvRyyuCrwzrFkmg7zc4QVj370uYDBekWpR9q+6GtGjt9f1Onf0k7CJPKQ1AOG/s+Yho58bK0ogxQJzf45jgfKH/GbOOKlwPzSVbOloajy+lQcepp4lqTKGt354fywwG+ePXFuID9iNJ3w9gsLu94xTjJ5GsB0FPq7V78fUFpKHnuCF5mRMR+mHevKb5oz0E6ANicoKr7yj15vUW0++AtM/qrEp0CbP3giIzQk0TBJsZrodr9vSIfHVPCd3z3/nAWnGhNftnZjw3/LcvUa5KXkuTvw9P5AA5EAXrd9SO77mhsH7+jVkooTI9nQxRnZkTEpPPQFZhZkXhSF/zOndQsX2gQd8/9d5Y+gq3ADo8ykrjgkYcO+J1MwZHlR54MKww+HbZqHDS3ePhphsUHYewldF/n82Ft9/dxUllVAKc9TfDMhyemdmh3yXJhrK8tlDoLB2/qzr4o2o40v9ew== rackware-rmm-server"
```
{: codeblock}

After configuring all of the above steps for SSH-only, verify that the SSH public key authentication (aka “passwordless SSH”) is working by running the following command from the RMM `ssh SYSTEM@<ip_address>` e.g. `ssh SYSTEM@192.168.10.12`.

If you have any issues with the RMM's keys, they are located in the following location`C:\Program Files (x86)\Rackware-winutil\etc\authorized_keys`

## Prepare the Source virtual machines
{: #virt-sol-vpc-migration-design-rmm-tutorial-prepare-source-vm}
{: step}


### Ubuntu Source virtual machine
{: #virt-sol-vpc-migration-design-rmm-tutorial-prepare-source-vm-ubuntu}

Before a Linux virtual machine can be migrated, it must be configured such that the RMM server can communicate with it and that it can create snapshots from which the RMM will copy the data from the live server.

The RMM must be able to ssh without using a password to the source server:

- Add a route to the RMM server via the bridge server. In this tutorial:
    - RMM Server IP: 10.68.70.11
    - RMM bridge server inside IP: 192.168.10.254
- Create a user `rackware`
- Edit the `sudoers` file with the contents of the sudoers information contained in `opt/rackware/docs/sudo-config.txt` in the RMM server.
- Paste the contents of the RMM public key file created earlier into the source host’s authorized_keys file for the rackware user.

1. Login to your source host through SSH as user with root privileges or with full sudo privileges, and prepend ‘sudo’ to the commands on the source server.
1. Add a route to the RMM server via the bridge server, for example in Ubuntu `sudo nano /etc/netplan/50-cloud-init.yaml`:

   ```yaml
   ens192:
       dhcp4: false
       dhcp6: false
       addresses:
           - 192.168.10.11/24
       routes:
           - to: default
           via: 192.168.10.1
           - to: 10.68.70.11
           via: 192.168.10.254
   ```
   {: codeblock}

1. Create a user `rackware`:

   ```bash
   # Create the user rackware
   sudo useradd -m -s /bin/bash rackware
   ```
   {: codeblock}

1. Edit the `sudoers` file with the contents of the sudoers information contained in `opt/rackware/docs/sudo-config.txt` in the RMM server.

   ```bash
   # Edit sudoers file on the source host
   sudo visudo
   ```
   {: codeblock}

1. Copy sudo-config.txt from RMM to bottom of sudoers file on source host:

   ```bash
   # ---- BEGIN RACKWARE SUDOERS CONFIGURATION ----
   # Append this file to /etc/sudoers
   # Example:
   # cat sudo-config.txt >> /etc/sudoers

   User_Alias RW_MGMT_USERS = rackware

   Runas_Alias RW_MGMT_RUNAS_USER = root

   RW_MGMT_USERS ALL=(RW_MGMT_RUNAS_USER) NOPASSWD: ALL

   Defaults:RW_MGMT_USERS !requiretty
   # ---- END RACKWARE SUDOERS CONFIGURATION ----
   ```
   {: codeblock}

   ```bash
   # Create .ssh directory and authorized_key file for rackware user on source host
   sudo su - rackware
   mkdir -p /home/rackware/ .ssh
   touch /home/rackware/.ssh/authorized_keys
   chmod 700 ~/.ssh/
   chmod 600 ~/.ssh/authorized_keys
   vi /home/rackware/.ssh/authorized_keys
   ```
   {: codeblock}

1. Paste the contents of the RMM public key file created earlier into the source host’s authorized_keys file for the rackware user.
1. You may need to enable RSA keys in some Linux distros e.g. Ubuntu 22.04 by editing the SSH server config:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   {: codeblock}

1. Add this line at the end of the file:

   ```bash
   PubkeyAcceptedAlgorithms +ssh-rsa
   ```
   {: codeblock}

1. Save and exit (Ctrl+X, Y, Enter) and then restart SSH:

   ```bash
   sudo systemctl restart sshd
   ```
   {: codeblock}

1. Test connectivity between your RMM and the source virtual machine e.g. `ssh rackware@10.194.177.82`. You should connect without being prompted for a password.

### Windows Source virtual machine
{: #virt-sol-vpc-migration-design-rmm-tutorial-prepare-source-vm-windows}

The RMM replicates and syncs Windows servers without requiring a windows password. This is referred to as SSH-Only. To properly configure SSH on Windows the RMM provides a small MSI that needs to be installed on the source virtual machine and target virtual server instance.
The RMM should ssh to the Windows servers as the `SYSTEM` user, this is the user most commonly used by RackWare customers, and should be considered the default choice

The RMM SSHD msi installer package can be run either from a web browser or directly on the Windows virtual machine. After adding the route to the RMM server, we can use `https://<your-RMM-IP-or-FQDN-Address>/windows/RWSSHDService_x64.msi` e.g. `https://10.68.70.11/windows/RWSSHDService_x64.msi`.

On the source virtual machine you will need to add a static route, use the following syntax `route add destination_network MASK subnet_mask gateway_ip metric_cost` where:

- destination_network: The network you want to route.
- subnet_mask: The subnet mask for the destination network (optional, defaults to 255.255.255.0).
- gateway_ip: The IP address of the gateway.
- metric_cost: The cost metric for the route (optional).

For example, to route all traffic bound for the 10.0.0.0/8 subnet to a gateway at 192.168.10.254, use: `route add 10.0.0.0 MASK 255.0.0.0 192.168.10.254`

To make the route persistent (i.e., it remains after a reboot), add the -p option: `route -p add 10.0.0.0 MASK 255.0.0.0 192.168.10.254`

To remove a static route, use the following syntax: `route delete destination_network`. For example, to delete the route to the 10.0.0.0 network, use: `route delete 10.0.0.0`

Allow SSH through the Windows Firewall by using the following in a PowerShell Window:

```ps1
New-NetFirewallRule `
  -DisplayName "Allow SSH (TCP 22)" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -Action Allow `
  -Profile Any
```
{: codeblock}

To install Rackware SSHD Service in the source virtual machine:

1. Once the “Welcome to the Rackware SSHD Service Setup Wizard” page is displayed, press Next to begin the installation.
1. Read and accept the “License Agreement”, press Next, and the Installation Folder window will be shown.
1. Then press the ‘Next’ button. The SSHD Configuration window will be shown.
1. On the SSHD Configuration window:
1. As RMM will be accessing the Windows host as the SYSTEM user, then the username field should show SYSTEM.
1. Enter the RMM’s public SSH ke, which is the contents of the file /root/.ssh/id_rsa.pub on the RMM server.
1. Then press Next and the Confirm Installation screen will be shown.
1. Press Next to begin the installation.
1. After the installation completes, you will see the Installation Complete window.
1. Press the Close button.

Alternatively, if you want to use the command line, in a CMD window in the source virtual machine:

```bash
# Create a directory
mkdir C:\<temp_dir>

# Download the RackWare SSHD MSI from the RMM server
curl.exe -k https://<your-RMM-IP-or-FQDN-Address>/windows/RWSSHDService_x64.msi -o C:\<temp_dir>\RWSSHDService_x64.msi

# Install the RackWare SSHD
msiexec.exe /i "C:\<temp_dir>\RWSSHDService_x64.msi" /passive /L*v C:\<temp_dir>\rwsshd.log TARGETDIR="C:\Program Files" SVCUSERNAME="" PASSWORD="" PORT="22" RMMSSHKEY="<RMM_PUBLIC_SSH KEY>"
```
{: codeblock}

For example:

```bash
# Create a directory
mkdir C:\Temp

# Download the RackWare SSHD MSI from the RMM server
curl.exe -k https://10.68.70.11/windows/RWSSHDService_x64.msi -o C:\Temp\RWSSHDService_x64.msi

# Install the RackWare SSHD
msiexec.exe /i "C:\Temp\RWSSHDService_x64.msi" /passive /L*v C:\Temp\rwsshd.log TARGETDIR="C:\Program Files" SVCUSERNAME="" PASSWORD="" PORT="22" RMMSSHKEY="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDXA2L3ppb3YQxYQYC2eZon0I7J3Xm/zFMb5PuDHcxKTyFzZzyW6FZlZKM8iQMuO+4bkRVs1nLeqSl3bZR1ubWjaiYyj6cb4BY1/QVR+LNomzfW+hHfohb/MvDsEYNVkhJMHk1vdrZgIjQ5ADRjv2+1D/lUR1j8kHWdM+Bf0ilcuELdZI79iPV+T0umvYPyjtPO1zgkbMgfxh7qAjYkkrdDhbPAY3qvRyyuCrwzrFkmg7zc4QVj370uYDBekWpR9q+6GtGjt9f1Onf0k7CJPKQ1AOG/s+Yho58bK0ogxQJzf45jgfKH/GbOOKlwPzSVbOloajy+lQcepp4lqTKGt354fywwG+ePXFuID9iNJ3w9gsLu94xTjJ5GsB0FPq7V78fUFpKHnuCF5mRMR+mHevKb5oz0E6ANicoKr7yj15vUW0++AtM/qrEp0CbP3giIzQk0TBJsZrodr9vSIfHVPCd3z3/nAWnGhNftnZjw3/LcvUa5KXkuTvw9P5AA5EAXrd9SO77mhsH7+jVkooTI9nQxRnZkTEpPPQFZhZkXhSF/zOndQsX2gQd8/9d5Y+gq3ADo8ykrjgkYcO+J1MwZHlR54MKww+HbZqHDS3ePhphsUHYewldF/n82Ft9/dxUllVAKc9TfDMhyemdmh3yXJhrK8tlDoLB2/qzr4o2o40v9ew== rackware-rmm-server"
```
{: codeblock}

After configuring all of the above steps for SSH-only, verify that the SSH public key authentication (aka “passwordless SSH”) is working by running the following command from the RMM `ssh SYSTEM@<ip_address>` e.g. `ssh SYSTEM@10.194.177.83`.

If you have any issues with the RMM's keys, they are located in the following location`C:\Program Files (x86)\Rackware-winutil\etc\authorized_keys`

## Use the RMM GUI to configure the migrations and perform the initial migration
{: #virt-sol-vpc-migration-design-rmm-tutorial-initial-migration}
{: step}

RackWare RMM follows a phased approach that minimizes downtime through delta synchronization:

1. Initial Replication - Non-disruptive to the origin system; production servers continue running
1. Verification Phase - Test and verify applications in the target environment
1. Delta Sync(s) - Multiple delta syncs can be performed to keep target up-to-date with only changed files
1. Final Sync/Cutover - The actual cutover with minimal downtime. It is recommended that any applications are quiesced, while performing the final delta sync

   Once the source virtual machines has been prepared and the RMM can ssh to them, and the information for the target virtual server instances has been obtained, the virtual machine's workload can be migrated from the source virtual machines to the target virtual server instances using the RMM GUI.

   You can migrate servers one by one or run multiple, simultaneous migrations. If you are running multiple, simultaneous migrations, then download the CSV template from the RMM server and populate the appropriate fields. In this tutorial we will not be using this method.

1. To bring up the GUI, point a web browser at the Floating IP address of the RMM server.
1. Use `admin` as the Username. `rackware` is the default Password. If you have not already changed the password for the `admin` user, do so by using the standard Linux ‘passwd’ command. Press the Login button. The RMM home page will then be shown.
1. This step is only required, if you are using the auto-provisioning feature. Navigate to Configuration, Environments and click Add Environment.
   1. In the form add the following values and then click Add:
      - Name
      - Environment
      - Region
      - API Key

### Creating a wave and replication for the Ubuntu virtual machine
{: #virt-sol-vpc-migration-design-rmm-tutorial-wave-replication-ubuntu}

A wave contains a single host or multiple hosts that will be migrated. For this tutorial, you need to create one wave, provide information about the host in the wave, and then start the wave.

1. Via the RMM Server's UI, create a Wave (e.g. Wave1) using the following information:

   - Source:
       - Target Type: Existing System
       - DNS name / IP Address: 10.194.177.82
       - Friendly Name: rmm-source-1-src
       - OS: Linux
       - Username: rackware
   - Target:
       - Sync Type: Direct Sync
       - Hostname: rmm-source-1
       - DNS name / IP Address: 192.168.10.11
       - Friendly Name: rmm-source-1-tgt

1. Start the replication.

### Creating a wave and replication for the Windows virtual machine
{: #virt-sol-vpc-migration-design-rmm-tutorial-wave-replication-windows}

A wave contains a single host or multiple hosts that will be migrated. For this tutorial, you need to create a second wave, provide information about the host in the wave, and then start the wave.

1. Via the RMM Server's UI, create a Wave (e.g. Wave2) using the following information:

   - Source:
       - Target Type: Existing System
       - DNS name / IP Address: 10.194.177.83
       - Friendly Name: rmm-source-2-src
       - OS: Windows
       - Username: SYSTEM
   - Target:
       - Sync Type: Direct Sync
       - Hostname: rmm-source-2
       - DNS name / IP Address: 192.168.10.12
       - Friendly Name: rmm-source-2-tgt
       - Username: SYSTEM

1. Start the replication.

## Verification
{: #virt-sol-vpc-migration-design-rmm-tutorial-verification}
{: step}

The replications should complete. Once completed, you can connect to the target virtual server instances and carryout any testing as needed. The source virtual machines are still powered on and functioning. The target virtual server instances are isolated and can be tested without disrupting any production services.

## Perform delta syncs
{: #virt-sol-vpc-migration-design-rmm-tutorial-delta-sync}
{: step}

Delta syncs from the source environment to the IBM Cloud VPC virtual server instances capture changes to the source virtual machines since the initial replication was performed.

## Cut-over
{: #virt-sol-vpc-migration-design-rmm-tutorial-cutover}
{: step}

In this step we quiesce the applications and perform a cut-over/final delta sync from the source virtual machines to the IBM Cloud VPC virtual server instances.

1. Quiesce the applications on the source virtual machines. This step minimizes I/O during the final sync and ensures data consistency:
      - Stop applications/databases on the source virtual machines
      - Log out other users from the system
      - For systems with databases, ensure applications flush outstanding I/O to disk
1. Execute the final delta sync:
      - Trigger the final sync operation through the RMM GUI
      - RMM takes LVM/VSS snapshots (on Linux/Windows respectively) to ensure point-in-time consistency
      - Only changed files since the last sync are transferred (delta sync). This drastically reduces the final sync time to minutes rather than hours
1. Verify Sync Completion:
      - Monitor the wave status in the RMM console
      - Ensure sync completes successfully with no errors
1. Shutdown source virtual machines
1. Network cutover. Redirect user traffic to the new environment in VPC. This step depends on how you have been connecting to your IBM Cloud VMware VCF-Autonmated instance but could include:
      - routing traffic to the VPC.
      - removing prefix filters from the transit gateway connection.
      - redirecting traffic from the VPN tunnels to the VPC.
1. Post-Cutover Validation:
      - Verify all applications are functioning correctly
      - Check data integrity
      - Monitor performance
      - Test user access

## Next steps
{: #virt-sol-vpc-migration-design-rmm-tutorial=step-next}
