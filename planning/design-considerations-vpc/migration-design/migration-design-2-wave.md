---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-17"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migration wave and execution plan
{: #virt-sol-vpc-migration-design-wave}

Successfully migrating dozens or hundreds of virtual servers requires structured wave planning.
{: shortdesc}

## Dependency planning
{: #virt-sol-vpc-migration-design-wave-planning}

Before you start your migration waves, you need to map the following application dependencies:

Tier-based:

- Web tier → App tier
- App tier → Database tier
- Database tier → Shared storage and services

Cross-application:

- Authentication
- Monitoring
- Backup
- Log aggregation
- DNS and NTP

Tools for discovery:

- VMware vRealize Network Insight
- Application dependency-mapping tools
- Network flow analysis
- Manual documentation from application owners

For each virtual server, record the following information in your migration plan:

- Inbound dependencies
- Outbound dependencies
- Shared resources

## Test migration wave design
{: #virt-sol-vpc-migration-design-wave-pilot}

Your first migration wave is the test (pilot) wave. To implement a successful test wave, you must adhere to the following information:

Proper representation of your virtual servers:

- Single-disk Linux virtual server
- Multi-disk Linux virtual server
- Single-disk Windows virtual server
- Multi-disk Windows virtual server
- Application with dependencies (3-tier app)

Use "minimum risk" options:

- Implement the migration in a nonproduction environment. Or, use a production environment that has large maintenance windows.
- Know the rollback procedure of your applications.

Perform a complete migration:

- Full end-to-end test of your chosen methods
- Note migration timing versus estimated timing
- Discovery of issues that were not found in testing

Successful migration criteria:

- All virtual servers start successfully
- Applications function correctly
- Network connectivity verified
- Performance meets or exceeds baseline
- No data loss or corruption
- Documentation of migration is complete and accurate

## Wave structure guidelines
{: #virt-sol-vpc-migration-design-wave-principles}

Subnet-based grouping:

Remember, you can't stretch a subnet between VMware and VPC. Which means that you need to do the following actions:

- Group virtual servers by subnet
- Migrate entire subnets in a single wave or know that you need to re-IP some virtual servers
- Plan subnet to VPC subnet mapping early

Application stack grouping for multitier applications:

- Migrate the entire stack in one wave, if possible.
- If your migration is too large, migrate from the database first, then apps, and so on.
- Maintain connectivity between migrated and nonmigrated tiers through Transit Gateway.

Application stack grouping for dependency-aware sequencing:

- Migrate infrastructure services first (DNS, monitoring, backup).
- Migrate shared services before the applications that they need.
- Consider the effects of each wave and wt the impact if it fails.

Parallel migration capabilities:

[Method 3 Live network transfer (Recommended for Scale)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method3) excels here:

- Provision multiple worker virtual server instances
- Migrate multiple virtual servers simultaneously
- Limited by network bandwidth and worker virtual server instance resources
- Typical: 4-8 concurrent migrations per worker virtual server instance

### Test wave example structure 
{: #virt-sol-vpc-migration-example-wave-structure}

The following example shows a 50 virtual server migration.

Wave 0 (test): 5 virtual servers

- 1x Single-disk Linux (Method 1 test)
- 1x Multi-disk Linux (Method 2 test)
- 1x Single-disk Windows (Method 1 with sysprep)
- 1x Multi-disk Windows (Method 2 with virt-v2v)
- 1x 3-tier test app (Methods 2 and 3, full stack)

Wave 1 (infrastructure): 8 virtual servers

- DNS servers
- Monitoring servers
- Jump hosts or bastion servers
- Shared file servers that migrated to VPC file storage

Wave 2 (application A): 12 virtual servers

- Database tier (3 virtual servers)
- App tier (6 virtual servers)
- Web tier (3 virtual servers)
- Subnet `10.50.10.0/24` → VPC subnet `10.240.10.0/24`

Wave 3 (Application B): 10 virtual servers

- Combined database and app tier (4 virtual servers)
- Web tier (6 virtual servers)
- Subnet `10.50.20.0/24` → VPC subnet `10.240.20.0/24`

Wave 4 (Application C): 15 virtual servers

- Large multitier application
- Subnet `10.50.30.0/24` → VPC subnet `10.240.30.0/24`

## Cutover Window design
{: #virt-sol-vpc-migration-design-wave-cutover}

Each wave needs a well-defined cutover window.

Pre-cutover timeline (7 days to migration day):

- Finalize wave plan and runbook
- Confirm Transit Gateway connectivity
- Provision worker virtual server instances and target volumes
- Communicate cutover window to stakeholders
- Reduce DNS TTLs for services that are migrating
- Notify users of maintenance window

### Cutover Execution (T-0)
{: #virt-sol-vpc-migration-design-wave-cutover-exec}

The following information describes the cutover phases.

#### Phase 1: Suspend and migrate (Hours 0-4)
{: #virt-sol-vpc-migration-design-wave-cutover1}

1. Drain connections - remove from load balancers and wait for sessions to close.
1. Gracefully stop applications
1. Shut down virtual servers or start from live ISO for Method 3
1. Begin disk transfer
1. Monitor transfer progress

#### Phase 2: Transform and provision (Hours 4-6)
{: #virt-sol-vpc-migration-design-wave-cutover2}

1. Run virt-v2v, if needed, to inject drivers
1. Verify disk transfers (fdisk, checksums)
1. Flush buffers and detach volumes from worker
1. Create the virtual server instances from migrated volumes
1. Start the virtual server instances

#### Phase 3: Validate and cutover (Hours 6-8)
{: #virt-sol-vpc-migration-design-wave-cutover3}

1. Start virtual server instances and access through the VNC console if needed
1. Verify network configuration and adjust if necessary
1. Start applications
1. Functional testing (application works, data accessible)
1. Add to load balancers and update DNS
1. Monitor application performance

#### Phase 4: Stabilize (Hours 8-12)
{: #virt-sol-vpc-migration-design-wave-cutover4}

1. Monitor for issues
1. Verify external connectivity
1. Check application logs for errors
1. Compare performance to baseline

Migration rollback decision points:

- After Phase 1: Easy rollback (restart the VMware virtual servers)
- After Phase 2: Medium difficulty (discard virtual server instances, restart VMware virtual servers, restore DNS)
- After Phase 3: Difficult (might have new data in VPC, requires data sync back to VMware)

Design recommendation: Define explicit go, no-go checkpoints. Examples:

- After Phase 2, if more than 20% of virtual server instances fail to start, implement a rollback.
- After Phase 3, if application functional tests fail, implement a rollback.
- After Phase 4, if performance is more than 30% less than the baseline, investigate but don't implement a rollback.

## Migration velocity estimation
{: #virt-sol-vpc-migration-design-wave-velocity}

Estimate the migration time per virtual server to plan realistic wave sizes and windows. The following timeline is achievable, but you need to check in a PoC before the actual migration.

Time components:

Export time estimates by using Methods 1-2:

- 100 GB virtual server: 20-30 minutes
- 500 GB virtual server: 2-3 hours
- Dependent on VMware storage performance

Transfer time estimates:

- Network: 100 GB = 20-25 minutes
- Network: 500 GB = 90-120 minutes
- Method 3 with compression: often 2 or 3 times faster because of compression

Transformation time estimates with virt-v2v:

- Linux: 5-10 minutes
- Windows: 10-20 minutes
- Dependent on worker virtual server instance performance

Provisioning time estimates:

- Virtual server instance creation: 5 minutes
- Boot and network configuration: 5-10 minutes

Example time estimates:

Small Linux virtual server (1 disk, 100 GB, Method 3):

- No export: 0 minutes
- Transfer with compression: 25 minutes
- Transformation: 5 minutes
- Provisioning: 10 minutes
- Total: 40 minutes

Large Windows virtual server time estimates by using Method 2 with 4 disks, 1 TB total:

- Export: 3 hours
- Transfer: 2 hours
- Transformation (virt-v2v): 20 minutes
- Provisioning: 10 minutes
- Total: 5.5 hours

Parallel migration efficiency by using both Method 3, 4 time estimate:

- 4x 100 GB virtual servers that are migrated in parallel
- 30 minutes each
- Total wave time: 35 minutes, which include startup and shutdown times

Compared with migrating 4x 100 GB virtual servers that you migrate serially:

Total wave time is 120 minutes

Parallelism gives you a 3 - 4 times improvement for this scenario.
