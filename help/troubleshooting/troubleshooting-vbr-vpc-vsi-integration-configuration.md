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

# Why Can't I Configure Veeam Backup & Replication Configuration on VPC Virtual Server Instances?
{: #troubleshooting-vbr-vpc-vsi-integration-configuration}

Use this topic to identify and resolve common issues when you configure Veeam Backup & Replication integration for {{site.data.keyword.vpc_full}} virtual server instances.
{: shortdesc}

If you encounter issues while configuring or running Veeam Backup & Replication in {{site.data.keyword.vpc_short}}, review the following common problems and solutions. Each troubleshooting scenario includes symptoms, potential causes, and step-by-step resolutions.

## Cannot connect to IBM Cloud Object Storage repository
{: #veeam-vbr-vpc-troubleshoot-cos}

Use this section to troubleshoot connectivity issues between Veeam Backup & Replication and your {{site.data.keyword.cos_full_notm}} repository.

### What is happening?
{: #tsSymptoms-cos}

Veeam cannot connect to {{site.data.keyword.cos_full_notm}} repository.

### Why is it happening?
{: #tsCauses-cos}

- Repository settings are incorrect.
- When the VBR server cannot reach the {{site.data.keyword.cos_short}} endpoint.
- Incorrect HMAC credentials.
- An invalid endpoint format.
- Missing bucket access.
- Network restrictions.

### How do you fix it?
{: #tsResolve-cos}

1. Verify that the hash-based message authentication code (HMAC) credentials are correct:
   - Check `access_key_id` and `secret_access_key`.
   - Ensure that the credentials are assigned to the Writer role.
2. Verify the endpoint URL:
   - Use the correct regional endpoint (for example, `s3.us-south.cloud-object-storage.appdomain.cloud`).
   - Exclude the `https://` prefix.
3. Check network connectivity:
   - Ensure that the VBR server can reach the {{site.data.keyword.cos_short}} endpoints.
   - Run: `Test-NetConnection -ComputerName s3.us-south.cloud-object-storage.appdomain.cloud -Port 443`.
4. Verify bucket permissions:
   - Ensure that the bucket exists and is accessible.
   - Ensure that the IAM policies allow access to the bucket.

## Slow backup performance
{: #veeam-vbr-vpc-troubleshoot-performance}

Use this section to review common causes of slow backup operations and identify changes that can improve performance.

### What is happening?
{: #tsSymptoms-performance}

Backups take longer than expected or fail to complete within the backup window.

### Why is it happening?
{: #tsCauses-performance}

- Limited network throughput.
- Constrained disk performance.
- Resource contention on the VBR server.
- Protected virtual server instances.
- Compression settings.
- Concurrent task levels.
- Storage profile limits.

### How do you fix it?
{: #tsResolve-performance}

1. Check network bandwidth:
   - Verify network connectivity between the VBR server and target virtual servers.
   - Use `iperf3` to measure throughput.
2. Verify disk performance:
   - Check the input/output operations per second (IOPS) limits for block storage volumes.
   - Upgrade to higher IOPS profiles.
   - Use virtual server profiles that support high IOPS. For more information, see [Bandwidth allocation for instance profiles](/docs/vpc?topic=vpc-bandwidth-allocation-profiles).
3. Adjust parallel processing:
   - Reduce concurrent tasks if CPU or RAM usage is saturated.
   - Increase concurrent tasks if system resources are underutilized.
4. Review compression settings:
   - Reduce compression levels to improve backup speed.
   - Balance compression levels against storage space requirements.
5. Check for bottlenecks:
   - Review Veeam performance counters.
   - Monitor CPU, RAM, disk, and network utilization.

## Agent installation fails
{: #veeam-vbr-vpc-troubleshoot-agent}

Use this section to diagnose issues that prevent the Veeam agent from installing on target virtual server instances.

### Why is it happening?
{: #tsCauses-agent-fails}

- When the target system does not meet Veeam requirements.
- When the VBR server cannot complete remote installation tasks.
- Unsupported operating systems.
- Blocked ports.
- Insufficient privileges.
- Connectivity and name resolution problems.

### What is happening?
{: #tsSymptoms-agent-fails}

Veeam agent fails to install on a target virtual server.

### How do you fix it?
{: #tsResolve-agent-fails}

1. Verify minimum system requirements for the virtual server:
   - Ensure that the system is running with Windows Server 2012 R2 or later.
   - Ensure that Linux systems use supported distributions such as Red Hat Enterprise Linux, Ubuntu, and SUSE Linux Enterprise Server.
2. Check firewall configuration:
   - Ensure that security groups allow Veeam ports (`2500-3300`, `6160-6163`).
   - Verify Windows Firewall or `iptables` rules.
3. Verify credentials:
   - Ensure that the provided credentials have administrative privileges.
   - Verify that SSH-key based authentication works for Linux systems.
4. Check network connectivity:
   - Ping the target virtual server from the VBR server.
   - Verify DNS resolution when using hostnames.

## Backup job fails with "insufficient space"
{: #veeam-vbr-vpc-troubleshoot-space}

Use this section to troubleshoot backup failures that are caused by limited repository or volume capacity.

### What is happening?
{: #tsSymptoms-space}

Backup jobs fail because of insufficient disk space.

### Why is it happening?
{: #tsCauses-space}

- Backup repository or backup volume does not have enough available capacity for new restore points or retention operations.
- SOBR offload settings.
- Retention policies.
- Local storage sizing.

### How do you fix it?
{: #tsResolve-space}

1. Monitor local repository space:
   - Check the available space on the `E:` drive.
   - Review space usage in the Veeam console.
2. Adjust SOBR capacity tier offload settings:
   - Reduce the `Move backups to object storage` days.
   - Enable `Copy backups to object storage as soon as they are created`.
3. Review retention policies:
   - Reduce the number of restore points to keep.
   - Enable Grandfather-Father-Son (GFS) retention.
4. Expand storage capacity:
   - Increase the size of the backup volume or add another volume through the {{site.data.keyword.cloud_notm}} console.
   - Extend the volume by using Windows Disk Management.

## Log locations
{: #veeam-vbr-vpc-log-locations}

Use these log locations to review Veeam Backup & Replication activity and troubleshoot configuration or backup issues. The paths identify where the VBR server, agents, and SQL Server store diagnostic information.

- VBR Server Logs: `C:\ProgramData\Veeam\Backup\`
- Agent Logs:
   - Windows: `C:\ProgramData\Veeam\Endpoint\`
   - Linux: `/var/log/veeam/`
- SQL database logs: `C:\Program Files\Microsoft SQL Server\MSSQL15.VEEAMSQL2016\MSSQL\Log\`
