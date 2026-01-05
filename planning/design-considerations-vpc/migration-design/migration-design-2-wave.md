---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Wave Planning and Execution Design
{: #virt-sol-vpc-migration-design-wave}

Successfully migrating dozens or hundreds of VMs requires structured wave planning.

## Dependency Mapping
{: #virt-sol-vpc-migration-design-wave-mapping}

Before creating waves, map application dependencies:

**Tier-Based Dependencies**:
- Web tier → App tier
- App tier → Database tier
- Database tier → Shared storage/services

**Cross-Application Dependencies**:
- Authentication (AD, LDAP)
- Monitoring (agents reporting to central servers)
- Backup (backup agents, backup targets)
- Log aggregation (syslog, Splunk forwarders)
- DNS and NTP

**Tools for Discovery**:
- VMware vRealize Network Insight (if available)
- Application dependency mapping tools (ServiceNow CMDB, etc.)
- Network flow analysis
- Manual documentation from application owners

**Document in Migration docs**: For each VM, record:
- Inbound dependencies (who talks to this VM)
- Outbound dependencies (who this VM talks to)
- Shared resources (storage, load balancers, etc.)

## Pilot Wave Design
{: #virt-sol-vpc-migration-design-wave-pilot}

Your first migration wave is the pilot. It should:

**Be Representative**: Include examples of:
- Single-disk Linux VM
- Multi-disk Linux VM  
- Single-disk Windows VM
- Multi-disk Windows VM
- Application with dependencies (3-tier app)

**Be Low-Risk**:
- Non-production if possible
- Or production with large maintenance windows
- Applications with understood rollback procedures

**Be Complete**:
- Full end-to-end test of your chosen method(s)
- Documentation of actual timing vs. estimates
- Discovery of gotchas not found in testing

**Success Criteria**:
- All VMs boot successfully
- Applications function correctly
- Network connectivity verified (inbound and outbound)
- Performance meets or exceeds baseline
- No data loss or corruption
- Documentation complete and accurate

## Wave Structure Principles
{: #virt-sol-vpc-migration-design-wave-principles}

**Subnet-Based Grouping**:

Remember, you cannot stretch a subnet between VMware and VPC. This means:
- Group VMs by subnet
- Migrate entire subnets in a single wave (or accept that some VMs will need re-IPing)
- Plan subnet-to-VPC-subnet mapping ahead of time

**Application Stack Grouping**:

For multi-tier applications:
- Migrate the entire stack in one wave if possible
- If too large, migrate from bottom up (database first, then app, then web)
- Maintain connectivity between migrated and non-migrated tiers via Transit Gateway

**Dependency-Aware Sequencing**:

- Migrate infrastructure services first (DNS, monitoring, backup)
- Migrate shared services before applications that depend on them
- Consider the "blast radius" of each wave—what's impacted if it fails?

**Parallel Migration Capabilities**:

Method 3 (network transfer) excels here:
- Provision multiple worker VSIs
- Migrate multiple VMs concurrently
- Limited by network bandwidth and worker VSI resources
- Typical: 4-8 concurrent migrations per worker VSI

**Example Wave Structure** (50-VM migration):

**Wave 0 (Pilot)**: 5 VMs
- 1x Single-disk Linux (Method 1 test)
- 1x Multi-disk Linux (Method 2 test)
- 1x Single-disk Windows (Method 1 with sysprep)
- 1x Multi-disk Windows (Method 2 with virt-v2v)
- 1x 3-tier test app (Methods 2/3, full stack)

**Wave 1 (Infrastructure)**: 8 VMs
- DNS servers
- Monitoring servers
- Jump hosts / bastion servers
- Shared file servers (migrated to VPC file storage)

**Wave 2 (Application A)**: 12 VMs
- Database tier (3 VMs)
- App tier (6 VMs)
- Web tier (3 VMs)
- Subnet 10.50.10.0/24 → VPC subnet 10.240.10.0/24

**Wave 3 (Application B)**: 10 VMs
- Combined database and app tier (4 VMs)
- Web tier (6 VMs)
- Subnet 10.50.20.0/24 → VPC subnet 10.240.20.0/24

**Wave 4 (Application C)**: 15 VMs
- Large multi-tier application
- Subnet 10.50.30.0/24 → VPC subnet 10.240.30.0/24

## Cutover Window Design
{: #virt-sol-vpc-migration-design-wave-cutover}

Each wave needs a well-defined cutover window.

**Pre-Cutover (T-7 days to T-1 day)**:
- Finalize wave plan and runbook
- Confirm Transit Gateway connectivity
- Provision worker VSIs and target volumes
- Communicate cutover window to stakeholders
- Reduce DNS TTLs for services being migrated
- Notify users of maintenance window

### Cutover Execution (T-0)
{: #virt-sol-vpc-migration-design-wave-cutover-exec}

#### Phase 1: Quiesce and Migrate (Hours 0-4)
{: #virt-sol-vpc-migration-design-wave-cutover1}

1. Drain connections (remove from load balancers, wait for sessions to close)
2. Stop applications gracefully
3. Shut down VMs (or boot from live ISO for Method 3)
4. Begin disk transfer
5. Monitor transfer progress

#### Phase 2: Transform and Provision (Hours 4-6)
{: #virt-sol-vpc-migration-design-wave-cutover2}

1. Run virt-v2v if needed for driver injection
2. Verify disk transfers (fdisk, checksums)
3. Flush buffers and detach volumes from worker
4. Create VSIs from migrated volumes
5. Start VSIs

#### Phase 3: Validate and Cutover (Hours 6-8)
{: #virt-sol-vpc-migration-design-wave-cutover3}

1. Boot VSIs, access via VNC console if needed
2. Verify network configuration, adjust if necessary
3. Start applications
4. Functional testing (application works, data accessible)
5. Add to load balancers / update DNS
6. Monitor application performance

#### Phase 4: Stabilize (Hours 8-12)
{: #virt-sol-vpc-migration-design-wave-cutover4}

1. Monitor for issues
2. Verify external connectivity (users can access)
3. Check application logs for errors
4. Compare performance to baseline

**Rollback Decision Points**:
- After Phase 1: Easy rollback (just restart VMware VMs)
- After Phase 2: Medium difficulty (discard VSIs, restart VMware VMs, restore DNS)
- After Phase 3: Difficult (may have new data in VPC, requires data sync back to VMware)

**Design Recommendation**: Define explicit go/no-go checkpoints. For example:
- "After Phase 2, if more than 20% of VSIs fail to boot, execute rollback"
- "After Phase 3, if application functional tests fail, execute rollback"
- "After Phase 4, if performance is more than 30% below baseline, investigate but don't rollback"

## Migration Velocity Estimation
{: #virt-sol-vpc-migration-design-wave-velocity}

Estimate time per VM to plan realistic wave sizes and windows. The following should be achievable but you should check in a PoC before the actual migration.

**Time Components**:

**Export (Methods 1-2)**:
- 100GB VM: ~20-30 minutes
- 500GB VM: ~2-3 hours
- Dependent on VMware storage performance

**Transfer**:
- Network: 100GB = ~20-25 minutes
- Network: 500GB = ~90-120 minutes
- Method 3 with compression: Often 2-3x faster due to compression

**Transformation (virt-v2v)**:
- Linux: ~5-10 minutes (minimal work)
- Windows: ~10-20 minutes (driver injection, registry updates)
- Dependent on worker VSI performance

**Provisioning**:
- VSI creation: ~5 minutes
- Boot and network configuration: ~5-10 minutes

**Example Estimates**:

**Small Linux VM (1 disk, 100GB, Method 3)**:
- No export: 0 minutes
- Transfer with compression: ~25 minutes
- Transformation: ~5 minutes
- Provisioning: ~10 minutes
- **Total: ~40 minutes**

**Large Windows VM (4 disks, 1TB total, Method 2)**:
- Export: ~3 hours
- Transfer: ~2 hours
- Transformation (virt-v2v): ~20 minutes
- Provisioning: ~10 minutes
- **Total: ~5.5 hours**

**Parallel Migration Efficiency** (Method 3, 4 concurrent):
- 4x 100GB VMs migrating in parallel
- ~30 minutes each
- **Total wave time: ~35 minutes** (including startup/shutdown overhead)

vs.

- 4x 100GB VMs migrating serially
- **Total wave time: ~120 minutes**

Parallelism gives you a 3 - 4x improvement for this scenario.
