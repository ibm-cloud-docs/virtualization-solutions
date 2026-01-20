---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Risk Mitigation and Rollback Strategies
{: #virt-sol-vpc-migration-design-risk}

## Common Migration Risks
{: #virt-sol-vpc-migration-design-risk-common}

### Risk: Network Connectivity Failure
{: #virt-sol-vpc-migration-design-risk-common-network}

**Manifestation**: After migration, VSI cannot communicate with other systems.

**Prevention**:
- Test Transit Gateway connectivity thoroughly before migration
- Verify security group rules allow required traffic
- Test DNS resolution in target VPC

**Detection**: Post-migration connectivity tests fail (ping, application cannot connect to database, etc.)

**Mitigation**:
- Access via VNC console to troubleshoot
- Verify VNI configuration (IP address, security groups)
- Check routing tables in VPC and Transit Gateway
- Review security group rules (use VPC flow logs to see dropped packets)

**Rollback**: Restart VMware VMs, update DNS/load balancers to point back to VMware

### Risk: Application Fails to Start
{: #virt-sol-vpc-migration-design-risk-common-application}

**Manifestation**: Application service won't start after migration, or starts but doesn't function.

**Prevention**:
- Verify all dependencies are also migrated or accessible via Transit Gateway
- Test application startup procedures in pilot wave
- Document application-specific configuration that may need adjustment

**Detection**: Service fails to start, or starts but health checks fail

**Mitigation**:
- Check application logs for errors
- Verify configuration files (paths may have changed if volumes are mounted differently)
- Check environment variables
- Verify database connectivity
- Check for licensing issues (some apps are hardware-locked)

**Rollback**: Stop application in VPC, restart in VMware environment

### Risk: Data Corruption
{: #virt-sol-vpc-migration-design-risk-common-data}

**Manifestation**: Disk contents are corrupted, filesystem errors, application data inconsistencies.

**Prevention**:
- Cleanly shut down VMs before migration (don't force-off)
- Verify transfers with checksums where possible
- Use `blockdev --flushbufs` before detaching volumes

**Detection**: Filesystem check errors, application reports data errors, database won't start

**Mitigation**:
- Attempt filesystem repair (fsck, xfs_repair, chkdsk)
- If repair fails, retry migration from source VM
- Verify source VM was cleanly shut down

**Rollback**: Discard corrupted VPC VSI, restart source VM, investigate root cause before retry

### Risk: Performance Degradation
{: #virt-sol-vpc-migration-design-risk-common-performance}

**Manifestation**: Application performs worse in VPC than in VMware.

**Prevention**:
- Baseline performance in VMware before migration
- Select appropriate instance and storage profiles based on baselines
- Enable pooled storage bandwidth allocation

**Detection**: Response times increase, throughput decreases, users complain of slowness

**Mitigation**:
- Check CPU, memory, disk I/O, network bandwidth metrics
- Verify storage profile provides adequate IOPS
- Verify instance profile provides adequate network bandwidth
- Check for application configuration issues (connection pooling, caching, etc.)
- Consider upgrading instance or storage profiles

**Rollback**: If critical, failback to VMware while investigating

## Rollback Strategy Design
{: #virt-sol-vpc-migration-design-risk-rollback}

**Rollback Trigger Criteria**:

Define explicit criteria that trigger rollback:
- More than 20% of VSIs in a wave fail to boot
- Critical application fails functional testing
- Data corruption detected in migrated VMs
- Performance degradation > 50% from baseline with no quick fix
- Security group configuration error exposes sensitive services

**Rollback Decision Authority**:
- Define who can make rollback decision (migration lead, application owner, etc.)
- Define escalation path if decision-makers disagree
- Time-box rollback decision (e.g., "must decide within 2 hours of cutover")

**Rollback Procedures**:

**Phase 1 Rollback** (before VSIs are created):
1. Abort migration
2. Discard worker VSI and volumes
3. Restart VMware VMs
4. Update status communications
5. **Time required**: ~30 minutes

**Phase 2 Rollback** (after VSIs created, before DNS cutover):
1. Stop and delete VSIs
2. Restart VMware VMs
3. Verify VMware VMs are functional
4. Update status communications
5. **Time required**: ~1 hour

**Phase 3 Rollback** (after DNS cutover, data may have changed):
1. Stop VSIs (don't delete yet—they may have new data)
2. Update DNS/load balancers to point back to VMware
3. Restart VMware VMs
4. **Data decision**: 
   - If no data changed (read-only apps), proceed with rollback
   - If data changed in VPC (write operations occurred), must sync data back to VMware before full rollback
5. Sync data if needed (application-specific procedure)
6. Verify VMware VMs are functional
7. After verification period, delete VPC VSIs
8. **Time required**: ~2-4 hours (depending on data sync)

**Source VM Preservation**:
- Don't delete VMware VMs immediately after migration
- Retention period: 7-30 days depending on risk tolerance
- Create VMware snapshots as rollback points
- Document snapshot locations and retention policies
