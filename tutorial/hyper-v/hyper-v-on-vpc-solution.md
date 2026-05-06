---

copyright:
  years: 2026, 2026
lastupdated: "2026-05-06"

keywords: Hyper-V, VPC, VSI, cloud infrastructure, Virtual Machines

subcollection: virtualization-solutions

content-type: tutorial
services: Hyper-V, VPC VSI, VPC Bare-metal
account-plan: paid
completion-time: 60m

---

> ⚠️ **Attention:** Currently the Hyper-V on IBM VPC is a BYOL solution, i.e., customers need to bring their own Windows Server 2025 Datacenter license for bare-metral server OS activation.

# Hyper-V on IBM VPC
{: #virt-sol-hyperv-on-vpc-tutorial}

Hyper-V is Microsoft's enterprise-grade hypervisor technology built into Windows Server and Windows. Hyper-V on IBM VPC provides hardware virtualization capabilities that enable organizations to create, manage, and run virtual machines at scale on Windows Server and Windows hosted by bare-metal servers on top of IBM VPC isolated and secured private cloud.

## Key benefits
{: #virt-sol-hyperv-on-vpc-key-benefits}

- **Cost Efficiency & Savings**: Hyper-V is included with Windows Server/Windows 10/11 Pro+, eliminating extra licensing fees for the hypervisor itself. It reduces hardware, power, cooling, and data center space costs through server consolidation.
- **High Availability & Business Continuity**: Features like Fail-over Clustering and Hyper-V Replica ensure services remain available, providing redundancy and minimizing downtime during maintenance or failures.
- **Flexibility & Mobility**: Live Migration allows moving running VMs between hosts without user disruption, enabling efficient host maintenance and workload balancing.
- Security & Isolation: Provides secure environment isolation, including support for shielded virtual machines to protect sensitive data. It supports secure boot and TPM 2.0 for enhanced security, reducing risks from compromised external code.
- **Scalability & Performance**: As a type-1 hypervisor, Hyper-V can easily run on bare-metal servers hosted by {{site.data.keyword.vsi_is_full}} to deliver near-native performance and robust isolation for virtualized workloads leveraging IBM VPC's infrastructure capabilities. Hyper-V easily supports scaling resources (CPU, RAM) up or down for virtual machines.
- **Management & Automation**: Integrates with Windows-based tools, PowerShell, and system center for easy, automated provisioning of VMs and management of infrastructure.

## Planning for Hyper-V installation
{: #virt-sol-hyperv-on-vpc-planning-installation}

To plan for the installation of a typical Hyper-V cluster on IBM VPC infrastructure, it is highly suggested to refer to the  [Reference Architecture of Hyper-V on IBM Cloud](/docs/virtualization-solutions?topic=virt-sol-hyperv-on-vpc-architecture).

This tutorial requires the following prerequisites.

- An IBM VPC instance is ordered within the target IBM Cloud account. Make sure you are familiar with IBM Cloud VPC concepts and operations and have the administrator rights to order below infrastructure components within the VPC instance:
  - Bare-metal server
  - Virtual server instance
  - Subnet
  - Virtual network interface
  - Public gateway
  - Floating IP
  - Security group
- A management subnet is created within above created VPC instance with a public gateway attached to allow access to the Internet.
- As Hyper-V should be run on Windows 2025 Server due to the latest NVMe support, it is now a requirement of customer to BYOL for Windows 2025 Datacenter OS until IBM offers Datacenter version license for bare-metal servers.

## Create Security Group for Inbound/Outbound Network Traffic
{: #virt-sol-hyperv-on-vpc-security-group-creation}
{: step}

Use the following steps to create a security group with inbound and outbound rules used by the VSIs and Bare-metal servers:

1. From IBM Cloud console switch to the account where the VPC in the prerequisites resides.
2. From left navigation bar select "Infrastructure -> Network -> Security groups".
3. Click "Create" button and in the shown page choose the same region where the VPC resides, and make sure "Virtual private cloud" has the exact VPC instance selected.
4. For the inbound rule, make sure at least below 2 rules are included:

| Name | Protocol | Source type | Source | Destination type |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| &lt;Public network inbound rule name for icmp-tcp-udp protocol&gt; | ICMP_TCP_UDP | IP address | &lt;Public network gateway IP customer used to access the jump server&gt; | Any |
| &lt;Local(private) network inbound rule name for icmp-tcp-udp protocol&gt; | ICMP_TCP_UDP | CIDR block | 10.0.0.0/8 | Any |

{: caption="Inbound rules for Hyper-V servers on IBM VPC" caption-side="bottom"}

5. For the outbound rule, make sure at least below 1 rule are included:

| Name | Protocol | Source type | Source | Destination type |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| &lt;Outbound rule name for icmp-tcp-udp protocol&gt; | ICMP_TCP_UDP | Any | 0.0.0.0/0 | Any |

{: caption="Outbound rules for Hyper-V servers on IBM VPC" caption-side="bottom"}

## Deploy VSIs for Active Directory and Jump Usage
{: #virt-sol-hyperv-on-vpc-vsi-creation}
{: step}

Use the following steps to create 2 VSIs used as the Active Directory and jump server, as well as one extra VSI as AD server backup in the Hyper-V cluster on VPC environment:

1. From IBM Cloud console switch to the account where the VPC in the prerequisites resides.
2. From left navigation bar select "Infrastructure -> Compute -> Virtual server instances".
3. Click "Create" button and in the shown page:
   1. Choose "Windows Server 2025 Standard Edition (amd64)" as the image.
   2. it is highly recommended to select an "x3" profile for all the VSIs deployed here. It is also advised to install Windows Admin Center (WAC) and System Center Virtual Machine Manager (SCVMM) on the jump server for management of Hyper-V cluster hosts which require extra resources. Besides launching several remote desktop sessions from jump server, profile e.g. "cx3d-16x40" is more suitable to achieve reliable resizability for jump server.
   3. In "Storage" section, change the profile of the boot volume to SDP.
   4. In "Networking" section, make sure "Virtual private cloud" has the exact VPC instance selected and the "Network interface type" has "Virtual network interface" checked, and the VNI interface is on the management subnet mentioned in prerequisite list.
   5. Fill other fields with proper values and click "Create virtual server" button on the right panel.
4.For the jump server, after the VSI has been created, from left navigation bar select "Infrastructure -> Network -> Floating IPs", click "Reserve" button. In the shown dialog, make sure choose the region where the jump serverVSI is resided, provide a name, then click "Reserve" button.
5. From left navigation bar select "Infrastructure -> Compute -> Virtual server instances", and from the shown list of VSIs click on the name of the jump server just created. In the shown detail page, go to "Networking" tab, click on the 3 dots of "eth0" VNI, and choose "Edit Floating IPs". In the shown dialog, click "Attach" button, and then choose above newly created floating IP. Click "Attach" to associate the floating IP to the jump server VSI.
6. Add the newly associated public floating IP and a new inbound rule to the security group created in last section to allow perform remote desktop access to Windows server to be created on the bare-metal servers below.
7. Log on both the AD VSI and jump VSI, update and rename the IPv4 private network configuration from default DHCP to fixed IP per user's network environment(usually the IP address, gateway and DNS are required) with below PowerShell script. After configuration, reboot both VSIs. 

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

8. Follow the reference steps of [Install Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-){: external} to install and configure Active Directory Domain Services on the AD VSI.
9. Follow the same steps above to create and configure another VSI as second AD server which acts as backup (Refer to [Active Directory Forest Recovery - Back up a full server](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-backing-up-a-full-server){: external} for detailed guidance when setting up a full AD server backup.

** NOTE:
- To connect and login a VPC VSI with Windows OS, please refer to the guide [Connecting to Windows instances](/docs/vpc?topic=vpc-vsi_is_connecting_windows).
- The reason the VSIs along with below bare-metal servers are configured with static IP lies in that, although VPC DHCP provides the same IP to the bare-metal servers consistently, Active Directory and S2D along with other Microsoft products require fixed IP's instead. Converting to static IP is simple and avoids many issues and blockers when using DHCP in a Hyper-V environment for the infrastructure.

## Deploy Bare-metal Servers for Hyper-V Cluster
{: #virt-sol-hyperv-on-vpc-bm-creation}
{: step}

** NOTE: There is a known RAID NVMe Disk issue with current VPC bare-metal server: Users must create a customer image with "esxi-8" in the name in order to properly configure the NVMe disks in BIOS such that Windows S2D can utilize them. To create such a custom image:
- Deploy a VSI with Windows 2025 image. Please steps in last section.
- After creation, login the VSI, from a cmd.exe prompt run "c:\windows\system32\sysprep\sysprep.exe"
- In the shown window, choose the OOBE experience, click the “Generalize” button, and Shutdown (Note: do not reboot)
- After the VSI shutdowns, user should have a VSI option to create image - make sure the image name includes "exsi-8". The custom image will be saved in the custom image list of current IBM cloud account.

Use the following steps to create bare-metal servers which will be used for installation of Hyper-V cluster.

1. From IBM Cloud console switch to the account where the VPC in the prerequisites resides.
2. From left navigation bar select "Infrastructure -> Compute -> Bare metal servers".
3. Click "Create" button and in the shown page:
   1. Choose the custom image created above as the image.
   2. Choose the bare-metal profile with 'd' in the name that meets user's business needs with proper CPU, RAM and NVMe storage. Please remember that S2D storage is network intensive so we recommend 100Gbe network capacity. User may choose diskless profiles here if they are separating out their storage nodes.
   3. In "Networking" section, make sure "Virtual private cloud" has the exact VPC instance selected and the "Network interface type" has "Virtual network interface" checked, and the VNI interface is on the management subnet mentioned in prerequisite list.
   4. Fill other fields with proper values and click "Create bare metal server" button on the right panel.
4. After the bare-metral server is created, login and open a "Windows PowerShell" prompt. Issue command `Get-PhysicalDisk | Select-Object FriendlyName, BusType, CanPool, CannotPoolReason | Format-Table -AutoSize`. Make sure the result is NVMe instead of RAID.
5. Upgrade the Windows license to "Windows Server 2025 Datacenter" referring to below PowerShell scripts:

```PowerShell
$nodes = "<Current bare-metal server's computer name>"
$key = "<Your Windows Server 2025 Datacenter license key>"

Invoke-Command -ComputerName $nodes -ScriptBlock {
    param($k)
    DISM /online /Set-Edition:ServerDatacenter `
         /ProductKey:$k `
         /AcceptEula
} -ArgumentList $key
```

   Run below command to make sure the license has been successfully upgraded:

```PowerShell
Invoke-Command -ComputerName $nodes -ScriptBlock {
    Get-ComputerInfo | Select-Object WindowsProductName
} | Select-Object PSComputerName, WindowsProductName | Format-Table -AutoSize
```

6. From left navigation bar select "Infrastructure -> Compute -> Subnets", create a new subnet acting as an additional PCI network for Hyper-V workload bare-metal servers. Make sure "Virtual private cloud" has the exact VPC instance selected and IP range specified based on users' current network environment. Attach a public gateway to the subnet (You need to create a public gateway first from "Infrastructure -> Compute -> Public gateways" if you haven't to associate a floating public IP to it).
7. From left navigation bar select "Infrastructure -> Compute -> Bare metal servers", click on the name of newly create bare-metal server, and from the detail page go to "Networking" tab and attach the PCI subnet created in last step.
8. Like what has been done to VSIs created above, configure the IPv4 network of the bare-metal server from DHCP to fixed IP for both the management as well as the workload subnet.
9. Repeat above steps for other bare-metal servers to be created.

** NOTE:
To connect and login a VPC bare-metal server with Windows OS, please refer to the guide [Connecting to a Windows bare metal server](/docs/vpc?topic=vpc-bare_metal_server_connecting_windows).

## Install Hyper-V and Hyper-V Cluster on Bare-metal Servers
{: #virt-sol-hyperv-on-vpc-hyperv-installation}
{: step}

Use the following steps to install Hyper-V for each bare-metal servers(step 1 to 5) and the fail-over cluster on top of the Hyper-V nodes:

1. Login to the bare-metal server installed in last section (you may achieve this through remote desktop connection from the jump VSI), and open a "Windows PowerShell" prompt window, and issue command `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart` to install and enable Hyper-V on current Windows platform. This command will auto restart the server.
2. After restart of the bare-metal server, login and issue PowerShell command `Get-WindowsFeature Hyper-V` to make sure the result shown has the Hyper-V turned on.
3. Download Windows Admin Center from https://aka.ms/WACDownload and install it to the bare-metal server.
4. Run below PowerShell script to install fail-over cluster capability to bare-metal server (if you specify the comma separated names of all the bare-metal servers installed, you may only need to run the command once without running it again on other servers):

```PowerShell
$nodes = "<Current bare-metal server's computer name>"

Invoke-Command -ComputerName $nodes -ScriptBlock {
    Install-WindowsFeature `
        -Name Failover-Clustering `
        -IncludeManagementTools `
        -Restart
}
```

5. Reboot the server and run below script to verify the fail-over cluster capability is equipped:

```PowerShell
Invoke-Command -ComputerName $nodes -ScriptBlock {
    Get-WindowsFeature -Name Failover-Clustering |
        Select-Object Name, InstallState
} | Select-Object PSComputerName, Name, InstallState | Format-Table -AutoSize

```

6. Run script `Get-NetAdapter` to get existing net adapter current bare-metal server has. The net adapter for the workload network should be used to create the virtual switch. For most of the cases, the Hyper-V hosted virtual machines need to access the Internet, so the external typed switch needs to be ceated. Run script `New-VMSwitch -Name <switch-name>  -NetAdapterName <netadapter-name>` to create the switch with the second parameter be the adapter you get from `Get-NetAdapter` output.

7. After finishing step 1-6 for each bare-metal server that will be a member of future workload cluster, run below script to run validation tests on fail-over cluster hardware and settings to ensure compatibility and stability before actual installation:

```PowerShell
Test-Cluster `
    -Node <Comma separated member bare-metal server's computer name> `
    -Include "Storage Spaces Direct","Inventory","Network","System Configuration"
```

8. Run below script to create the fail-over cluster on top of them (you may only need to run these script from one of the bare-metal server):

```PowerShell
New-Cluster `
    -Name "<Name of the Hyper-V cluster, e.g., HyperV-Cluster>" `
    -Node <Comma separated member bare-metal server's computer name> `
    -ManagementPointNetworkType Distributed `
    -NoStorage
```

9. When finish, use below script to enable highly available storage using directly attached drives on the created cluster:

```PowerShell
Enable-ClusterStorageSpacesDirect `
    -CimSession "<Name of the Hyper-V cluster>" `
    -SkipEligibilityChecks `
    -Confirm:$false
```

10. Create the Cluster Shared Volumes (CSVs) for the created cluster. e.g., below script will create 4x25TB volumes with one per node affinity:

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

11. Use below script to obtain and check the created storage pool details:

```PowerShell
Get-StoragePool -CimSession "<Name of the Hyper-V cluster>" `
>>     -FriendlyName "<Name of the storage pool to created" |
>>     Get-PhysicalDisk -CimSession "HyperV-Cluster" |
>>     Select-Object FriendlyName,
>>                   @{N="SizeGB";E={[math]::Round($_.Size/1GB)}},
>>                   Usage |
>>     Format-Table -AutoSize
```

12. Use below script to check fault domains of created Cluster Shared Volumes (how disks are distributed across cluster nodes):

```PowerShell
Get-StorageFaultDomain -CimSession "<Name of the Hyper-V cluster>" |
>>     Select-Object FriendlyName, FaultDomainType |
>>     Format-Table -AutoSize
```

## Create and Configure Virtual Machine Hosted by Hyper-V
{: #virt-sol-hyperv-on-vpc-vm-creation}
{: step}

Use the following steps to create and configure a virtual machine hosted by a Hyper-V host node created former sections:

1. Download and copy the guest OS image iso file to a path where the bare-metal server that the virtual machine is to be installed can access. It is highly suggest the iso file is copied to above created CSV volumes, such that when performing live migration, there will be no need to move the storage of the VM any more, and only the VM itself needs to be moved which could be achieved instantly.
2. Run `New-VM -Name <Name> -MemoryStartupBytes <Memory> -BootDevice <BootDevice> -VHDPath <VHDPath> -Path <Path> -Generation <Generation> -Switch <SwitchName>` to create the virtual machine, where <SwitchName> is the virtual switch created in last section. e.g., `New-VM -Name TestVM1 -MemoryStartupBytes 4GB -BootDevice VHD -NewVHDPath .\VMs\Test1.vhdx -Path .\VMData1 -NewVHDSizeBytes 20GB -Generation 2 -Switch Hyper-V-PoC-Switch`. To benefit from latest optimization from Hyper-V, usually the VM generation 2 is used.
3. Specify the guest OS iso file path by setting it to a PowerShell variable: `$ISOPath ="<Guest OS iso file path>"`.
4. Run script `Add-VMDvdDrive -VMName <VM Name> -Path $ISOPath` to attach the iso as the DVD driver for the VM.
5. Run below script to set the DVD driver as the first boot device:

```PowerShell
$DVDDrive = Get-VMDvdDrive -VMName <VM Name>
Set-VMFirmware -VMName <VM Name> -FirstBootDevice $DVDDrive
```

6. Start VM by running script `Start-VM -Name <VM Name>`
7. Open VM console by running VMConnect from Command Prompt `VMConnect.exe localhost <VM Name>`. You will see the OS is booted from the iso image (for a Windows iso, you perhaps need to push any key to really choose boot from iso). Follow the UI steps to install the guest OS to the VM.

** NOTE: It has been found some Linux iso by default cannot be booted from if the secure boot is enabled (by installation default the secure boot is enabled). This is because the default boot template used is “Microsoft Windows“. One of the options can be chosen:
- Run script `Set-VMFirmware -VMName <VM Name> -EnableSecureBoot Off` to disable secure boot before starting the VM if you want to keep using template “Microsoft Windows“.
- Run script `Set-VMFirmware -VMName <VM Name> -SecureBootTemplate "MicrosoftUEFICertificateAuthority"` to use template “Microsoft UEFI Certificate Authority” if secure boot is still needed.

8. After guest OS has been installed, follow below steps to configure the network from both VPC setting on IBM cloud console and within the Hyper-V VM (Repeat this process for all bare metal servers in the cluster for the first 3 steps):
   1. From IBM cloud console, Open the detail page of the bare-metal server where the VM is created. In the “Networking“ tab, for the already attached PCI subnet, edit details, and in the shown dialog, input an integer for the VLAN number (e.g., 100) and add to allowed VLAN IDs.
   2. From IBM Cloud console where your environment is deployed, select "Infrastructure" from left navigator, and click "Virtual Network Interface" under "Network" drawer. Create a VNI on above workload subnet with Interface Type as "VLAN" and VLAN ID to be the number specified in above step. Turn on "Allow interface to float". Specify the Primary IP to be one within the workload subnet IP range (suggest to start using IP with last part above 50).
   3. Open the detail page of the bare-metal server, in the "Networking" tab, attach above created VNI with type VLAN and ID to be one specified in step 1 associated to the PCI interface.
   4. Login the bare-metal server, run script `Set-VMNetworkAdapterVlan -VMName "<VM Name>" -Access -VlanId <Vlan Id specified in IBM console>` to allow the VM enable virtual LAN identification for the what is configured in IBM console.
   5. Back to the bare-metal server, enter the VM through VM console, and edit the guest OS's network configuration from DHCP to fixed IP for IPv4 configuration. Again, this usually include the IPV4 address, IP mask, default gateway as well as IPV4 DNS servers. After making the configuration change, reset the network and make it in effect. Now you may open a command prompt window for the guest OS and try to ping an external web site, or open a browser to verify if the Internet is being connected from the VM (Please follow the guess OS's self guide to do the IP configuration. There may be differences between a Window and a Linux OS, or even different brands of Linux OSs, but the general goal is to have the guest OS use fixed IP).

** NOTE: 
The VLAN ID should be the same and get shared for the same subnet, and different is it is used for different subnets.

## Perform Live Migration of Virtual Machine between Hyper-V Host without Fail-over Cluster
{: #virt-sol-hyperv-on-vpc-vm-live-migration}
{: step}

** Prerequisite: For Windows Server 2025, CredSSP is no longer supported for authentication during live migration. To facilitate successful moving of such OS machines, need to configure Kerberos constrained delegation. Please follow the guide [Use the Users and Computers snap-in to set up constrained delegation](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/set-up-hosts-for-live-migration-without-failover-clustering#use-the-users-and-computers-snap-in-to-set-up-constrained-delegation){: external} to set up the configuration.

Run below script to perform moving a running virtual machine through Hyper-V Manager:

```PowerShell
Move-VM <VM Name> <Destination Server Name> [-IncludeStorage -DestinationStoragePath <File Path where VM Storage is placed>]
```

** NOTE: 

- As mentioned in former section, it is highly suggested when creating of the VM with Hyper-V, the VM storage is supposed to be placed in a CSV volume, and if this is followed, the optional parameters `-IncludeStorage` and `-DestinationStoragePath` are no longer needed, and the moving of the VM is just VM itself. If not, you must specify both of them and the moving will include the storage migration from source to destination server also.
- As for the VNI configured and attached to the bare-metal server from IBM cloud console, just one of the source or destination bare-metal server needs to attach the VNI, and after VM migration, the VNI will float automatically to target host without further configuration changes, provided that the VM's VLAN is an allowed_vlan for the new host.

## Perform Planned and Unplanned Fail-over with Hyper-V Cluster
{: #virt-sol-hyperv-on-vpc-failover-with-cluster}
{: step}

Use the following steps to create a highly available VM and perform planned and unplanned fail-over through Hyper-V cluster:

1. Use the same PowerScript with step 2 of the section "Create and Configure Virtual Machine Hosted by Hyper-V" to create a normal VM first. Remember for a highly available VM, the storage must be put in the CSV volumes of the cluster.

2. Use the following PowerScript to add the created VM to the Hyper-V cluster to make it a cluster resource, thus become highly available within the fail-over cluster:

```PowerShell
Add-ClusterVirtualMachineRole -Name "<New Role Name>" -VirtualMachine "<VM Name>" -Cluster "<Hyper-V Cluster Name>"
```

3. Follow the similar steps in section "Create and Configure Virtual Machine Hosted by Hyper-V" to install the guest OS of the created VM and configure the network for it.

4. The planned fail-over test of the highly available VM is actually done by live moving it from one node of the Hyper-V cluster to another. To achieve this, run the following PwoerScript, and when done, remote login the destination host from the jump VSI and check from its Failover Cluster Manager if the VM is resided in current machine and has the status up and running. You may even open the VM using VMConnect to see if it is just in the state before it is moved from source machine.

```PowerShell
Move-ClusterVirtualMachineRole -Name "<VM Name>" –Node <Destination Server Name>
```

5. The Unplanned fail-over test of the highly available VM is done by bringing down current cluster node where the VM is resided, and allow the cluster to self decide and schedule the most preferred node to move the VM to without explicitly run the moving script. Run below PowerScript to bring down a Hyper-V cluster node:

```PowerShell
Stop-ClusterNode –Name <Server Name Owning the VM>
```

After this is completed, wait a moment, and from the Failover Cluster Manager of current server to check where the VM has been moved to. You may remote login that server from jump VSI and check if the VM is really there and open it by VMConnect to verify the guest OS status.

** NOTE: When performing unplanned fail-over test, the bring-down of one node will cause all of the cluster wide resources on current node being moved, so if you have other loads still running, to avoid unexpected impact you may move those resources away from current node first.

6. When unplanned fail-over test is completed, you may run below PwoerScript to bring the stopped node up again:

```PowerShell
Start-ClusterNode –Name <Server Name Originally Owning the VM>
```
