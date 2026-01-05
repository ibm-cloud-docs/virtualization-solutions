---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Pre-Migration Design Decisions
{: #virt-sol-vpc-migration-design-premigration}

## Workload Assessment and Categorization
{: #virt-sol-vpc-migration-design-premigration-assessment}

Before you migrate anything, you need to understand what you have. This goes beyond a simple inventory.

### Discovery and Dependencies
{: #virt-sol-vpc-migration-design-premigration-assessment-discovery}

Use tools like RVTools to extract your VMware inventory. For each VM, document:

- **Dependencies**: What does this VM talk to? What talks to it? Can it be migrated independently?
- **Network characteristics**: Static IP or DHCP? Multiple NICs? Specific VLAN requirements?
- **Storage layout**: How many disks? Current IOPS usage? Total capacity?
- **Performance baseline**: Current CPU, memory, network, and storage utilization
- **Application characteristics**: Database? Web server? Can it tolerate a cold migration?
- **Compliance requirements**: Data residency, encryption, audit logging

This assessment drives your wave planning, method selection, and timeline estimation.

### The Re-IP Decision
{: #virt-sol-vpc-migration-design-premigration-assessment-reip}

Because you cannot stretch subnets and VPC reserves certain addresses, you need to decide early:

1. **What additional subnets do I need?** If you cannot migrate subnet-by-subnet, you may need to re-subnet certain applications. Consider separating applications into their own smaller subnets.

2. **Which VMs must be re-IPed?** If they use .0, .1, .2, .3, or broadcast addresses, or if you're consolidating IP ranges, you'll need to change IPs.

3. **What's the DNS/load balancer update strategy?** Even if you maintain IPs within the VPC, you'll likely need to update external DNS, load balancers, or firewall rules to point to the new environment.

Document this in a migration-specific document. You'll reference it constantly during execution.

## Connectivity Architecture: Bridging VMware and VPC
{: #virt-sol-vpc-migration-design-premigration-connectivity}

Your VMs may need to talk to each other during migration. It is expected that you will migrate a subnet at a time, but this may need to be multiple subnets if latency is a limiting factor.

### IBM Cloud Transit Gateway
{: #virt-sol-vpc-migration-design-premigration-connectivity-tgw}

Transit Gateway is the primary mechanism for connecting your VMware environment to VPC. Think of it as a cloud router that connects different network domains.

**From IBM Cloud Classic (VMware on Classic Infrastructure):**
- Transit Gateway connects directly to your Classic account
- Provides routing between Classic VLANs and VPC subnets

**From VMware on NSX (Classic with NSX Overlay):**
- Transit Gateway connects to your NSX edges via GRE tunnels
- You'll configure GRE endpoints on your NSX edges and in Transit Gateway
- Routing propagates between NSX segments and VPC subnets

**From VCF as a Service (VCFaaS):**
- Transit Gateway connects to your VCFaaS edge via GRE tunnels
- Similar pattern to NSX but managed through VCFaaS portal

**Design Considerations:**
- Provision Transit Gateway early in your migration project
- Plan your routing carefully as overlapping IP ranges between VMware and VPC will cause problems
- Consider a hub-and-spoke topology if connecting multiple VPCs or Classic accounts
- Test connectivity thoroughly before your first migration wave

## Instance Profile Selection
{: #virt-sol-vpc-migration-design-premigration-instance}

Instance profiles tie together CPU generation, vCPU count, memory, network bandwidth, and maximum network interfaces. This is quite different from VMware where you independently set vCPUs and RAM.

### Understanding vCPU:pCPU Ratios
{: #virt-sol-vpc-migration-design-premigration-instance-vcpu}

In VMware, you're accustomed to oversubscription ratios of 4:1 or 8:1 for vCPU to pCore. VPC's standard instance profiles guarantee a **1:1 ratio** of virtual CPU to hyperthreaded core. This means an 8-vCPU instance in VPC has 8 dedicated hyperthreads, not 8 vCPUs sharing potentially fewer cores.

IBM recently introduced **burst profiles** that allow for oversubscription ratios of 2:1, 4:1, and 10:1, with the ability to burst up to 2x the guaranteed allocation. This comes with improved pricing but with some constraints:

- Currently limited to "flex" CPU generation (IBM chooses the processor generation)
- In beta at the time of writing, not available to all accounts yet
- Performance characteristics may vary compared to dedicated 1:1 profiles

### Network Bandwidth Allocation
{: #virt-sol-vpc-migration-design-premigration-instance-network}

Each instance profile specifies total network bandwidth. This is allocated in a **3:1 ratio** between network traffic and storage I/O by default, and you can adjust this after provisioning.

Example: A profile with 32 Gbps total bandwidth allocates:
- 24 Gbps for VM network traffic
- 8 Gbps for storage I/O (across all attached volumes)

**Pooled vs. Fixed Storage Bandwidth**: If your profile supports it (check the documentation), choose **pooled allocation**. This allows your VSI to dynamically share storage bandwidth across all volumes rather than dividing it equally. Your boot volume still gets a guaranteed minimum.

## Storage Profile Selection
{: #virt-sol-vpc-migration-design-premigration-storage}

VPC offers two generations of storage profiles.

### First-Generation Profiles
{: #virt-sol-vpc-migration-design-premigration-storage-gen1}

- **general-purpose**: Required for boot volumes, 3 IOPS/GB
- **5iops-tier**: 5 IOPS/GB
- **10iops-tier**: 10 IOPS/GB  
- **custom**: Specify exact IOPS, currently up to 48,000 per volume

**Advantages**:
- Snapshot consistency groups (snapshot multiple volumes atomically)
- Reliable GPT/UEFI detection for boot volumes
- Available in all regions

**Design Recommendation**: Use first-generation profiles for boot volumes and for production workloads requiring snapshot consistency groups.

### Second-Generation: SDP Profiles
{: #virt-sol-vpc-migration-design-premigration-storage-gen2}

The `sdp` profile offers improved performance and granular IOPS control.

**Current Limitations**:
- Cannot snapshot as part of a consistency group (individual snapshots only)
- May not reliably detect GPT-formatted volumes, potentially booting to BIOS instead of UEFI
- Cannot use with Secure Boot
- Not available in all regions (notably Montreal, and likely not in new regions initially)

**Design Recommendation**: Avoid `sdp` for boot volumes. Consider for secondary data volumes where you need high performance and can tolerate individual snapshots. Monitor IBM's announcements—these limitations are expected to be addressed over time.

## Security and Compliance Mapping
{: #virt-sol-vpc-migration-design-premigration-security}

### Security Groups: Your New Distributed Firewall
{: #virt-sol-vpc-migration-design-premigration-sg}

If you're using VMware's distributed firewall (DFW) or NSX micro-segmentation, you'll need to translate these rules to VPC security groups.

**Key Differences**:
- Security groups are **stateful** (like DFW), allowing return traffic automatically
- You can assign **multiple security groups** to a VNI, and traffic is permitted if **any** group allows it
- Rules can reference **the group itself** as source/destination, meaning "members of this group can talk to each other on these ports"
- No explicit deny rules, security groups are allow-only (use lack of allow rule to deny)

**Migration Strategy**:
1. Document your current DFW or firewall rules
2. Group VMs by security posture (web tier, app tier, database tier, etc.)
3. Create security groups that mirror these tiers
4. Use group-to-group references where possible to avoid enumerating IPs
5. Test thoroughly in a non-production migration first

### Encryption
{: #virt-sol-vpc-migration-design-premigration-security-encryption}

**In VMware**: You might use vSAN encryption, encrypted VMDKs, or guest OS encryption.

**In VPC**: 
- All volumes support encryption at rest (provider-managed or customer-managed keys via Key Protect/HPCS)
- Confidential computing profiles offer additional security with Intel SGX/TDX and Secure Boot
- Data in transit between VSI and storage is encrypted by IBM's infrastructure

**Design Decision**: 
- For most workloads, IBM-managed encryption is sufficient and requires no migration-specific work
- For compliance-driven encryption, use Key Protect or HPCS with customer-managed keys
- If you require Secure Boot, use a confidential computing profile and avoid `sdp` storage

## Licensing Strategy
{: #virt-sol-vpc-migration-design-premigration-license}

IBM Cloud offers two licensing models for operating systems:

1. **IBM-provided licensing**: IBM includes the OS license in your hourly VSI cost. Available for RHEL, Windows Server, etc.

2. **BYOL (Bring Your Own License)**: You provide the license and create custom images marked as BYOL. Hourly costs are lower but you manage licensing compliance.

**VMware Migration Context**: If you have Windows Server licenses with Software Assurance or RHEL subscriptions with Cloud Access, you can leverage BYOL. Otherwise, IBM-provided licensing simplifies operations.

**Design Considerations**:
- BYOL requires custom images (covered in migration methods below)
- License portability rules vary by vendor—verify compliance
- IBM-provided licensing includes support for the OS from IBM
- Cost comparison: factor in both hourly costs and administrative overhead
