---

copyright:
  years: 2026
lastupdated: "2026-05-18"

keywords: Hyper-V, VPC, VSI, cloud infrastructure, Virtual Machines

subcollection: virtualization-solutions

content-type: tutorial
services: Hyper-V, VPC VSI, VPC Bare-metal
account-plan: paid
completion-time: 60m

---

{{site.data.keyword.attribute-definition-list}}

# Deploying and configuring a Hyper-V cluster on IBM Cloud VPC
{: #virt-sol-hyperv-on-vpc-tutorial}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="60m"}

Hyper-V is a Microsoft&reg; enterprise-grade hypervisor technology that is built into the Windows&reg; Server and Windows operating system. Hyper-V on {{site.data.keyword.vpc_short}} provides hardware virtualization capabilities that enable organizations to create, manage, and run virtual machines at scale on Windows-based environments that are hosted on bare metal servers within the {{site.data.keyword.vpc_short}} isolated and secured private cloud.
{: shortdesc}

## Overview
{: #virt-sol-hyperv-on-vpc-overview}

This tutorial guides you through deploying and configuring a Hyper-V cluster on {{site.data.keyword.vpc_short}} infrastructure. You learn how to set up the required network infrastructure, deploy virtual servers for Active Directory and management, create bare metal servers for the Hyper-V cluster, and configure highly available virtual machines. The tutorial also covers live migration and failover testing to help ensure business continuity.

Currently, Hyper-V on {{site.data.keyword.vpc_short}} follows the Bring Your Own License (BYOL) model. Customers need to bring their own Windows Server 2025 Datacenter license for activating the bare metal server operating system (OS).
{: attention}

## Key benefits
{: #virt-sol-hyperv-on-vpc-key-benefits}

Deploying Hyper-V on {{site.data.keyword.vpc_short}} provides several advantages for organizations that require enterprise-grade virtualization capabilities in the cloud. The following section highlights why this solution is effective for running enterprise virtualized workloads:

- Cost efficiency and savings: Hyper-V is included with Windows Server, Windows 10, or 11 Pro+, eliminating extra licensing fees for the hypervisor itself. It reduces hardware, power, cooling, and data center space costs through server consolidation.
- High availability and business continuity: Features such as failover clustering and Hyper-V replica help ensure that services remain available, providing redundancy and minimizing downtime during maintenance or failures.
- Flexibility and mobility: Live migration allows moving running virtual machines (VMs) between hosts without disrupting users, enabling efficient host maintenance and workload balancing.
- Security and isolation: Hyper-V provides secure environment isolation, including support for shielded virtual machines to protect sensitive data. It supports secure boot and TPM 2.0 for enhanced security, reducing risks from compromised external code.
- Scalability and performance: As a type-1 hypervisor, Hyper-V runs efficiently on bare metal servers that are provided by {{site.data.keyword.vpc_short}}, delivering near-native performance, and robust isolation for virtualized workloads by using its infrastructure capabilities. Hyper-V supports scaling resources (CPU, RAM) up or down for virtual machines.
- Management and automation: Hyper-V integrates with Windows-based tools, PowerShell, and System Center to simplify automated VM provisioning and infrastructure management.

## Planning for Hyper-V installation
{: #virt-sol-hyperv-on-vpc-planning-installation}

To plan for the installation of a typical Hyper-V cluster on {{site.data.keyword.vpc_short}} infrastructure, refer to the [Reference Architecture of Hyper-V on IBM Cloud](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-hyperv-on-vpc-architecture).

This tutorial requires the following prerequisites.

- Confirm that the {{site.data.keyword.vpc_short}} instance is available in the target {{site.data.keyword.cloud_notm}} account. Also, verify that you are familiar with {{site.data.keyword.vpc_short}} concepts and operations and have administrator privileges to provision the following infrastructure components within the {{site.data.keyword.vpc_short}} instance:
    - Bare-metal server
    - Virtual server instance
    - Subnet
    - Virtual network interface (VNI)
    - Public gateway
    - Floating IP
    - Security group
- Create a management subnet in the {{site.data.keyword.vpc_short}} instance and attach a public gateway to enable internet access.
- Because Hyper-V requires Windows Server 2025 due to the latest NVMe support, you must use the Bring Your Own License (BYOL) model for Windows 2025 Datacenter edition until IBM&reg; offers a Datacenter edition license for bare metal servers.

## Create a security group for inbound and outbound network traffic
{: #virt-sol-hyperv-on-vpc-security-group-creation}
{: step}

Complete the following steps to create a security group with inbound and outbound rules that the virtual servers and bare metal servers use:

1. From the {{site.data.keyword.cloud_notm}} console, switch to the account that contains the {{site.data.keyword.vpc_short}} instance that is specified in the prerequisites.
2. From the navigation menu, select **Infrastructure** > **Network** > **Security groups**.
3. Click **Create**. On the displayed page, choose the same region where the {{site.data.keyword.vpc_short}} instance is deployed, and ensure that **Virtual private cloud** has the exact {{site.data.keyword.vpc_short}} instance selected.
4. For the inbound rule, ensure that at least the following two rules are included:

    | Name | Protocol | Source type | Source | Destination type |
    | ---- | -------- | ----------- | ------ | ---------------- |
    | `<Public network inbound rule name for icmp-tcp-udp protocol`> | ICMP_TCP_UDP | IP address | `<Public network gateway IP customer used to access the jump server>` | Any |
    | `<Local(private) network inbound rule name for icmp-tcp-udp protocol>` | ICMP_TCP_UDP | CIDR block | 10.0.0.0/8 | Any |
    {: caption="Inbound rules for Hyper-V servers on IBM VPC" caption-side="bottom"}

5. For the outbound rule, ensure that at least the following rule is included:

    | Name | Protocol | Source type | Source | Destination type |
    | ---- | -------- | ----------- | ------ | ---------------- |
    | `<Outbound rule name for icmp-tcp-udp protocol>` | ICMP_TCP_UDP | Any | 0.0.0.0/0 | Any |
    {: caption="Outbound rules for Hyper-V servers on IBM VPC" caption-side="bottom"}

## Deploy virtual servers for Active Directory and jump server use
{: #virt-sol-hyperv-on-vpc-vsi-creation}
{: step}

Complete the following steps to create two virtual servers for Active Directory (AD) and jump server usage, along with an additional virtual server that serves as a backup AD server in the Hyper-V cluster environment on {{site.data.keyword.vpc_short}}:

1. From the {{site.data.keyword.cloud_notm}} console, switch to the account that contains the {{site.data.keyword.vpc_short}} instance that is specified in the prerequisites.
2. From the navigation menu, select **Infrastructure** > **Compute** > **Virtual server instances**.
3. Click **Create**. On the displayed page:
   1. Select **Windows Server 2025 Standard Edition (amd64)** as the image.
   2. Select an **x3** profile for all virtual servers that you deploy. Install Windows Admin Center (WAC) and System Center Virtual Machine Manager (SCVMM) on the jump server to manage Hyper-V cluster hosts that require additional resources. If multiple remote desktop sessions are expected on the jump server, use a profile such as `cx3d-16x40` to improve scalability and performance.
   3. In the **Storage** section, change the boot volume profile to SDP.
   4. In the **Networking** section, ensure that **Virtual private cloud** has the exact {{site.data.keyword.vpc_short}} instance that is selected and that the **Virtual network interface** is selected as the **Network interface type**. Verify that the VNI is attached to the management subnet specified in the prerequisites.
   5. Complete the remaining fields as required, and then click **Create virtual server** on the side panel.

4. After you create the jump server virtual server instance, from the navigation menu, select **Infrastructure** > **Network** > **Floating IPs**, and then click **Reserve**. In the displayed dialog, select the region where the jump server resides, enter a name, and then click **Reserve**.
5. From the navigation menu, go to **Infrastructure** > **Compute** > **Virtual server instances**.
6. From the list of displayed virtual servers, click the jump server that you created.
    1. On the detail page, go to the **Networking** tab, and then click the ellipsis menu or 3 dots for the `eth0` VNI, and choose **Edit Floating IPs**.
    2. In the displayed dialog, click **Attach**, and then select the newly created floating IP.
    3. Click **Attach** to associate the floating IP with the jump server virtual server.
7. Add the newly associated public floating IP and a new inbound rule to the security group that you created in the [previous section](#virt-sol-hyperv-on-vpc-security-group-creation). This configuration allows Remote Desktop access to the Windows server on the following bare metal servers.
8. Log in to both the AD virtual server and the jump virtual server. Update and rename the IPv4 private network configuration from default DHCP to a fixed IP according to your network environment (usually the IP address, gateway, and DNS are required) by using the following PowerShell command. After configuration, restart both virtual servers.

    To connect and log in to the {{site.data.keyword.vpc_short}} virtual server with Windows OS, see [Connecting to Windows instances](/docs/vpc?topic=vpc-vsi_is_connecting_windows).

    The virtual servers and the following bare metal servers are configured with static IP addresses. Although {{site.data.keyword.vpc_short}} DHCP consistently assigns the same IP addresses to the bare metal servers, Active Directory, S2D, and other Microsoft products require fixed IP addresses. Converting to a static IP address is straightforward and helps avoid issues and blockers when DHCP is used in a Hyper-V environment for the infrastructure.
    {: note}

    ```PowerShell
      # Get the network adapter index
      Get-NetAdapter
      # Generate the New Network Configuration
      New-NetIPAddress -InterfaceIndex <Network Adapter Index Obtained from Above> -IPAddress <Fixed IP Address> -PrefixLength <IP Address Prefix Length> -DefaultGateway <Default Gateway of the Fixed IP Address>
      # Set primary and secondary DNS
      Set-DnsClientServerAddress -InterfaceIndex <Network Adapter Index Obtained from Above> -ServerAddresses ("<DNS IP Address 1>","<DNS IP Address 2")
      # Rename the network adapter
      Rename-NetAdapter -Name "Current Adapter Name" -NewName "Desired AdapterName"
      # Verify the changes are applied successfully
      Get-NetIPConfiguration -InterfaceIndex <Network Adapter Index Obtained from Above>
    ```

9. Follow the instructions in [Install Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-){: external} to install and configure Active Directory Domain Services on the AD virtual server.
10. Repeat the same preceding steps to create and configure a second AD virtual server for backup purposes. For detailed guidance on setting up a full AD server backup, see [Active Directory Forest Recovery - Back up a full server](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-backing-up-a-full-server){: external}.

## Deploy bare metal servers for a Hyper-V cluster
{: #virt-sol-hyperv-on-vpc-bm-creation}
{: step}

A known RAID NVMe disk issue exists with the current {{site.data.keyword.vpc_short}} bare metal servers. You must create a custom image with `esxi-8` in the name to properly configure the NVMe disks in the Basic Input/Output System (BIOS) for Windows S2D support. To create the custom image, complete the following steps:

1. Deploy a virtual server by using a Windows 2025 image. Follow the steps in the [previous section](#virt-sol-hyperv-on-vpc-vsi-creation).
2. After you create the virtual server, log in to it. From the `cmd.exe` command prompt, run `c:\windows\system32\sysprep\sysprep.exe`.
3. In the displayed window, select the OOBE experience, click **Generalize**, and then click **Shutdown** (do not restart).

After the virtual server shuts down, use the virtual server option to create an image. Confirm that the image name includes `esxi-8`. The image is saved in the custom image list for the current {{site.data.keyword.cloud_notm}} account.
{: note}

Complete the following steps to create bare metal servers for the Hyper-V cluster installation.

1. From the {{site.data.keyword.cloud_notm}} console, switch to the account where the prerequisite {{site.data.keyword.vpc_short}} resides.
2. From the navigation menu, go to **Infrastructure** > **Compute** > **Bare metal servers**.
3. Click **Create**. On the displayed page:
   1. Select the custom image that you created earlier.
   2. Select a bare metal profile with `d` in the name that meets your business requirements for CPU, RAM, and NVMe storage. S2D storage is network-intensive, so `100 GB` network capacity is recommended. You can choose diskless profiles if your storage nodes are deployed separately.
   3. In the **Networking** section, help ensure that the correct VPC instance is selected for **Virtual private cloud** and that the **Network interface type** is set to **Virtual network interface**. Verify that the VNI interface is on the management subnet that is mentioned in the prerequisite list.
   4. Fill other fields with proper values, and then click **Create bare metal server** on the side panel.
4. After you create the bare metal server, log in and open a Windows PowerShell prompt. Run command `Get-PhysicalDisk | Select-Object FriendlyName, BusType, CanPool, CannotPoolReason | Format-Table -AutoSize`. Verify that the result is NVMe instead of RAID.
5. Upgrade the Windows license to Windows Server 2025 Datacenter by using the following PowerShell commands:

     ```PowerShell
        $nodes = "<Current bare metal server's computer name>"
        # if $nodes contains multiple nodes here, it should be in the value format @( "<computer 1 name>", "<computer 2 name>", ... "<computer n name>" )
        $key = "<Your Windows Server 2025 Datacenter license key>"

        Invoke-Command -ComputerName $nodes -ScriptBlock {
        param($k)
        DISM /online /Set-Edition:ServerDatacenter `
              /ProductKey:$k `
              /AcceptEula
        } -ArgumentList $key
    ```

     Run the following command to ensure that you successfully upgraded the license:

     ```PowerShell
        Invoke-Command -ComputerName $nodes -ScriptBlock {
        Get-ComputerInfo | Select-Object WindowsProductName
           } | Select-Object PSComputerName, WindowsProductName | Format-Table -AutoSize
     ```

6. From the navigation menu, go to **Infrastructure** > **Compute** > **Subnets**.
    1. Create a new subnet to serve as an additional PCI network for Hyper-V workload bare metal servers.
    2. Verify that the correct VPC is selected for the **Virtual private cloud** and that the IP range is configured according to your network environment.
    3. Attach a public gateway to the subnet.
    4. If you don't need to associate a floating public IP address with the subnet, first create a public gateway from **Infrastructure** > **Compute** > **Public gateways**.
7. From the navigation menu, go to **Infrastructure** > **Compute** > **Bare metal servers**, and then click the newly created bare metal server.
8. From the detail page, go to the **Networking** tab and attach the PCI subnet that you created in the previous step.
9. Configure the IPv4 network settings of the bare metal server from DHCP to static IP addresses for both the management and the workload subnets, similar to configuration used for the virtual servers in the [previous section](#virt-sol-hyperv-on-vpc-vsi-creation).
10. Repeat the preceding steps for each additional bare metal server that you create.

To connect and log in to a Windows bare metal server in {{site.data.keyword.vpc_short}}, see [Connecting to a Windows bare metal server](/docs/vpc?topic=vpc-bare_metal_server_connecting_windows).
{: note}

## Install Hyper-V and a Hyper-V cluster on bare metal servers
{: #virt-sol-hyperv-on-vpc-hyperv-installation}
{: step}

Complete the following steps to install Hyper-V on each bare metal server (steps 1 - 5) and configure the failover cluster on the Hyper-V nodes:

1. Log in to the bare metal server that you created in the [previous section](#virt-sol-hyperv-on-vpc-bm-creation).
    1. You can connect through a Remote Desktop connection from the jump virtual server.
    2. Open a Windows PowerShell prompt window and run the command `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart` to install and enable Hyper-V on the current Windows platform. This command automatically restarts the server.
2. After the bare metal server restarts, log in and run the PowerShell command `Get-WindowsFeature Hyper-V` to verify that Hyper-V is enabled.
3. Download Windows Admin Center from [https://aka.ms/WACDownload](https://aka.ms/WACDownload){: external} and install it on the bare metal server.
4. Run the following PowerShell command to install failover cluster capability on the bare metal server. If you specify the comma-separated names of all the bare metal servers you configured, you need to run the command once only without running it again on other servers.

     ```PowerShell
        $nodes = "<Current bare metal server's computer name>"
        # if $nodes contains multiple nodes here, it should be in the value format @( "<computer 1 name>", "<computer 2 name>", ... "<computer n name>" )
        Invoke-Command -ComputerName $nodes -ScriptBlock {
        Install-WindowsFeature `
            -Name Failover-Clustering `
            -IncludeManagementTools `
            -Restart
             }
     ```

5. Restart the server and run the following command to verify that the failover cluster capability is available:

      ```PowerShell
         Invoke-Command -ComputerName $nodes -ScriptBlock {
         Get-WindowsFeature -Name Failover-Clustering |
         Select-Object Name, InstallState
        } | Select-Object PSComputerName, Name, InstallState | Format-Table -AutoSize

     ```

6. Run the command `Get-NetAdapter` to get the network adapter that is available on the current bare metal server. Use the workload network adapter to create the virtual switch. In most cases, the Hyper-V-hosted virtual machines need internet access, so you need to create the external type switch. Run the command `New-VMSwitch -Name <switch-name> -NetAdapterName <netadapter-name>` to create the switch. The second parameter is the adapter that is returned by the `Get-NetAdapter` command.

7. After you finish steps 1 - 6 for each bare metal server that becomes a member of the workload cluster, run validation tests on failover cluster hardware and settings to help ensure compatibility and stability before the actual installation:

      ```PowerShell
         Test-Cluster `
         -Node <Comma separated member bare metal server's computer name> `
         -Include "Storage Spaces Direct","Inventory","Network","System Configuration"
      ```

8. Run the following command to create the failover cluster from a single bare metal server.

      ```PowerShell
        New-Cluster `
           -Name "<Name of the Hyper-V cluster, e.g., HyperV-Cluster>" `
           -Node <Comma separated member bare metal server's computer name> `
           -ManagementPointNetworkType Distributed `
           -NoStorage
      ```

9. After the cluster is created, run the following command to enable highly available storage by using directly attached drives on the created cluster:

      ```PowerShell
         Enable-ClusterStorageSpacesDirect `
          -CimSession "<Name of the Hyper-V cluster>" `
          -SkipEligibilityChecks `
          -Confirm:$false
     ```

10. Create Cluster Shared Volumes (CSVs) for the created cluster. For example, the following command creates 4x25TB volumes with one per node affinity:

      ```PowerShell
         $vols = @("Vol01","Vol02","Vol03","Vol04")
         foreach ($vol in $vols) {
            New-Volume `
              -CimSession "<Name of the Hyper-V cluster>" `
              -StoragePoolFriendlyName "<Name of the storage pool to be created>" `
              -FriendlyName $vol `
              -FileSystem CSVFS_ReFS `
              -ResiliencySettingName Mirror `
              -Size 25TB
         }
      ```

11. Run the following command to obtain and check the created storage pool details:

       ```PowerShell
          Get-StoragePool -CimSession "<Name of the Hyper-V cluster>" `
                   -FriendlyName "<Name of the storage pool to be created>" |
                    Get-PhysicalDisk -CimSession "HyperV-Cluster" |
                    Select-Object FriendlyName,
                           @{N="SizeGB";E={[math]::Round($_.Size/1GB)}},
                           Usage |
                  Format-Table -AutoSize
       ```

12. Run the following command to check fault domains of created Cluster Shared Volumes (how disks are distributed across cluster nodes):

     ```PowerShell
       Get-StorageFaultDomain -CimSession "<Name of the Hyper-V cluster>" |
              Select-Object FriendlyName, FaultDomainType |
              Format-Table -AutoSize
     ```

## Create and configure a virtual machine hosted by Hyper-V
{: #virt-sol-hyperv-on-vpc-vm-creation}
{: step}

Complete the following steps to create and configure a virtual machine hosted on the Hyper-V host node that is created in the [previous sections](#virt-sol-hyperv-on-vpc-hyperv-installation):

1. Download and copy the guest OS image ISO file to a location that the bare metal server hosting the virtual machine can access. Copy the ISO file to the CSV volumes that you created previously. This set up helps ensure that during live migration, only the VM needs to be moved, without moving its storage.
2. Run `New-VM -Name <Name> -MemoryStartupBytes <Memory> -BootDevice <BootDevice> -VHDPath <VHDPath> -Path <Path> -Generation <Generation> -Switch <SwitchName>` to create the virtual machine, where `<SwitchName>` is the virtual switch that you created in the [previous section](#virt-sol-hyperv-on-vpc-hyperv-installation). For example, `New-VM -Name TestVM1 -MemoryStartupBytes 4GB -BootDevice VHD -NewVHDPath .\VMs\Test1.vhdx -Path .\VMData1 -NewVHDSizeBytes 20GB -Generation 2 -Switch Hyper-V-PoC-Switch`. To benefit from the latest optimizations from Hyper-V, use Generation 2 VMs.
3. Set the guest OS ISO file path in a PowerShell variable: `$ISOPath ="<Guest OS iso file path>"`.
4. Run the command `Add-VMDvdDrive -VMName <VM Name> -Path $ISOPath` to attach the ISO file as the VM's DVD drive.
5. Run the following command to set the DVD drive as the first boot device:

      ```PowerShell
         $DVDDrive = Get-VMDvdDrive -VMName <VM Name>
         Set-VMFirmware -VMName <VM Name> -FirstBootDevice $DVDDrive
      ```

6. Start the VM by running the command `Start-VM -Name <VM Name>`.
7. Open the VM console by running `VMConnect` from the command prompt: `VMConnect.exe localhost <VM Name>`. Verify that the OS boots from the ISO image. For a Windows ISO, you might need to press a key to boot from the ISO image. Follow the UI steps to install the guest OS on the VM.

   Some Linux ISOs cannot boot by default from when secure boot is enabled. By installation default, secure boot is enabled. This behavior occurs because the default boot template is `Microsoft Windows`. Choose one of the following options:

      - Run the command `Set-VMFirmware -VMName <VM Name> -EnableSecureBoot Off` to disable secure boot before you start the VM if you want to keep using the template `Microsoft Windows`.
      - Run the command `Set-VMFirmware -VMName <VM Name> -SecureBootTemplate "MicrosoftUEFICertificateAuthority"` to use the template `Microsoft UEFI certificate authority` if secure boot is still required.

8. After you install the guest OS, follow these steps to configure the network from both the {{site.data.keyword.vpc_short}} setting on the {{site.data.keyword.cloud_notm}} console and within the Hyper-V VM. Repeat this process for all bare metal servers in the cluster for the first three steps.
   1. From the {{site.data.keyword.cloud_notm}} console, open the detail page of the bare metal server where the VM is hosted. On the **Networking** tab, edit the details of the attached PCI subnet. In the displayed dialog, enter an integer for the virtual local area network (VLAN) number (for example, 100). Then, add the VLAN ID to the allowed VLAN IDs.
   2. From the {{site.data.keyword.cloud_notm}} console where you deploy your environment, select **Infrastructure** from the navigation pane, and then click **Virtual Network Interface** under the **Network** section. Create a VNI on the workload subnet with the following settings:
        1. **Interface Type**: `VLAN`.
        2. **VLAN ID:** VLAN number specified in the preceding step.
        3. Turn on **Allow the interface to float**.
        4. Specify a primary IP address to be one within the workload subnet IP range.
        5. Use an IP address whose last octet is greater than 50.
   3. Open the detail page of the bare metal server. On the **Networking** tab, attach the VNI that you created with type `VLAN` and the ID that you specified in step 1 associated with the PCI interface.
   4. Log in to the bare metal server and run the command `Set-VMNetworkAdapterVlan -VMName "<VM Name>" -Access -VlanId <Vlan Id specified in IBM console>` to enable VLAN identification in the VM based on the VLAN configuration in the {{site.data.keyword.cloud_notm}} console.
   5. Return to the bare metal server and access the VM through the VM console. Change the guest OS network configuration from DHCP to a static IPv4 configuration. This configuration typically includes the IPv4 address, IP mask, default gateway, and IPv4 DNS servers. After updating the configuration, restart the network connection for the changes to take effect. Open a command prompt window in the guest OS and ping an external website, or open a browser to confirm that the VM has internet access. Follow the guest OS documentation to complete the IP configuration. Configuration steps can vary between Windows and Linux OS, as well as among different Linux distributions. However, the overall goal is to configure the guest OS with a static IP address.

Use and share the same VLAN ID for the same subnet and different VLAN IDs for different subnets.
{: note}

## Perform live migration of virtual machine between Hyper-V hosts without failover clustering
{: #virt-sol-hyperv-on-vpc-vm-live-migration}
{: step}

Before performing live migration, review the following prerequisites:

- For Windows Server 2025, CredSSP is no longer supported for authentication during live migration. To support live migration for these systems, configure Kerberos constrained delegation.
- For set up configuration, see [Use the Users and Computers snap-in to set up constrained delegation](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/set-up-hosts-for-live-migration-without-failover-clustering#use-the-users-and-computers-snap-in-to-set-up-constrained-delegation){: external}.

Run the following command in Hyper-V Manager to move a running virtual machine:

```PowerShell
Move-VM <VM Name> <Destination Server Name> [-IncludeStorage -DestinationStoragePath <File Path where VM Storage is placed>]
```

As described in the [previous section](#virt-sol-hyperv-on-vpc-failover-with-cluster), place the VM storage in a CSV volume when you create the VM with Hyper-V. If you follow this approach, you do not need the optional parameters `-IncludeStorage` and `-DestinationStoragePath` because only the VM configuration is migrated. Otherwise, you must specify both parameters to migrate the VM storage migration from the source server to the destination server. For the VNI configured and attached to the bare metal server from the {{site.data.keyword.cloud_notm}} console, only one of the source or destination bare metal servers must have the attached VNI. After the VM migration, the VNI automatically floats to the target host without additional configuration changes if the VM's VLAN is listed as an `allowed_vlan` on the target host.
{: note}

## Perform planned and unplanned failover with a Hyper-V cluster
{: #virt-sol-hyperv-on-vpc-failover-with-cluster}
{: step}

Complete the following steps to create a highly available VM and perform planned and unplanned failover in the Hyper-V cluster:

1. Use the same PowerShell command from step 2 of the **Create and configure a virtual machine that is hosted by the Hyper-V** section to create a standard VM. To configure a highly available VM, place the VM storage in the cluster CSV volumes.

2. Run the following PowerShell commands to configure the VM to the Hyper-V cluster as a highly available cluster resource in the failover cluster:

   ```PowerShell
   Add-ClusterVirtualMachineRole -Name "<New Role Name>" -VirtualMachine "<VM Name>" -Cluster "<Hyper-V Cluster Name>"
   ```

3. Repeat all the steps from the **Create and configure a virtual machine that is hosted by Hyper-V** section to install the guest OS of the VM that you created and configure the network for it.

4. Perform a planned failover test by live migrating the highly available VM to another Hyper-V cluster node using the following PowerShell command. After the migration completes, remotely log in to the destination host from the jump virtual server. In Failover Cluster Manager, verify that the VM is running on the destination node and that its status is **Running**. You can also open the VM by using `VMConnect` to confirm that the VM state was preserved after migration.

   ```PowerShell
   Move-ClusterVirtualMachineRole -Name "<VM Name>" -Node <Destination Server Name>
   ```

5. Perform the unplanned failover test by shutting down the cluster node that currently hosts the VM. The cluster automatically selects another preferred node and moves the VM without requiring a manual migration command. Run the following PowerShell command to shut down a Hyper-V cluster node:

   ```PowerShell
   Stop-ClusterNode -Name <Server Name Owning the VM>
   ```

6. After the command in the step 5 completes:

    1. Wait a few moments for failover to finish.
    2. In the Failover Cluster Manager of the current server, verify the node to which the VM was migrated.
    3. Remotely log in to the destination server from the jump virtual server to verify that the VM is running.
    4. Use `VMConnect` to verify the guest OS status.

       During an unplanned failover test, stopping one node causes all cluster-wide resources on that node to migrate. If other workloads are still running, move those resources to another node first to avoid unexpected disruptions.
       {: note}

7. When the unplanned fail-over test completes, run the below PwoerScript to restart the stopped node:

   ```PowerShell
   Start-ClusterNode -Name <Server Name Originally Owning the VM>
   ```
