---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-19"

keywords: Migration, post migration, after migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Post-migration validation and optimization
{: #virt-sol-vpc-migration-design-post}

After each migration wave, validate success and identify optimization opportunities. 
{: shortdesc}

## Functional validation checklist
{: #virt-sol-vpc-migration-design-functional}

After a migration wave, verify the following items for each virtual server instance.

- [ ] Starts successfully
- [ ] All disks are present and mounted
- [ ] Network connectivity
- [ ] Application starts without errors
- [ ] Application responds to requests
- [ ] Application logs show no critical errors

For each application, check the following items.

- [ ] All tiers communicate correctly
- [ ] Database connections successful
- [ ] Passed load balancer health checks
- [ ] Confirm external access
- [ ] Authentication working
- [ ] Integrations working

For the environment, check the following items.

- [ ] Monitoring agents are reporting
- [ ] Backup jobs are configured and running
- [ ] Security group rules tested
- [ ] Compliance checks passed

## Performance validation
{: #virt-sol-vpc-migration-design-performance}

Use the following information to help you compare post-migration metrics to premigration baselines.

- CPU usage needs to be similar or less than the baseline. If CPU performance is higher, investigate for application inefficiencies or for incorrect instance profile.
- Memory usage needs to be identical. If memory swap occurs, the instance profile is insufficient.
- Disk IOPS needs to meet or exceed VMware performance. If IOPS is less than expected, check storage profile tier selection and verify that pooled bandwidth allocation is enabled.
- Network throughput needs to meet or exceed VMware performance. If throughput is less than expected, check instance profile network bandwidth and verify that storage bandwidth isn't using too much of the 3:1 ratio.
- Application response time needs to be similar to VMware. If the response time is higher, investigate network latency or storage I/O.

## Right-sizing opportunities
{: #virt-sol-vpc-migration-design-rightsize}

Initial migrations often use "lift and shift" sizing (same vCPU and RAM as VMware). After a migration, optimize the following resources:

CPU right-sizing:

- If average CPU is < 20%, consider a smaller instance profile
- If CPU bursts greater than 80% frequently, consider a larger profile or a burstable profile
- Use VPC monitoring to track CPU usage over 2-4 weeks

Memory right-sizing:

- If memory usage is < 50%, consider profiles with less memory
- If memory swap occurs or you have high memory pressure, consider profiles with more memory
- Check for application memory leaks

Storage right-sizing:

- Compare actual IOPS usage and provisioned tier usage
- Rollback from a `10iops-tier` to a `5iops-tier` profile if usage is less than expected
- Consider an `sdp` profile for high-performance needs

Network bandwidth right-sizing:

- Monitor actual network and storage bandwidth usage
- Adjust the instance profile if consistently reaching limits
- Adjust the storage to network ratio if storage throughput is slow

## Optimization for cloud-native patterns
{: #virt-sol-vpc-migration-design-optimization}

After migration stabilizes, consider the following cloud-native enhancements:

Managed services

- Replace self-managed databases with IBM Cloud databases
- Replace file servers with VPC file storage or object storage
- Replace load balancers with VPC Application Load Balancers

High availability

- Deploy in multi-zone regions
- Use load balancers for automatic failover
- Implement autoscale for stateless tiers

Backup and disaster recovery

- Use VPC snapshots for backup
- Implement cross-region snapshot copies for disaster recovery
- Consider IBM Cloud Backup for file-level backup

Monitoring and observability

- Integrate with IBM Cloud Monitoring
- Implement log aggregation with IBM Cloud Logs
- Use VPC Flow Logs for network traffic analysis
