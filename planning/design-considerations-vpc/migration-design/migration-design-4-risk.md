---

copyright:
  years: 2025, 2026
lastupdated: "2026-03-03"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Risk mitigation and rollback strategies
{: #virt-sol-vpc-migration-design-risk}


The following information covers common difficulties that you might encounter after a migration and offers tips to correct the issues.
{: shortdesc}

## Common migration risks
{: #virt-sol-vpc-migration-design-risk-common}

The following information covers common difficulties that you might encounter after a migration.

### Risk: Network connectivity failure

{: #virt-sol-vpc-migration-design-risk-common-network}

Symptom: After migration, the virtual server can't communicate with other systems.


Detection: Post-migration connectivity tests fail.

Fix:

- Access through VNC console to troubleshoot
- Verify VNI configuration (IP address, security groups)
- Check routing tables in VPC and Transit Gateway
- Review security group rules (use VPC flow logs to see dropped packets)

Rollback: Restart VMware virtual servers and update DNS and load balancers to point back to VMware.

Prevention:

- Test Transit Gateway connectivity thoroughly before migration
- Verify that the security group rules allow required traffic
- Test DNS resolution in the target VPC

### Risk: The application doesn't start
{: #virt-sol-vpc-migration-design-risk-common-application}

**Symptom**: Application service doesn't start after migration, or starts but doesn't function.

**Detection**: The service fails to start, or starts but fails health checks.

**Fix**:

- Check application logs for errors
- Verify configuration files
- Check environment variables
- Verify database connectivity
- Check for licensing issues

**Rollback**: Stop the application in VPC, restart in a VMware environment.

**Prevention**:

- Verify that all dependencies are migrated or accessible through Transit Gateway.
- Test application startup procedures that are in the pilot wave.
- Document application-specific configuration that you might need to adjust.

### Risk: Data corruption
{: #virt-sol-vpc-migration-design-risk-common-data}

**Symptom**: Corrupted data, file system errors, or application data inconsistencies.

**Detection**: File system check errors, application reports data errors, database doesn't start

**Fix**:

- Attempt a file system repair
- If repair fails, retry migration from the source virtual server
- Verify that the source virtual server was cleanly shut down

**Rollback**: Discard the corrupted VPC virtual server, restart the source virtual server, and investigate the root cause before you retry the migration.

**Prevention**:

- Cleanly shut down virtual servers before migration
- Verify transfers with checksums, where possible
- Use `blockdev --flushbufs` before detaching volumes

### Risk: Performance degradation
{: #virt-sol-vpc-migration-design-risk-common-performance}

**Symptom**: Application performs worse in VPC than in VMware.

**Detection**: Increased response times and decreased throughput

**Fix**:

- Check CPU, memory, disk I/O, network bandwidth metrics
- Verify that the storage profile has adequate IOPS
- Verify that the instance profile has adequate network bandwidth
- Check for application configuration issues
- Consider upgrading instance or storage profiles

**Rollback**: If critical, failback to VMware while you investigate performance.

**Prevention**:

- Baseline performance in VMware before migration
- Select appropriate instance and storage profiles that are based on baselines
- Enable pooled storage bandwidth allocation

## Rollback strategy design
{: #virt-sol-vpc-migration-design-risk-rollback}

Use the following criteria to determine whether you need to roll back.

- More than 20% of virtual servers in a wave fails to start
- Critical applications fail functional testing
- Corrupted data is found in migrated virtual servers
- Performance degradation greater than 50% from baseline with no quick fix
- Security group configuration errors expose sensitive services

Rollback decision authority:

- Define who can make the rollback decision
- Define escalation path if decision-makers disagree
- Timebox rollback decision

### Rollback procedures
{: #virt-sol-vpc-migration-design-risk-rollback-procedures}

The following section explains the rollback procedure phases.

#### Using the Phase 1 rollback procedure
{: #virt-sol-vpc-migration-design-risk-rollback-phase1}

Before the virtual servers are created during the migration, follow these steps to use the Phase 1 rollback procedure. This process takes ~30 minutes.

1. Stop the migration.
2. Discard worker virtual servers and volumes.
3. Restart VMware virtual servers.
4. Update status communications.

#### Using the Phase 2 rollback procedure
{: #virt-sol-vpc-migration-design-risk-rollback-phase2}

After virtual servers created during the migration, but before DNS cutover, follow these steps to use the Phase 2 rollback procedure. This process takes ~1 hour.

1. Stop and delete migrated virtual servers.
2. Restart VMware virtual servers.
3. Verify that VMware virtual servers are functional.
4. Update status communications.

#### Using the Phase 3 rollback procedure
{: #virt-sol-vpc-migration-design-risk-rollback-phase3}

After DNS cutover during the migration, data might change. Use the following steps to use the Phase 3 rollback procedure. Depending on data size, this process takes 2-4 hours.

1. Stop virtual servers, but don't delete them.
2. Update DNS and load balancers to point back to VMware.
3. Restart VMware virtual servers.
4. Data decision:
   - If no data was changed, proceed with the rollback.
   - If data changed in VPC, you must sync data back to VMware before you can roll back.
5. Sync data, if needed.
6. Verify that VMware virtual servers are functional.
7. After verification period, delete VPC virtual servers

### Source virtual server preservation
{: #virt-sol-vpc-migration-design-risk-rollback-preservation}

To help make sure that the source virtual servers are preserved, use the following information.

- Don't delete VMware virtual servers immediately after the migration.
- The retention period is 7-30 days, depending on your risk tolerance.
- Create VMware snapshots.
- Document snapshot locations and your retention policies.
