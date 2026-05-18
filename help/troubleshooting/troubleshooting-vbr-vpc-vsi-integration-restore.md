---

copyright:
  years: 2026
lastupdated: "2026-05-18"

keywords: Veeam Backup & Replication, VBR, backup, recovery, IBM Cloud VPC, VSI, Cloud Object Storage, SOBR

subcollection: virtualization-solutions

content-type: troubleshooting
services: OpenShift Virtualization, VMware

---

{{site.data.keyword.attribute-definition-list}}

# Why Can't I Restore Veeam Backup & Replication Restoration on VPC Virtual Server Instances?
{: #troubleshooting-vbr-vpc-vsi-integration-restore}
{: toc-services="OpenShift Virtualization, VMware"}

The troubleshooting section helps you identify common restore issues and provide actions to resolve them.

Start by checking connectivity, repository access, and the status of the temporary worker virtual server instance.

## Restore process is slow
{: #troubleshooting-vbr-vpc-vsi-integration-restore-slow}
{: #troubleshoot}
{: #support}

When restoring virtual server instances using Veeam Backup & Replication, the restore operation takes significantly longer than expected, impacting your recovery time objectives (RTO).

### What is happening?
{: #tsSymptoms-vbr-vpc-vsi-integration-restore-slow}

You might experience one or more of the following symptoms:

- The restore job shows slow progress in the Veeam Backup & Replication console, with transfer rates significantly below expected throughput.
- The estimated time remaining for the restore operation continues to increase or remains unusually high.
- Network utilization appears low during the restore process, indicating that data is not being transferred at optimal speeds.
- The restore process takes hours to complete for volumes that should restore in minutes based on their size.
- Veeam reports low throughput rates (for example, less than 10 MB/s) when restoring from {{site.data.keyword.cos_full}}.

### Why is it happening?
{: #tsCauses-vbr-vpc-vsi-integration-restore-slow}

Several factors can contribute to slow restore performance:

- Network connectivity issues: Poor network connectivity between the temporary worker virtual server instance and the Veeam Backup & Replication server can significantly impact restore speeds. This includes high latency, packet loss, or insufficient bandwidth.
- Repository performance bottlenecks: The Scale-Out Backup Repository (SOBR) might be experiencing performance issues due to high concurrent operations, storage backend limitations, or insufficient resources on the repository server.
- {{site.data.keyword.cos_short}} tier limitations: If backups are stored in {{site.data.keyword.cos_full}} archive or cold storage tiers, retrieval times are significantly slower than standard storage tiers. Data must be restored from archive storage before it can be accessed.
- Insufficient worker virtual server instance resources: The temporary worker virtual server instance might not have adequate CPU, memory, or network bandwidth to handle the restore operation efficiently.
- Firewall or security group restrictions: Overly restrictive firewall rules or security group configurations can limit network throughput or cause connection timeouts during the restore process.
- Concurrent backup or restore operations: Multiple simultaneous backup or restore jobs competing for the same resources can reduce the performance of individual operations.
- Geographic distance: Large geographic distances between the Veeam server, {{site.data.keyword.cos_short}} bucket, and target virtual server instance can introduce latency that affects restore performance.

### How do you fix it?
{: #tsResolve-vbr-vpc-vsi-integration-restore-slow}

Try the following solutions to improve restore performance:

1. Verify network connectivity and bandwidth:
   - Test network connectivity between the temporary worker virtual server instance and the Veeam Backup & Replication server using tools like `ping` and `traceroute`.
   - Check network bandwidth utilization on both the worker virtual server instance and the Veeam server to identify bottlenecks.
   - Ensure that the virtual server instance profile provides adequate network bandwidth for your restore requirements. Consider upgrading to a higher profile if needed.
   - Verify that no network throttling or quality of service (QoS) policies are limiting traffic between components.

2. Check repository performance:
   - Monitor the SOBR performance metrics in the Veeam console to identify any performance degradation.
   - Verify that the repository server has sufficient CPU, memory, and disk I/O capacity to handle the restore operation.
   - Check for concurrent operations that might be competing for repository resources and consider scheduling restores during off-peak hours.
   - If using local disk storage, ensure that the disk subsystem is not experiencing high latency or I/O bottlenecks.

3. Optimize {{site.data.keyword.cos_short}} configuration:
   - Verify that backups are stored in the appropriate {{site.data.keyword.cos_short}} storage tier. Standard tier provides the fastest retrieval times.
   - If backups are in archive or cold storage, initiate a restore request to move data to standard tier before attempting the restore operation.
   - Ensure that the {{site.data.keyword.cos_short}} bucket is in the same region as your virtual server instances to minimize latency.
   - Check {{site.data.keyword.cos_short}} service status and performance metrics for any ongoing issues.

4. Increase worker virtual server instance resources:
   - Upgrade the temporary worker virtual server instance to a higher profile with more CPU cores, memory, and network bandwidth.
   - Consider using compute-optimized or network-optimized profiles for better restore performance.
   - Monitor resource utilization during the restore to identify specific bottlenecks (CPU, memory, or network).

5. Review and adjust security configurations:
   - Verify that security group rules allow unrestricted traffic between the Veeam server and worker virtual server instances on required ports.
   - Check firewall rules on both the Veeam server and worker virtual server instances to ensure they are not limiting throughput.
   - Review any network ACLs that might be affecting traffic flow.
   - Temporarily disable Windows Firewall on Windows worker instances to test if it's causing performance issues (re-enable after testing).

6. Manage concurrent operations:
   - Limit the number of concurrent restore jobs to reduce resource contention.
   - Schedule large restore operations during off-peak hours when backup jobs are not running.
   - Use Veeam's backup window settings to prevent backup and restore operations from overlapping.

7. Optimize Veeam configuration:
   - In the Veeam console, adjust the number of parallel processing tasks for restore operations.
   - Enable network acceleration features if available in your Veeam Backup & Replication version.
   - Verify that the Veeam agent on the worker virtual server instance is up to date.
   - Check Veeam logs for any warnings or errors that might indicate configuration issues.

8. Consider regional placement:
   - Ensure that the Veeam server, {{site.data.keyword.cos_short}} bucket, and target virtual server instances are in the same {{site.data.keyword.cloud_notm}} region to minimize latency.
   - If cross-region restores are necessary, expect longer restore times and plan accordingly.

If the issue persists after trying these solutions, contact [{{site.data.keyword.cloud_notm}} Support](/docs/get-support) for further assistance. Provide details about your configuration, restore job logs, and the troubleshooting steps you have already attempted.
