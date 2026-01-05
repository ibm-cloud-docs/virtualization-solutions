---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Post-Migration Validation and Optimization
{: #virt-sol-vpc-migration-design-post}

After each wave, validate success and identify optimization opportunities.

## Functional Validation Checklist
{: #virt-sol-vpc-migration-design-functional}

**For Each VSI**:
- [ ] Boots successfully
- [ ] All disks present and mounted
- [ ] Network connectivity (ping gateway, ping external host)
- [ ] Application starts without errors
- [ ] Application responds to requests
- [ ] Application logs show no critical errors

**For Each Application**:
- [ ] All tiers communicating correctly
- [ ] Database connections successful
- [ ] Load balancer health checks passing
- [ ] External access working (DNS, firewall rules)
- [ ] Authentication working (AD, LDAP, OAuth)
- [ ] Integrations working (APIs, file transfers, etc.)

**For the Environment**:
- [ ] Monitoring agents reporting
- [ ] Backup jobs configured and running
- [ ] Security group rules tested (allowed traffic works, blocked traffic blocked)
- [ ] Compliance checks passing (encryption, audit logging, etc.)

## Performance Validation
{: #virt-sol-vpc-migration-design-performance}

Compare post-migration metrics to pre-migration baselines:

**CPU Utilization**:
- Should be similar or lower (1:1 vCPU:pCPU vs. VMware's oversubscription)
- If higher, investigate application inefficiencies or incorrect instance profile

**Memory Utilization**:
- Should be identical (same allocated memory)
- If swapping, instance profile is too small

**Disk I/O**:
- IOPS should meet or exceed VMware performance
- If lower, check storage profile tier selection
- Verify pooled bandwidth allocation is enabled

**Network Throughput**:
- Should meet or exceed VMware performance
- If lower, check instance profile network bandwidth
- Verify storage bandwidth isn't consuming too much of the 3:1 ratio

**Application Response Time**:
- Should be similar to VMware
- If higher, investigate network latency (Transit Gateway overhead?) or storage I/O

## Right-Sizing Opportunities
{: #virt-sol-vpc-migration-design-rightsize}

Initial migrations often use "lift and shift" sizing (same vCPU and RAM as VMware). After migration, optimize:

**CPU Right-Sizing**:
- If average CPU < 20%, consider smaller instance profile
- If CPU bursts above 80% frequently, consider larger profile or burst profile
- Use VPC monitoring to track CPU utilization over 2-4 weeks

**Memory Right-Sizing**:
- If memory utilization < 50%, consider smaller profile
- If swapping or high memory pressure, increase profile
- Check application memory leaks (migration is a good opportunity)

**Storage Right-Sizing**:
- Review actual IOPS usage vs. provisioned tier
- Downgrade from 10iops-tier to 5iops-tier if actual usage is low
- Consider SDP for high-performance needs (once limitations are resolved)

**Network Bandwidth Right-Sizing**:
- Monitor actual network and storage bandwidth usage
- Adjust instance profile if consistently hitting limits
- Adjust storage:network ratio if storage is bottlenecked

## Optimization for Cloud-Native Patterns
{: #virt-sol-vpc-migration-design-optimization}

After migration stabilizes, consider cloud-native enhancements:

**Managed Services**:
- Replace self-managed databases with IBM Cloud Databases (Postgres, MySQL, MongoDB, etc.)
- Replace file servers with VPC file storage or Object Storage
- Replace load balancer VMs with VPC Application Load Balancer

**High Availability**:
- Deploy multi-zone (place VSIs across zones 1, 2, 3)
- Use VPC load balancers for automatic failover
- Implement auto-scaling for stateless tiers

**Backup and DR**:
- Leverage VPC snapshots for backup (consistency groups for multi-disk)
- Implement cross-region snapshot copies for DR
- Consider IBM Cloud Backup for file-level backup

**Monitoring and Observability**:
- Integrate with IBM Cloud Monitoring (Sysdig)
- Implement log aggregation with IBM Log Analysis (LogDNA)
- Use VPC Flow Logs for network traffic analysis
