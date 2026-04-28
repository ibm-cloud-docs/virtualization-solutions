---

copyright:
  years: 2026
lastupdated: "2026-04-28"

keywords: Red Hat OpenShift Virtualization, virtual servers, Red Hat OpenShift Kubernetes Service, VSI, ODF, RBD

subcollection: virtualization-solutions

content-type: known-issues
services: OpenShift Virtualization, VMware

---

{{site.data.keyword.attribute-definition-list}}

# Troubleshooting ODF workloads
{: #troubleshooting-odf-workloads}

When working with Red Hat OpenShift Data Foundation (ODF) storage from local NVMe drives that are in the bare metal servers, you might encounter issues. Issues, symptoms, likely causes, and resolutions are described in the following sections.

## Verification and recovery of missing NVMe disks
{: #verification-and-recovery-of-missing-nvme-disks}

Due to a known issue on {{site.data.keyword.redhat_openshift_full}} Kubernetes Service bare metal nodes, some NVMe drives might not mount correctly after cluster provisioning. This procedure verifies disk visibility and recovers missing drives.

This procedure removes and rescans PCI devices to temporarily take devices offline. Do not perform these steps on devices that are actively serving I/O to ODF or other workloads. Perform this procedure only during initial cluster setup or a planned maintenance window.
{: warning}

1. Log in to the OpenShift web console. Go to **Compute** > **Nodes**, select a **Node**, and open the **Terminal** tab.
2. Run the following command:

   ```bash
   lsblk | grep -c nvme
   ```

    Compare the output with the expected disk count that is defined in the bare metal profile that is used when ordering the cluster.

    - If the numbers match, the node is healthy.
    - If the numbers do not match, continue with the following steps to rediscover the missing disks.

3. Identify problematic devices:

    ```bash
       dmesg | fgrep error
     ```

     The PCI device addresses associated with the missing NVMe disks.
     {: note}

4. Remove the affected PCI devices (repeat for each device found in Step 3):

    ```bash
       echo 1 > /sys/bus/pci/devices/<device_address>/remove
    ```

5. Rescan the PCI bus to rediscover devices:

    ```bash
       echo 1 > /sys/bus/pci/rescan
    ```

6. Verify again:

    ```bash
       lsblk | grep -c nvme
       ```

    Confirm that the disk count now matches the expected number.

7. Repeat the process before this for each remaining node in the cluster.

    Example: The following example shows how 3 missing disks were rediscovered on a node that should have a total of 8 NVMe disks.

    ```text
       sh-5.1# lsblk | fgrep -c nvme
       5
       sh-5.1# dmesg | fgrep error
       [    9.133039] pci 10000:01:00.0: BAR 0: error updating (0xb6010004 != 0xffffffff)
       [    9.140352] pci 10000:01:00.0: BAR 0: error updating (high 0x00000000 != 0xffffffff)
       [    9.987954] pci 10001:01:00.0: BAR 0: error updating (0xc2010004 != 0xffffffff)
       [    9.995260] pci 10001:01:00.0: BAR 0: error updating (high 0x00000000 != 0xffffffff)
       [   10.749558] pci 10002:01:00.0: BAR 0: error updating (0xde010004 != 0xffffffff)
       [   10.756870] pci 10002:01:00.0: BAR 0: error updating (high 0x00000000 != 0xffffffff)
       sh-5.1# echo 1 > /sys/bus/pci/devices/10000:01:00.0/remove
       sh-5.1# echo 1 > /sys/bus/pci/devices/10001:01:00.0/remove
       sh-5.1# echo 1 > /sys/bus/pci/devices/10002:01:00.0/remove
       sh-5.1# echo 1 > /sys/bus/pci/rescan
       sh-5.1# lsblk | fgrep -c nvme
       8
       ```

## PVC stuck in pending
{: #pvc-stuck-in-pending}

Symptom: A PVC remains in `Pending` state after creation.

Diagnosis:

```bash
oc describe pvc <pvc-name>
```

Common causes:

| Events / Condition | Cause | Fix |
| ------------------ | ----- | --- |
| No events, no provisioner activity | StorageClass name is misspelled or doesn't exist. | Verify with `oc get sc`. |
| `waiting for a volume to be created` | CSI provisioner is working but slow. | Wait a few minutes. Check CSI provisioner pod logs. |
| `ExternalProvisioning` | CSI driver is waiting on an external dependency (for example, KMS for encrypted volumes). | Check `ibm-kp-secret` and `ceph-csi-kms-token` exist (see [Encryption](/docs/virtualization-solutions?topic=virtualization-solutions-odf-for-vm-workloads#odf-supported-encryption)). |
| `cannot create encrypted volume from unencrypted volume` (in CSI logs) | Encrypted StorageClass used for a cloned volume from an unencrypted source. | Use nonencrypted SC for root disks (see [Encryption](/docs/virtualization-solutions?topic=virtualization-solutions-odf-for-vm-workloads#odf-supported-encryption)). |
| `ExternalProvisioning` | CSI driver is waiting on an external dependency (for example, KMS for encrypted volumes). | Check `ibm-kp-secret` and `ceph-csi-kms-token` exist (see [Encryption](/docs/virtualization-solutions?topic=virtualization-solutions-odf-for-vm-workloads#odf-supported-encryption)). |
| `cannot create encrypted volume from unencrypted volume` (in CSI logs) | Encrypted StorageClass used for a cloned volume from an unencrypted source. | Use nonencrypted SC for root disks (see [Encryption](/docs/virtualization-solutions?topic=virtualization-solutions-odf-for-vm-workloads#odf-supported-encryption)). |
| `node(s) did not have enough free storage` | Ceph cluster is full or nearfull. | Check `ceph df`. Add capacity or delete unused PVCs. |

{: caption="Common causes of PVC stuck in Pending"}

## OSD down
{: #osd-down}

Symptom: `ceph status` shows one or more OSDs as `down`.

Diagnosis:

```bash
# Check which OSDs are down
oc exec -n openshift-storage ${TOOLS_POD} -- ceph osd tree | grep down

# Check the OSD pod status
oc get pods -n openshift-storage -l app=rook-ceph-osd
```

Common causes:

NVMe drive failure: The underlying drive has failed. Check dmesg on the worker node for I/O errors.
OSD pod evicted or OOMKilled: Resource pressure on the node. Check that pod events with oc describe pod.
Node reboot or maintenance: The OSD come back when the node returns, and Ceph automatically recovers.

With rep3, a single OSD down does not cause data unavailability. I/O continues in degraded mode while Ceph rereplicates. With rep2, I/O to affected PGs is blocked until the OSD returns (see [Understanding data protection](/docs/virtualization-solutions?topic=virtualization-solutions-odf-for-vm-workloads#understanding-data-protection)).

## Ceph HEALTH_WARN: nearfull
{: #ceph-health-warn-nearfull}

Symptom: ODF fires the `CephOSDNearFull` alert, or `ceph status` shows `HEALTH_WARN` with `nearfull osd(s)`.

Cause: One or more OSDs have exceeded 75% usage (ODF `CephOSDNearFull` alert threshold). If usage continues to rise:

- At 85%, Ceph sets the native nearfull OSD flag (mon_osd_nearfull_ratio) and ODF fires the CephOSDCriticallyFull alert.
- At 90%, Ceph stops backfill and recovery to the affected OSD (mon_osd_backfillfull_ratio).
- At 95%, Ceph marks the OSD full (mon_osd_full_ratio), blocks all writes, and issues HEALTH_ERR.

Fix:

- Delete unused PVCs and VolumeSnapshots to free space.
- If the cluster is genuinely running out of capacity, add more ODF worker nodes (in multiples of 3).
- Check for PVCs with excessive snapshot chains consuming hidden space.
- Address `CephOSDNearFull` alerts promptly once OSDs reach 85% or higher, the cluster enters increasingly degraded states that affect recovery and write availability.

```bash
# Check per-OSD usage
oc exec -n openshift-storage ${TOOLS_POD} -- ceph osd df tree
```

## Slow VM storage performance
{: #slow-vm-storage-performance}

Symptom: VMs experience high I/O latency or low IOPS.

Possible causes and diagnostics:

1. Ceph daemons are under-resourced: If using the Lean profile, Ceph OSDs might be CPU-throttled. Check OSD pod CPU usage in the OpenShift metrics dashboard. Consider switching to the Balanced or Performance profile.

2. Custom pool with 1 PG: A custom CephBlockPool without `targetSizeRatio` gets 1 Placement Group, bottlenecking all I/O on a single OSD. Check:

   ```bash
   oc exec -n openshift-storage ${TOOLS_POD} -- ceph osd pool autoscale-status
   ```

   If `PG_NUM` shows `1` for your pool, set `targetSizeRatio: 0.1` on the CephBlockPool CR.

3. Missing `exclusive-lock` image feature: Without this on the StorageClass, write IOPS can be up to 7x worse. Check:

   ```bash
   oc get sc <storageclass-name> -o jsonpath='{.parameters.imageFeatures}'
   ```

   Should include `exclusive-lock`.

4. Cluster usage above 70%: Ceph performance degrades as disks fill. Check `ceph df`.

5. Rebalancing or recovery in progress: After an OSD failure or node addition, Ceph rebalances data across OSDs, consuming I/O bandwidth. Check:

   ```bash
   oc exec -n openshift-storage ${TOOLS_POD} -- ceph status
   ```

   Look for `recovering` or `backfilling` in the PG summary.

## VM stuck in provisioning
{: #vm-stuck-in-provisioning}

Symptom: A new VM remains in Provisioning state.

Diagnosis:

```bash
oc get vm <vm-name> -o yaml | grep -A5 conditions
oc get dv -l app=<vm-name>
oc get pvc
```

Common causes:

- PVC stuck in Pending: For more information, see [PVC stuck in pending](#pvc-stuck-in-pending).
- DataVolume clone failing silently: If you are using an encrypted StorageClass for the root disk, the clone from the unencrypted golden image fails without visible events. Check the CSI controller pod logs in `openshift-storage`.
- Golden image not available: The DataSource in `openshift-virtualization-os-images` might not be ready. Check `oc get datasource -n openshift-virtualization-os-images`.
- Insufficient node resources: The destination node might not have enough CPU or memory for the VM. Check `oc describe node` for resource pressure.

## Live migration fails
{: #live-migration-fails}

Symptom: VM live migration fails or is rejected.

Common causes:

- PVC access mode is ReadWriteOnce (RWO): Live migration requires ReadWriteMany (RWX). VMs that use the generic `ocs-storagecluster-ceph-rbd` StorageClass with RWO PVCs cannot live migrate. Use `ocs-storagecluster-ceph-rbd-virtualization`, which supports RWX block mode.
- Insufficient resources on destination node: The target node must have enough CPU and memory to accept the VM.
- Network connectivity: The migration network must be healthy between source and destination nodes.
