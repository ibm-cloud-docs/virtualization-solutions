---

copyright:
  years: 2026
lastupdated: "2026-07-31"

keywords: MTV troubleshooting, Migration Toolkit for Virtualization troubleshooting, VMware vSphere migration troubleshooting, OpenShift Virtualization migration errors, VMware to OpenShift Virtualization migration, warm migration retry limit, CBT snapshot limit, VDDK image pull denied, ESXi DNS resolution, virt-v2v read-only file system, preserve static IP migration, ForkliftController file system overhead, VMware migration plan failure, OpenShift Virtualization migration troubleshooting

subcollection: virtualization-solutions

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

# Troubleshooting MTV migrations from VMware vSphere to Red Hat OpenShift Virtualization
{: #troubleshooting-mtv-migration}

Troubleshoot common Migration Toolkit for Virtualization (MTV) errors when you
migrate VMware vSphere&reg; virtual machines (VMs) to
{{site.data.keyword.redhat_openshift_notm}} Virtualization, including warm
migration retry limits, Virtual Disk Development Kit (VDDK) image pull
failures, Domain Name System (DNS) resolution issues, `virt-v2v` file system
errors, and preserve static IP problems.
{: shortdesc}

## Warm import retry limit reached
{: #ts-warm-import-retry-limit}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-warm-import-retry}

A warm migration fails with an error that indicates that warm import exceeded
the retry limit.

### Why is it happening?
{: #tsCauses-warm-import-retry}

MTV created more than 28 changed block tracking (CBT) snapshots for the source
VM. A single VM supports a maximum of 28 CBT snapshots. When this limit is
exceeded, the warm migration cannot continue.

### How do you fix it?
{: #tsResolve-warm-import-retry}

1. Delete older CBT snapshots on the source VM to reduce the count to fewer
   than 28 snapshots.
2. If appropriate, reduce snapshot churn by increasing the precopy interval
   `controller_precopy_interval`.
3. Restart the migration plan.

## Unable to resize the disk image to the required size
{: #ts-disk-resize-failure}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-disk-resize}

The migration fails with an error that the disk image cannot be resized to the
required size.

### Why is it happening?
{: #tsCauses-disk-resize}

The destination VM persistent volumes that use `ext4` on block storage exceed
the default 10% file system overhead that Containerized Data Importer (CDI)
assumes. As a result, there is insufficient space for the root partition.

### How do you fix it?
{: #tsResolve-disk-resize}

1. Edit the `ForkliftController` custom resource (CR).
2. Increase the `controller_filesystem_overhead` parameter to a value greater
   than `0.10`, such as `0.15`.
3. Apply the change.
4. Rerun the migration.

## Migration plan fails after creation
{: #ts-plan-fails-after-creation}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-plan-fails-after-creation}

A migration plan fails immediately after it is created, before any VMs are
transferred.

### Why is it happening?
{: #tsCauses-plan-fails-after-creation}

Virtual Disk Development Kit (VDDK) image pulls are denied because the
validator pod cannot authenticate to the internal
{{site.data.keyword.redhat_openshift_notm}} image registry.

### How do you fix it?
{: #tsResolve-plan-fails-after-creation}

Grant pull access to the `default` service account in the target namespace. Run
the following command in the Red Hat OpenShift web terminal or from a local system that
has the `oc` CLI installed.

```sh
oc adm policy add-cluster-role-to-user registry-viewer system:serviceaccount:<target-namespace>:default
```
{: codeblock}

Replace `target-namespace` with your target namespace.

## Migration plan fails during initialize phase
{: #ts-plan-fails-initialize}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-plan-fails-initialize}

A migration plan fails during the initialization phase with errors that
indicate connectivity or hostname resolution problems.

### Why is it happening?
{: #tsCauses-plan-fails-initialize}

The importer pod cannot resolve ESXi hostnames. Domain Name System (DNS)
lookups fail for port 902 connections between the
{{site.data.keyword.redhat_openshift_notm}} cluster and the VMware ESXi hosts.

### How do you fix it?
{: #tsResolve-plan-fails-initialize}

Configure Domain Name System (DNS) forwarding for the vCenter or ESXi domain.
Add a forwarding zone that points to your domain controllers.

```yaml
servers:
- forwardPlugin:
        policy: Random
        upstreams:
        - <domain-controller-ip-1>
        - <domain-controller-ip-2>
    name: vcs-resolver
    zones:
    - vcs.example.com
```
{: codeblock}

In the YAML configuration, replace `domain-controller-ip-1` and
`domain-controller-ip-2` with the IP addresses of your domain controllers.
Replace `vcs.example.com` with your vCenter or ESXi domain. Then, apply the DNS
change after you save the configuration.

## `virt-v2v`: file system mounted read only
{: #ts-virt-v2v-read-only}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-virt-v2v-read-only}

The `virt-v2v` conversion tool reports that the file system is mounted
read only, and the migration fails.

### Why is it happening?
{: #tsCauses-virt-v2v-read-only}

The source Windows&reg; VM was not shut down cleanly before the Open
Virtualization Archive (OVA) was exported. Fast Startup or hibernation left the
file system in a dirty state, which prevents `virt-v2v` from mounting it for
read/write access.

### How do you fix it?
{: #tsResolve-virt-v2v-read-only}

1. Disable Fast Startup on the source Windows VM. Go to **Control Panel** >
   **Power Options** > **Choose what the power buttons do**, and clear the
   **Turn on fast startup** checkbox.
2. Disable hibernation. From an elevated command prompt, run
   `powercfg /h off`.
3. Perform a clean shutdown by running `shutdown /s /t 0`.
4. Reexport the OVA after the clean shutdown completes.
5. Upload the new OVA to the NFS server.
6. Retry the migration.

## Preserve static IP: destination subnet mismatch
{: #ts-static-ip-subnet-mismatch}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-static-ip-subnet-mismatch}

After you enable Preserve static IPs, the migrated VM does not receive the
expected IP address, and the destination network has a subnet mismatch.

### Why is it happening?
{: #tsCauses-static-ip-subnet-mismatch}

The destination Layer 2 primary network has a different subnet than the source
network.

### How do you fix it?
{: #tsResolve-static-ip-subnet-mismatch}

Delete and re-create the Layer 2 primary network so that it matches the correct
subnet. Then, rerun the migration.

## Preserve static IP: VM receives a free IP instead of the requested IP
{: #ts-static-ip-free-ip-assigned}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-static-ip-free-ip-assigned}

After you enable Preserve static IPs, the migrated VM is assigned as a free IP
address rather than the static IP from the source.

### Why is it happening?
{: #tsCauses-static-ip-free-ip-assigned}

The source VM was not powered on at the time of migration, or the VMware guest
agent was not installed. Both conditions are required for static IP
preservation to work correctly.

### How do you fix it?
{: #tsResolve-static-ip-free-ip-assigned}

1. Delete the incorrectly migrated VM.
2. Verify that the VMware guest agent (VMware Tools or `open-vm-tools`) is
   installed on the source VM.
3. Ensure that the source VM is powered on.
4. Retry the migration.

## Preserve static IP: network map points to a Layer 2 secondary network
{: #ts-static-ip-layer2-secondary}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-static-ip-layer2-secondary}

After you enable Preserve static IPs, the static IP is not preserved, and the
network map references a Layer 2 secondary network.

### Why is it happening?
{: #tsCauses-static-ip-layer2-secondary}

Layer 2 secondary networks do not support the `Preserve static IP` feature.

### How do you fix it?
{: #tsResolve-static-ip-layer2-secondary}

Create a Layer 2 secondary network without IP address management (IPAM), and
use manual IP configuration instead.

## Preserve static IP: network map points to a Localnet or VMNetwork network
{: #ts-static-ip-localnet}
{: troubleshoot}

### What is happening?
{: #tsSymptoms-static-ip-localnet}

After you enable Preserve static IPs, the static IP is not preserved, and the
network map references a Localnet or VMNetwork network.

### Why is it happening?
{: #tsCauses-static-ip-localnet}

Localnet networks do not support the `Preserve static IP` feature.

### How do you fix it?
{: #tsResolve-static-ip-localnet}

Use manual IP configuration on the migrated VM.

## Additional troubleshooting resources
{: #ts-mtv-additional-resources}

For more troubleshooting guidance and information about collected logs, see the
following resources:

- [Collected logs and custom resource information](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/logs-and-crs_mtv#collected-logs-cr-info_mtv){: external}
- [MTV documentation (v2.9)](https://docs.redhat.com/en/documentation/migration_toolkit_for_virtualization/2.9/html/installing_and_using_the_migration_toolkit_for_virtualization/index){: external}
