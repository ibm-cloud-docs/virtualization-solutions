---

copyright:
  years: 2025, {[CURRENT_YEAR]}
lastupdated: "2026-02-13"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Pre-migration design
{: #virt-sol-vpc-migration-design-premigration}

Before you begin a migration, you have several tasks that you need to complete.
{: shortdesc}

## Workload assessment and categorization
{: #virt-sol-vpc-migration-design-premigration-assessment}

Before you begin your migration, you need to understand what you have beyond an inventory of your environment. The following sections help you assess and categorize your migration environments.

### Discovery and dependencies
{: #virt-sol-vpc-migration-design-premigration-assessment-discovery}

To extract your VMware inventory, you can use extraction tools such as RVTools. For each virtual server, keep the following information in mind.

- Identify dependencies. What does this virtual server communicate with? Can the virtual server migrate independently?
- Network characteristics. Does the environment use a static IP or DHCP? Does it have multiple NICs? Does the environment have specific VLAN requirements?
- Storage layout. How many disks does the environment have? What is the current IOPS usage? What is the total capacity?
- Performance baseline. What CPUs, memory, network, and storage options are used?
- Application characteristics. Do you have databases? Does the environment host a web server? Can the environment support a [cold migration](https://blogs.vmware.com/cloud-foundation/2019/12/10/hot-and-cold-migrations-which-network-is-used/){: external}?
- Compliance requirements. Location of the data, type of encryption, audit logging.

Knowing this information helps your wave planning, method selection, and timeline estimation.

### Addressing re-IP
{: #virt-sol-vpc-migration-design-premigration-assessment-reip}

Because you can't extend subnets and that VPC reserves certain addresses, you need to consider the following choices during your migration planning phase.

- Do you need extra subnets? If you can't migrate subnet by subnet, you might need to reassign subnets with certain applications. Consider separating applications into their own subnets.
- Which virtual server do you need to re-IP? If the IPs use `.0`, `.1`, `.2`, `.3`, broadcast addresses, or if you need to consolidate IP ranges, you need to change IPs.
- What strategy do you need to use to update the DNS and load balancers? Even if you maintain IPs within the VPC, you might need to update the external DNS, load balancers, and or firewall rules to direct to the new environment.

Keep this information available during the migration process.

## Connectivity architecture
{: #virt-sol-vpc-migration-design-premigration-connectivity}

Your virtual server might need to talk to each other during migration. It is expected that you migrate one subnet at a time, but you might need multiple subnets if latency is a problem. See the following information about migration connectivity.

### IBM Cloud Transit Gateway
{: #virt-sol-vpc-migration-design-premigration-connectivity-tgw}

Transit Gateway is the primary mechanism to connect your VMware environment to your VPC. Think of it as a cloud router that connects different network domains.

From IBM Cloud Classic (VMware on Classic infrastructure)

- Transit Gateway connects directly to your Classic account
- Provides routing between Classic VLANs and VPC subnets

From VMware on NSX (Classic with NSX Overlay)

- Transit Gateway connects to your NSX edges through GRE tunnels
- You configure GRE endpoints on your NSX edges and in Transit Gateway
- Routing propagates between NSX segments and VPC subnets

From VCF as a Service (VCFaaS)

- Transit Gateway connects to your VCFaaS edge through GRE tunnels
- A similar pattern is followed to NSX, but managed through the VCFaaS portal

Design considerations

- Provision Transit Gateway early in your migration project
- Plan your routing carefully because overlapping IP ranges between VMware and VPC can cause problems
- Consider a hub-and-spoke topology if you need to connect multiple VPCs or Classic accounts
- Test connectivity thoroughly before your first migration wave

## Instance profile selection
{: #virt-sol-vpc-migration-design-premigration-instance}

Instance profiles combine CPU generation, vCPU count, memory, network bandwidth, and maximum network interfaces. These profiles are different from VMware where vCPUs and RAM are independently set.

### Understanding vCPU to pCPU ratios
{: #virt-sol-vpc-migration-design-premigration-instance-vcpu}

VMware uses oversubscription ratios of 4:1 or 8:1 for vCPU to pCore. VPC standard instance profiles guarantee a 1:1 ratio of virtual CPUs to hyperthreaded cores. This oversubscription means that an 8 vCPU instance in VPC has 8 dedicated hyperthreads (not 8 vCPUs that can potentially share fewer cores).

IBM Cloud offers burstable virtual servers that you can use for oversubscription ratios of 2:1, 4:1, and 10:1, with the ability to burst up to 2x the guaranteed allocation. For more information about burstable virtual servers, see [Burstable virtual servers](/docs/vpc?topic=vpc-burstable-virtual-servers).

### Network bandwidth allocation
{: #virt-sol-vpc-migration-design-premigration-instance-network}

Each instance profile specifies total network bandwidth. The bandwidth total is allocated in a 3:1 ratio between network traffic and storage I/O by default, but you can adjust this allocation after the servers are provisioned.

For more information about bandwidth about allocation, see [About bandwidth allocation for instance profiles](/docs/vpc?topic=vpc-bandwidth-allocation-profiles).

If your profile supports it, choose the pooled allocation option. Pooled allocation dynamically shares storage bandwidth (based on usage) across all volumes rather than dividing the bandwidth equally. Your boot volume still gets a guaranteed minimum.

## Storage profiles
{: #virt-sol-vpc-migration-design-premigration-storage}

VPC offers two generations of storage profiles.

### First-generation storage profiles
{: #virt-sol-vpc-migration-design-premigration-storage-gen1}

First-generation storage profiles are available in the following situations.

- General-purpose: Required for boot volumes, 3 IOPS per GB
- 5iops-tier: 5 IOPS per GB
- 10iops-tier: 10 IOPS per GB
- Custom: Specify exact IOPS, currently up to 48,000 per volume

First-generation storage profiles offer the following advantages.

- Snapshot consistency groups to automatically create snapshots of multiple volumes
- GPT and UEFI detection for boot volumes
- Available in all regions

Use first-generation profiles for boot volumes and for production workloads that require snapshot consistency groups.
{: tip}

### Second-generation (`sdp`) profiles
{: #virt-sol-vpc-migration-design-premigration-storage-gen2}

The `sdp` profile offers improved performance and granular IOPS control.

Keep the following limitations in mind:

- Snapshot consistency groups aren't supported
- `sdp` profiles might not detect GPT-formatted volumes, which can cause the system to boot to BIOS instead of UEFI
- Secure boot isn't supported

Because of these limitations, don't use `sdp` for boot volumes. `sdp` is best for secondary data volumes with high performance and can tolerate individual snapshots.

## Security groups and encryption
{: #virt-sol-vpc-migration-design-premigration-security}

IBM Cloud VPC offers security groups and encryption options.

### Security groups
{: #virt-sol-vpc-migration-design-premigration-sg}

If you use the VMware distributed firewall (DFW) or NSX micro-segmentation, you need to migrate these rules to VPC security groups.

VPC security group details:

- Security groups are stateful, which automatically allows return traffic.
- You can assign multiple security groups to a VNI while traffic is allowed.
- Rules can reference the security group as the source or destination that means that members of this group can talk to each other on these ports.
- No explicit deny rules exist. Security groups are allow-only.

Migration strategy:

- Document your current DFW or firewall rules
- Group virtual server by security posture (web tier, app tier, database tier)
- Create security groups that mirror these tiers
- Use group-to-group references where possible to avoid enumerating IPs
- Thoroughly test in a nonproduction migration first

### Encryption
{: #virt-sol-vpc-migration-design-premigration-security-encryption}

If you use vSAN encryption, encrypted VMDKs, or guest OS encryption, you need to migrate these options.

VPC encryption details:

- All volumes support encryption at rest by provider-managed or user-managed keys through Key Protect or HPCS.
- Confidential computing profiles offer extra security with Intel SGX and TDX and secure boot.
- Data in transit between virtual servers and storage is encrypted by the IBM Cloud infrastructure.

Migration considerations:

- For most workloads, IBM Cloud-managed encryption is sufficient and requires no migration-specific work.
- For compliance-driven encryption, use Key Protect or HPCS with user-managed keys.
- If you require secure boot, use a confidential computing profile and avoid `sdp` storage.

## Licensing options
{: #virt-sol-vpc-migration-design-premigration-license}

IBM Cloud offers different licensing options for operating systems.

- [IBM Cloud stock images](/docs/vpc?topic=vpc-getting-started-images-on-vpc-stock)
- [IBM Cloud catalog images](/docs/vpc?topic=vpc-custom-image-cloud-private-catalog)
- [BYOL (Bring Your Own License)](/docs/vpc?topic=vpc-byol-vpc-about)

Licensing considerations:

- BYOL requires custom images
- License portability rules vary by vendor—verify compliance
- IBM Cloud-provided licenses include OS support
