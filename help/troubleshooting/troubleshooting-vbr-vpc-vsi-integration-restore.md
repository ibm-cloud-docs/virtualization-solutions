---

copyright:
  years: 2026
lastupdated: "2026-10-16"

keywords: Veeam restore troubleshooting IBM Cloud, slow restore Veeam VBR IBM Cloud VPC, worker instance restore bottlenecks IBM Cloud, COS archive tier retrieval IBM Cloud, SOBR restore performance IBM Cloud, Veeam restore RTO IBM Cloud VPC, temporary worker VSI restore IBM Cloud, Veeam restore connectivity failures IBM Cloud, Veeam VBR restore IBM Cloud troubleshooting, network throughput restore IBM Cloud VPC

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Troubleshooting Veeam Backup & Replication restores on {{site.data.keyword.cloud_notm}} VPC
{: #troubleshooting-vbr-vpc-vsi-integration-restore}

Troubleshoot Veeam Backup & Replication (VBR) restore issues on {{site.data.keyword.vpc_full}} (VPC), including slow restore performance, connectivity failures, and worker server configuration.
{: shortdesc}

This information applies to both virtual server instances and virtual machines (VMs). In these topics, *target machine* refers to the virtual server instance (VSI) or virtual machine (VM).

To identify and resolve common restore issues, start by checking connectivity, repository access, and the status of the temporary worker target machine.

## Restore operations are slow
{: #troubleshooting-vbr-vpc-vsi-integration-restore-slow}

When you restore a target machine by using VBR, the restore operation takes significantly longer than expected, which affects your recovery time objective.

### What is happening?
{: #tsSymptoms-vbr-vpc-vsi-integration-restore-slow}

You might experience one or more of the following symptoms:

- The restore job shows slow progress in the VBR console, with transfer rates significantly below the expected throughput. For example, the restore job reports less than 10 MB/s when you restore from {{site.data.keyword.cos_full_notm}}.
- The estimated time remaining for the restore operation continues to increase or remains unusually high.
- Network utilization appears low during the restore process, which indicates that data does not transfer at optimal speeds.
- The restore process takes hours to complete for volumes that should restore in minutes, based on their size.

### Why is it happening?
{: #tsCauses-vbr-vpc-vsi-integration-restore-slow}

Several factors can contribute to slow restore performance:

Network connectivity issues
:   Poor network connectivity between the temporary worker target machine and the VBR server can significantly affect restore speeds. Causes include high latency, packet loss, or insufficient bandwidth.

Repository performance bottlenecks
:   Performance bottlenecks in the Scale-out Backup Repository (SOBR) cause slow restores when concurrent operations are high, storage backend capacity is limited, or the repository server has insufficient resources.

{{site.data.keyword.cos_short}} tier limitations
:   If backups are stored in the {{site.data.keyword.cos_full_notm}} archive or cold storage tiers, retrieval times are significantly slower than in standard storage tiers. You must restore data from archive storage before you can access it.

Insufficient worker target machine resources
:   Restore operations fail or run slowly when the temporary worker target machine has insufficient CPU, memory, or network bandwidth.

Firewall or security group restrictions
:   Overly restrictive firewall rules or security group configurations can limit network throughput or cause connection timeouts during the restore process.

Concurrent backup or restore operations
:   Multiple simultaneous backup or restore jobs that compete for the same resources can reduce the performance of individual operations.

Geographic distance
:   Large geographic distances between the VBR server, the {{site.data.keyword.cos_short}} bucket, and the target machine can introduce latency that affects restore performance.

### How do you fix it?
{: #tsResolve-vbr-vpc-vsi-integration-restore-slow}

Try the following solutions to improve restore performance:

1. Verify network connectivity and bandwidth:
   - Test network connectivity between the temporary worker target machine and the VBR server by using tools such as `ping` and `traceroute`.
   - Check network bandwidth utilization on both the worker target machine and the VBR server to identify bottlenecks.
   - Help ensure that the target machine profile provides adequate network bandwidth for your restore requirements.
   - Verify that no network throttling or quality of service policies are limiting traffic between components.

2. Check repository performance:
   - Monitor the SOBR performance metrics in the VBR console to identify any performance degradation.
   - Verify that the repository server has sufficient CPU, memory, and disk I/O capacity to handle the restore operation.
   - Check for concurrent operations that might be competing for repository resources.
   - If you use local disk storage, help ensure that the disk subsystem is not experiencing high latency or I/O bottlenecks.

3. Optimize {{site.data.keyword.cos_short}} configuration:
   - Verify that backups are stored in the appropriate {{site.data.keyword.cos_short}} storage tier. The standard tier provides the fastest retrieval times.
   - If backups are in cold storage, consider copying data to the standard tier before you attempt the restore operation to improve retrieval speed.
   - If backups are in archive storage, you must initiate a restore request to move data to the standard tier before you can access the data for restore.
   - Check the {{site.data.keyword.cos_short}} service status and performance metrics for any ongoing issues.

4. Increase worker target machine resources:
   - Upgrade the temporary worker target machine to a higher profile with more CPU cores, memory, and network bandwidth.
   - Consider using compute-optimized or network-optimized profiles for better restore performance.
   - Monitor resource utilization during the restore to identify specific bottlenecks (CPU, memory, or network).

5. Review and adjust security configurations:
   - Verify that security group rules allow traffic between the VBR server and worker target machines on the required ports.
   - Check firewall rules on both the VBR server and worker target machines to help ensure that they are not limiting throughput.
   - Review any network access control lists that might be affecting traffic flow.
   - Temporarily disable Windows&reg; Firewall on Windows worker target machines to test whether it is causing performance issues. Re-enable it after you test.

6. Manage concurrent operations:
   - Limit the number of concurrent restore jobs to reduce resource contention.
   - Schedule large restore operations during off-peak hours when backup jobs are not running.
   - Use backup window settings to prevent backup and restore operations from overlapping.

7. Optimize VBR configuration:
   - In the VBR console, adjust the number of parallel processing tasks for restore operations.
   - Enable network acceleration features if they are available in your VBR version.
   - Verify that the Veeam agent on the worker target machine is up to date.
   - Check VBR logs for any warnings or errors that might indicate configuration issues.

8. Consider regional placement:
   - Help ensure that the VBR server and the {{site.data.keyword.cos_short}} bucket are in the same {{site.data.keyword.cloud_notm}} region as your target machines to minimize latency.
   - If cross-region restores are necessary, expect longer restore times and plan accordingly.

If the issue persists after trying these solutions, contact [{{site.data.keyword.cloud_notm}} Support](/docs/account?topic=account-gettinghelp) for further assistance. Provide details about your configuration, restore job logs, and the troubleshooting steps you have already attempted.

## File-level recovery fails
{: #troubleshooting-vbr-vpc-vsi-integration-flr}

When you perform file-level recovery (FLR) by using VBR, the restore operation times out or fails immediately.

### What is happening?
{: #tsSymptoms-flr}

You might experience one or more of the following symptoms:

- The VBR server cannot connect to the target machine.
- The VBR server cannot log in to the target machine.
- The VBR server cannot install the Veeam Transport Service on the target machine.

### Why is it happening?
{: #tsCauses-flr}

FLR can fail for one or more of the following reasons:

- {{site.data.keyword.vpc_short}} networking restrictions
- Target machine networking restrictions
- Incorrect time and date settings on the target machine
- Invalid target machine credentials that you provide during the FLR operation

### How do you fix it?
{: #tsResolve-flr}

Try the following solutions:

1. If Internet Control Message Protocol (ICMP) traffic is allowed on the target machine, verify that the VBR server can reach it by using `ping`.
2. If the Secure Shell (SSH) port is open on the target machine, verify that the VBR server can log in to it by using `ssh`.
3. Verify that there is no time drift between the VBR server and the target machine.
   - If the target machine has internet access, verify that the Network Time Protocol (NTP) server settings are correct and that time sync is successful.
   - If the target machine has no internet access, manually adjust the time and date settings.
