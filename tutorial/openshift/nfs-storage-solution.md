---

copyright:
  years: 2026
lastupdated: "2026-06-30"

keywords: Red Hat OpenShift Virtualization, Red Hat OpenShift Kubernetes Service, VSI, NFS, storage, File Storage for VPC, IBM Cloud

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 60m

---

{{site.data.keyword.attribute-definition-list}}

# IBM VPC File Storage with Red Hat OpenShift Virtualization
{: #file-storage-vpc-virtualization}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMWare"}
{: toc-completion-time="60m"}


Learn how to configure IBM VPC File Storage as persistent storage for {{site.data.keyword.redhat_openshift_full}} Virtualization virtual machine workloads.
{: shortdesc}

Explore the sections to access the guide:

- [Product overview and planning information](#product-overview-and-planning-information) - Understand IBM VPC File Storage capabilities, profiles, IOPS relationships, and storage concepts
- [Installation and configuration](#installation-and-configuration) - Installation of the add-on, configure StorageProfiles, and create virtual machines
- [Additional configuration scenarios](#additional-configuration-scenarios) - Configure cluster defaults, virtual machine (VM) persistent state, and advanced settings
- [Monitoring and troubleshooting](#monitoring-troubleshooting) - Monitor file storage and resolve common issues
- [Best practices and recommendations](#best-practices) - Follow recommended practices for StorageClass selection, capacity planning, and security

## Product overview and planning information
{: #product-overview-and-planning-information}

### Key benefits
{: #key-benefits}

- Network File System (NFS)-based shared storage. Network-attached file storage is accessible across multiple zones and virtual server instances within your Virtual Private Cloud (VPC).
- Native Red Hat OpenShift integration. Full integration with Kubernetes&reg; PersistentVolumeClaims (PVCs) through the IBM VPC File Container Storage Interface (CSI) driver.
- Managed service. This fully managed {{site.data.keyword.cloud_notm}} service includes automated provisioning and lifecycle management and can help ensure storage resiliency if hardware failures occur.
- Cost-effective pricing. Use a pay-as-you-go model for file share storage capacity and performance with no additional licensing fees.
- Flexible performance tiers. Multiple IOPS options are available to match workload requirements.
- Kubernetes snapshot support. Kubernetes PVC snapshots are supported for data protection and recovery, enabling backup tools to work seamlessly.
- Storage efficiency. Ordered capacity is fully allocated with no overprovisioning or increase in reservations.
- Capacity expansion. File storage capacity can expand dynamically to match workload demands.
- Simplified management. Setup is minimal, with no additional expertise required.
- Replication capabilities. Zone-to-zone replication is available for disaster recovery scenarios through the {{site.data.keyword.cloud_notm}} VPC portal, not from the {{site.data.keyword.redhat_openshift_notm}} cluster.

### IBM VPC File Storage overview
{: #file-storage-for-vpc-overview}

IBM VPC File Storage provides NFS-based file storage services within the VPC Infrastructure. File shares provide network-accessible storage that allows multiple clients to access the same folders and files simultaneously.
For more information and considerations, see [Enabling the {{site.data.keyword.cloud_notm}} File Storage for VPC cluster add-on](/docs/openshift?topic=openshift-storage-file-vpc-install).


{{site.data.keyword.redhat_openshift_notm}} on {{site.data.keyword.cloud_notm}} provides predefined StorageClasses that specify the type of IBM VPC File Storage to provision, including available size, IOPS, file system, and retention policy.


### File storage profiles for Red Hat OpenShift Virtualization
{: #file-storage-profiles}

For {{site.data.keyword.redhat_openshift_notm}} Virtualization workloads, use zonal file shares, which provide the optimal balance of performance, cost, and reliability for virtual machine storage.


#### Zonal file shares
{: #zonal-file-shares}

Zonal file shares store data in a single availability zone while providing cross-zone accessibility through virtual network interface (VNI) and security group integration.

For more information about zonal file shares, see [Zonal file share overview](/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#zonal-file-storage-overview).

#### IOPS and capacity relationship for zonal file shares
{: #iops-capacity}

IBM VPC File Storage has specific IOPS ranges based on share size for zonal file shares (dp2 profile). The relationship between capacity and IOPS must be valid according to the dp2 profile specifications.

**Important:** If you provision a PVC with an invalid capacity or IOPS combination, the PVC fails with an error like:

```log

failed to provision volume with StorageClass "ibmc-vpc-file-3000-iops":
rpc error: code = InvalidArgument desc = {RequestID: xxx,
BackendError: {Code:shares_profile_capacity_iops_invalid,
Description:The capacity or IOPS specified in the request is not valid for the 'dp2' profile}}
```
{: screen}


For more information about the zonal file shares profile and a detailed table with the required minimum size per IOPS range, see [Zonal availability profile](/docs/vpc?topic=vpc-file-storage-profiles&interface=ui#dp2-profile).

### Considerations
{: #considerations}

Before you deploy IBM VPC File Storage for virtualization workloads, review the following considerations:

#### Capacity and size limits
{: #capacity-limits}

{{site.data.keyword.cloud_notm}} VPC File Storage has specific capacity limits, IOPS scaling requirements, and quotas that apply to file shares, mount targets, and snapshots. These limits can change as the service evolves.

For current information about:
- File share capacity and IOPS limits. See [About File Storage for VPC](/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui).
- Quotas and service limits. See [Quotas and service limits](/docs/vpc?topic=vpc-quotas#file-storage-quotas).
- Snapshot capabilities and limits. See [About File Storage for VPC snapshots](/docs/vpc?topic=vpc-fs-snapshots-about&interface=ui).

To increase a quota for a particular resource, [contact support](/docs/vpc?topic=vpc-quotas#file-storage-quotas).

#### Feature limitations
{: #feature-limitations}

- Encryption in transit. Not supported for NFS storage.
- PVC cloning. Not supported by the IBM VPC File CSI driver at the Kubernetes level.
- Thin provisioning. Not supported.
- Advanced Cluster Management (ACM) Disaster Recovery. Not supported.

#### Resource group considerations
{: #resource-group}

It is recommended that your {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster and VPC are part of the same resource group. If your cluster and VPC are in separate resource groups, you must create custom StorageClasses and provide your VPC resource group ID.

For troubleshooting and fix information, see [Why does PVC creation fail for File Storage for VPC?](/docs/containers?topic=containers-ts-storage-vpc-file-eit-pvc-fails).


#### Network and security
{: #network-security}

- File shares are provisioned in the `kube-<clusterID>` security group by default.
- Pods can access file shares across nodes and zones within the cluster.
- File shares are ordered based on worker node locations.

### Understanding storage concepts
{: #understanding-storage-concepts}


Before you configure IBM VPC File Storage for {{site.data.keyword.redhat_openshift_notm}} Virtualization, it's essential to understand the core storage concepts and how they work together in an {{site.data.keyword.redhat_openshift_notm}} cluster. In the section, you find the fundamental building blocks that enable storage for both traditional Kubernetes workloads and virtual machines.

### Understanding storage classes and storage profiles
{: #storage-classes-and-profiles}

{{site.data.keyword.redhat_openshift_notm}} uses two complementary resources to manage storage: StorageClasses for general Kubernetes storage provisioning, and StorageProfiles for virtual machine-specific storage behaviors.

#### Storage classes
{: #storage-classes-concept}

A `StorageClass` is a Kubernetes resource that defines how storage is dynamically provisioned. Think of it as a template or blueprint for creating storage volumes.

Purpose
:   Defines the provisioner, parameters, and policies for creating `PersistentVolumes` (`PVs`).

Scope
:   Kubernetes and {{site.data.keyword.redhat_openshift_notm}} cluster-wide resource.

Created by
:   {{site.data.keyword.cloud_notm}} (pre-defined) or cluster administrators (custom).

Used for
:   Specifying storage requirements when you create `PersistentVolumeClaims` (`PVCs`).

Example parameters
:   IOPS tier, profile type (`dp2`), reclaim policy, volume binding mode.

When you create a PVC and specify a `StorageClass`, Kubernetes uses that `StorageClass` to automatically provision the underlying storage (in this case, an {{site.data.keyword.cloud_notm}} File Share).

#### Storage profiles
{: #storage-profiles-concept}


A `StorageProfile` is an {{site.data.keyword.redhat_openshift_notm}} Virtualization (KubeVirt) resource that provides added metadata about how a `StorageClass` can be used for virtual machine workloads.


Purpose
:   Defines VM-specific storage capabilities and behaviors for a `StorageClass`.

Scope
:   {{site.data.keyword.redhat_openshift_notm}} Virtualization specific (not standard Kubernetes).

Created by
:   Automatically generated for each StorageClass, but requires manual configuration for File Storage.

Used for
:   Determining clone strategies, access modes, and volume modes for VM disks.

Example settings
:   Clone strategy (copy vs snapshot), access modes (`ReadWriteMany`), volume mode (`Filesystem`).

`StorageProfiles` tell {{site.data.keyword.redhat_openshift_notm}} Virtualization how to handle VM disk operations like cloning, snapshotting, and live migration for each `StorageClass`.

A `StorageClass` defines how to provision storage, while a `StorageProfile` defines how virtual machines can use that storage.

The {{site.data.keyword.redhat_openshift_notm}} Virtualization `Containerized Data Importer` (`CDI`) automatically creates `StorageProfiles` after you install the {{site.data.keyword.redhat_openshift_notm}} Virtualization operator. Whenever you create a new `StorageClass` in the cluster, `CDI` automatically generates a corresponding `StorageProfile`. However, for IBM VPC File Storage, you must manually configure these auto-generated `StorageProfiles` to add the necessary specifications for proper VM operation. For more information, see [Configuring StorageProfiles for File Storage](#configure-storageprofiles).
{: note}

### Understanding default storage class annotations
{: #default-storage-annotations}

{{site.data.keyword.redhat_openshift_notm}} clusters use two different default StorageClass annotations that serve distinct purposes. Understanding the difference between these annotations is critical for properly configuring storage for both traditional workloads and virtual machines.

#### Cluster default storage class
{: #cluster-default-annotation}

The cluster default `StorageClass` is identified by the `storageclass.kubernetes.io/is-default-class: "true"` annotation.

Purpose and usage:
- Kubernetes uses this `StorageClass` when you create a PVC without specifying a `StorageClass`.
- Applies to all workloads in the cluster (deployments, `StatefulSets`, stand-alone PVCs).
- Only one `StorageClass` in the cluster can have this annotation set to `true`.
- Automatically selected for general-purpose storage requests.

When it is used:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  # No storageClassName specified - cluster default will be used
```
{: codeblock}

#### Red Hat OpenShift Virtualization default storage class
{: #virtualization-default-annotation}

The {{site.data.keyword.redhat_openshift_notm}} Virtualization default `StorageClass` is identified by the `storageclass.kubevirt.io/is-default-virt-class: "true"` annotation.

Purpose and usage:
- Used by the `Containerized Data Importer` (`CDI`) when you create boot volume golden images.
- Specifically, used by {{site.data.keyword.redhat_openshift_notm}} Virtualization for VM-related storage operations.
- Used when you create `DataVolumes` or `DataSources` without specifying a `StorageClass`.
- It can coexist with the cluster default (different StorageClass can have this annotation).

When it is used:
- Creating VM boot disk golden images in the `openshift-virtualization-os-images` namespace.
- Importing VM disk images through CDI.
- Creating DataVolumes without an explicit StorageClass specification.
- Cloning VM disks when no `StorageClass` is specified.

#### Verifying default storage classes
{: #verify-defaults}

To check which `StorageClasses` have default annotations:

```bash
oc get storageclass -o custom-columns='NAME:.metadata.name,CLUSTER-DEFAULT:.metadata.annotations.storageclass\.kubernetes\.io/is-default-class,VIRT-DEFAULT:.metadata.annotations.storageclass\.kubevirt\.io/is-default-virt-class'
```
{: codeblock}

Example output:

```bash

NAME                              CLUSTER-DEFAULT   VIRT-DEFAULT
ibmc-vpc-file-500-iops           true              <none>
ibmc-vpc-file-retain-500-iops    <none>            true
ibmc-vpc-block-10iops-tier       <none>            <none>
```
{: screen}

#### Understanding minimum PVC size annotation
{: #minimum-pvc-size-concept}

The `cdi.kubevirt.io/minimumSupportedPvcSize` annotation is a `StorageProfile` setting that helps `CDI` provision PVCs with a minimum size that satisfies IBM VPC File Storage IOPS and capacity requirements.

When `CDI` creates a PVC for a virtual machine through virtual machine creation, cloning, or `DataVolume` operations, it checks the `StorageProfile` for the `cdi.kubevirt.io/minimumSupportedPvcSize` annotation. If the requested PVC size is smaller than the minimum, `CDI` provisions the PVC with the minimum size instead.

This annotation applies only to PVC sizing decisions that CDI makes during supported {{site.data.keyword.redhat_openshift_notm}} Virtualization CDI operations. It does not affect manually created PVCs. If you create a PVC directly by using `oc create` or a YAML manifest, the annotation is ignored, and you can ensure that the capacity meets the IOPS requirements.

The annotation is also not applied to the temporary restore PVC that `CDI` creates during snapshot-based clone workflows. For that intermediate PVC, `CDI` preserves the restore size from the source `VolumeSnapshot` instead of using the minimum size from the target `StorageProfile`.

Example scenario:

- You create a boot image PVC from an `ibmc-vpc-file-retain-500-iops` `StorageClass` with a size of 30 GiB.
- You use that boot image to create or clone a virtual machine into a target `StorageClass` with 3000 IOPS, such as `ibmc-vpc-file-3000-iops`.
- According to the [IOPS and capacity relationship table](#iops-capacity), 3000 IOPS requires a minimum of 80 GiB.
- During the snapshot-based clone workflow, `CDI` creates a temporary restore PVC from the `VolumeSnapshot`.
- That temporary PVC keeps the source snapshot size of 30 GiB instead of applying `cdi.kubevirt.io/minimumSupportedPvcSize: 80Gi` from the target `StorageProfile`.
- As a result, the workflow can fail because the temporary PVC size does not satisfy the minimum capacity that is required for the 3000 IOPS target `StorageClass`.

```log
(combined from similar events): failed to provision volume with StorageClass "ibmc-vpc-file-3000-iops": rpc error: code = InvalidArgument desc = {RequestID: bea20234-e6cb-490e-b1a2-e99292709f50 , BackendError: {Trace Code:, Code:shares_profile_capacity_iops_invalid, Description:The capacity or IOPS specified in the request is not valid for the 'dp2' profile, RC:400 Bad Request.Failed to create file share with the storage provider}, Action: Please provide valid parameters}
```
{: screen}

This behavior is expected with the current `CDI` logic for snapshot restore PVCs.
{: important}

As mentioned in the [IOPS and capacity relationship](#iops-capacity) section, you can use the minimum size IOPS table from the [Zonal availability profile](/docs/vpc?topic=vpc-file-storage-profiles&interface=ui#dp2-profile) to get the correct annotation value for your specific `StorageClass`. For example, if you want to annotate the `ibmc-vpc-file-3000-iops` `StorageClass`, as of now the current minimum size that is required for 3000 IOPS is 80 GiB. You need to use this approach whenever you create an IBM VPC File Storage `StorageClass` with a different IOPS tier.
{: tip}

### Advanced storage configuration concepts
{: #advanced-storage-configuration-concepts}

The section covers advanced storage configuration topics for {{site.data.keyword.redhat_openshift_notm}} Virtualization, including clone strategies and performance optimization.

#### Understanding CDI boot volumes
{: #cdi-boot-volumes}

With {{site.data.keyword.redhat_openshift_notm}} Virtualization, you can create and use VM templates to provision system environments consistently. For more information about VM templates, see [Creating virtual machines from Red Hat&reg; images](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/virtualization/advanced-vm-creation#virt-creating-vms-from-rh-images-overview){: external}.

#### Understanding CDI clone strategies
{: #cdi-clone-strategies}

When {{site.data.keyword.redhat_openshift_notm}} Virtualization needs to create a new VM disk from an existing source (such as cloning a VM or creating a VM from a template), the `Containerized Data Importer` (`CDI`) uses a clone strategy to copy the data. The clone strategy is defined in the `StorageProfile` and determines how efficiently and reliably the cloning operation is run.

`CDI` supports three clone strategies: `snapshot`, `copy`, and `csi-clone`. To learn more about the clone strategies, see [Customizing the storage profile default cloning strategy](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html/virtualization/storage#virt-customizing-storage-profile-default-cloning-strategy_virt-configuring-storage-profile){: external}.

#### Clone strategy for IBM Cloud File Storage
{: #clone-strategy-vpc-file}

For IBM VPC File Storage, both `snapshot` and `host cloning` strategies are supported, with `snapshot` as the recommended approach. Note that the `csi-clone` strategy is not supported for this provider.

How the snapshot clone strategy works:

1. CDI creates a snapshot of the source PersistentVolume by using the VPC File CSI driver.
2. The system creates a new PersistentVolume from the snapshot.
3. The system binds the new PV to the target PersistentVolumeClaim.

How the host copy clone strategy works:

1. CDI creates a temporary pod that mounts both the source and target volumes.
2. The pod copies the data from the source volume to the target volume over the cluster network.
3. After the copy completes, `CDI` removes the temporary resources, and the target PVC is ready for use.

Key benefits for IBM VPC File Storage:

- Snapshot strategy performance. Fastest cloning method.
- Snapshot strategy efficiency. Uses storage-level copy-on-write, minimizing data duplication.
- Snapshot strategy network usage. Minimal because data stays within the storage backend.
- Host copy strategy compatibility. Can be used when snapshot-based cloning is not suitable for the source and target storage combination.

Important considerations:

- Snapshot-based cloning is the preferred strategy for IBM VPC File Storage.
- Snapshot-based cloning requires the snapshot to be compatible with the target volume size and IOPS requirements.
- Host copy cloning is slower than snapshot-based cloning because data is copied through a temporary pod over the cluster network.
- Some cross-storage-class cloning scenarios might require the host copy strategy if IOPS and capacity combinations are invalid for snapshot-based cloning.

The clone strategy is defined in `StorageProfiles`. To modify `StorageProfiles` to update the clone strategy, see [Configuring StorageProfiles for File Storage](#configure-storageprofiles).

#### Understanding `dataImportCronSourceFormat`
{: #dataimportcronsourceformat}

The `dataImportCronSourceFormat` property is a `StorageProfile` setting that controls how `CDI` stores and manages boot volume *golden images* for virtual machine provisioning.

In general CDI behavior, this property determines the format that CDI uses for the source artifacts that `DataImportCron` manages.

Without `dataImportCronSourceFormat` set to `snapshot`, `CDI` stores boot volume golden images as PVCs. When you create multiple virtual machines concurrently from the same template by using snapshot-based cloning, each virtual machine creation can trigger a new snapshot operation from the source PVC.

With IBM VPC File Storage, this behavior is important because the provider currently supports only one snapshot per minute for a file share. As a result, concurrent virtual machine provisioning from the same boot volume source can be delayed when CDI must create repeated snapshots from the source PVC.

When `dataImportCronSourceFormat` is set to `snapshot`, `CDI` stores the boot volume source in snapshot form instead of relying on a PVC as the reusable source artifact. `CDI` expects the storage backend snapshot to remain usable even after the source PVC is deleted, so it creates the snapshot artifact and then removes the source PVC. With IBM VPC File Storage, this behavior is important because snapshots remain connected to the underlying file share rather than being fully disconnected snapshot artifacts.

With IBM VPC File Storage, this approach still helps avoid repeated snapshot creation from the same source file share during concurrent virtual machine provisioning. However, you must also understand that `CDI` snapshot-based boot volume handling assumes backend snapshot behavior that differs from the current IBM VPC File Storage provider implementation.

#### Clone strategy considerations for IBM Cloud File Storage
{: #clone-strategy-considerations-for-ibm-cloud-storage}

Before you configure IBM VPC File Storage as the default backend storage for {{site.data.keyword.redhat_openshift_notm}} Virtualization, consider which clone strategy best fits your virtual machine provisioning requirements. Your choice depends on provisioning speed needs, IOPS diversity, and operational preferences.

##### Fastest provisioning for concurrent VM creation
{: #fastest-provisioning-option}

For the fastest virtual machine provisioning when you create multiple VMs concurrently from the same boot volume template, use snapshot-based cloning with `dataImportCronSourceFormat` set to `snapshot`.

Requirements:

- Configure a StorageClass with a `Retain` reclaim policy as the default StorageClass for OpenShift Virtualization. You can use any pre-shipped `Retain` reclaim policy StorageClass.
- Set `dataImportCronSourceFormat: snapshot` in the StorageProfile

Trade-offs:

- Storage classes with `Retain` reclaim policy require manual cleanup of PersistentVolumes after virtual machines are deleted
- Best suited for environments where all virtual machines use the same IOPS tier

For detailed information, see [Understanding dataImportCronSourceFormat](#dataimportcronsourceformat) and [Set the {{site.data.keyword.redhat_openshift_notm}} Virtualization default StorageClass](#set-virt-default).

##### Multiple IOPS tiers for different workload types
{: #multiple-iops-option}

If you need to provision virtual machines with varying performance requirements across different IOPS tiers, create separate boot volume templates for each `StorageClass` you plan to use.

When to use:

- You need to provision virtual machines with different IOPS requirements (for example, development VMs with 500 IOPS and production VMs with 3000 IOPS)
- You want to optimize costs by using appropriate IOPS tiers for different workload types
- You are working with StorageClasses that have different minimum size requirements based on their IOPS tiers

Approach:

- Create a boot volume template for each IOPS tier StorageClass.
- Select the appropriate template when you provision virtual machines based on their performance requirements.

For information about IOPS tiers and minimum size requirements, see [Understanding minimum PVC size annotation](#minimum-pvc-size-concept).

##### Host-assisted cloning for flexibility
{: #host-assisted-option}

If provisioning speed is not critical and you need flexibility for cross-storage-class cloning, use the host-assisted copy clone strategy.

When to use:

- Provisioning time is not a primary concern.
- You need to clone virtual machines across different `StorageClasses`.
- You want to avoid managing multiple boot volume templates.

Trade-offs:

- Slower than snapshot-based cloning because data is copied through a temporary pod over the cluster network
- Network bandwidth and cluster resources can affect cloning performance

For configuration instructions, see [Configuring the host-assisted clone strategy](#config-host-assisted-clone).

## Installation and configuration
{: #installation-and-configuration}

### Prerequisites
{: #prerequisites}

This guidance assumes that your cluster already has {{site.data.keyword.redhat_openshift_notm}} Virtualization that is installed. For more information on installing {{site.data.keyword.redhat_openshift_notm}} Virtualization, see [Installing the {{site.data.keyword.redhat_openshift_notm}} Virtualization Operator](#install-virt-operator). The focus of the section is the storage configuration that is required for virtual machine workloads.

Before you set up IBM VPC File Storage with {{site.data.keyword.redhat_openshift_notm}} Virtualization, you can ensure that you have:

#### Cluster requirements
{: #cluster-requirements}

- {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster on VPC infrastructure. Cluster running on IBM Cloud VPC.
- Bare metal worker nodes. Required for {{site.data.keyword.redhat_openshift_notm}} Virtualization. Must run Red Hat CoreOS&reg;.
- {{site.data.keyword.redhat_openshift_notm}} version. Version 4.18 or later for {{site.data.keyword.redhat_openshift_notm}} Virtualization support.
- Outbound traffic protection. Disabled for pulling the image.
- {{site.data.keyword.redhat_openshift_notm}} Virtualization. Already installed and available on the cluster. For installation instructions, see [Installing the {{site.data.keyword.redhat_openshift_notm}} Virtualization Operator](#install-virt-operator).

Bare-metal node has a "d" suffix (such as bx2d or cx2d) include an extra NVMe&trade; local disk, which is not needed for a File Storage cluster on {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service.
{: note}

{{site.data.keyword.redhat_openshift_notm}} Virtualization on {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service VPC clusters currently supports bare metal worker nodes only. Virtualized (shared vCPU) worker nodes are not supported.
{: important}

#### Access requirements
{: #access-requirements}

- {{site.data.keyword.cloud_notm}} account with appropriate permissions.
- {{site.data.keyword.redhat_openshift_notm}} Command Line Interface (oc CLI).
- Access to the {{site.data.keyword.cloud_notm}} console.
- Sufficient permissions to update StorageClasses, StorageProfiles, VolumeSnapshotClasses, and the HyperConverged custom resource.

### Installing the File Storage for VPC add-on
{: #install-addon}

The IBM VPC File CSI driver must be installed as a cluster add-on before you can provision file storage.

For installation instructions for the File Storage for VPC add-on, see [Installing the File Storage for VPC add-on](/docs/openshift?topic=openshift-storage-file-vpc-install#file-vpc-addon-enable).

### Verifying the installation
{: #verify-installation}

After installation, verify that the File Storage CSI driver pods are running:

```bash
oc get pods -n kube-system | grep vpc-file-csi
```
{: codeblock}

You should see pods similar to:

```bash
ibm-vpc-file-csi-controller-xxxx (two for HA)
ibm-vpc-file-csi-node-xxxxx (one per worker node)
```
{: screen}

### Default storage classes
{: #default-storage-classes}

When you install the IBM VPC File Storage add-on, the provider automatically creates `StorageClasses`.

You need to either uninstall the VPC Block CSI driver (as it has a default `StorageClass`) or change the VPC Block default `StorageClass` to nondefault.
{: note}

#### Storage class parameters
{: #storage-class-parameters}

For a complete list of default `StorageClasses` and their characteristics, including IOPS tiers, volume binding modes, and reclaim policies, see [Storage class reference documentation](/docs/openshift?topic=openshift-storage-file-vpc-sc-ref).

#### Creating custom storage classes
{: #custom-storage-classes}

For a list of available features and instructions on creating custom `StorageClasses`, see [Creating your own storage class](/docs/openshift?topic=openshift-storage-file-vpc-apps#storage-file-vpc-custom-sc).

### Setting up IBM VPC File Storage for Red Hat OpenShift Virtualization
{: #setup-virtualization}

Currently, IBM VPC File Storage requires manual configuration to work optimally with {{site.data.keyword.redhat_openshift_notm}} Virtualization. IBM is working to provide special support for this integration in a future release. The section guides you through the required manual configuration steps.
{: shortdesc}

Until these enhancements are available, you must complete the following manual configuration steps:
{: important}

- StorageProfile specifications. Configuration of access modes, volume modes, and clone strategies for all File Storage `StorageClasses`.
- Minimum size annotations. Minimum PVC size requirements based on IOPS tiers to prevent provisioning failures.
- VM state StorageClass. Custom `StorageClass` for virtual Trusted Platform Module (vTPM) and Unified Extensible Firmware Interface (UEFI) support that is required by Windows&reg; 11 and Windows Server 2022 virtual machines.
- Default VolumeSnapshotClass. Default VolumeSnapshotClass configuration for virtual machine snapshots and data protection.

If you need to install {{site.data.keyword.redhat_openshift_notm}} Virtualization, prepare a cluster from scratch, or change the default StorageClass in the cluster, see [Additional configuration scenarios](#additional-configuration-scenarios).

#### Recommended configuration flow
{: #recommended-config-flow}

Use the following sequence to configure IBM VPC File Storage for {{site.data.keyword.redhat_openshift_notm}} Virtualization:

1. Configure the default `VolumeSnapshotClass`.
2. Configure `StorageProfiles` for IBM VPC File Storage `StorageClasses`.

This sequence will help ensure that cloning and snapshots work as expected for IBM VPC File Storage virtual machine workloads.

If your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment already uses another storage provider, such as [{{site.data.keyword.redhat_openshift_notm}} Data Foundation (ODF)](https://www.redhat.com/en/technologies/cloud-computing/openshift-data-foundation){: external}, you do not need to change the cluster default `StorageClasses` to complete the IBM VPC File Storage-specific configuration in the section.
However, if the environment is intended to use IBM VPC File Storage as the primary storage provider for {{site.data.keyword.redhat_openshift_notm}} Virtualization, it is recommended that you complete the optional steps in [Optional default `StorageClass` reconfiguration](#default-storageclass-reconfiguration).

If your virtual machines require persistent state storage in IBM VPC File Storage for features such as vTPM or UEFI, complete the optional steps in [Optional virtual machine persistent state configuration](#optional-vm-persistent-state-configuration).

##### Step 1: Configure default VolumeSnapshotClass
{: #configure-volumesnapshotclass}

When you install the IBM VPC File Storage add-on, the system automatically creates two `VolumeSnapshotClasses`, but neither is set as the default. You must configure one as the default so that {{site.data.keyword.redhat_openshift_notm}} Virtualization can use snapshots effectively.

For IBM VPC File Storage snapshot functions in the provider, including configuration and usage instructions, see [Setting up volume snapshots for VPC file storage](/docs/openshift?topic=openshift-vpc-volume-snapshot-file).

Verify that a default `VolumeSnapshotClass` is configured for IBM VPC File Storage:

```bash
oc get volumesnapshotclass -o jsonpath='{range .items[?(@.metadata.annotations.snapshot\.storage\.kubernetes\.io/is-default-class=="true")]}{.metadata.name}{"\t"}{.driver}{"\n"}{end}'
```
{: codeblock}

Validate that the command returns a default `VolumeSnapshotClass` for IBM VPC File Storage. The output must include an `ibmc-vpcfile` snapshot class and the `vpc.file.csi.ibm.io` driver.

A default `VolumeSnapshotClass` is required for {{site.data.keyword.redhat_openshift_notm}} Virtualization snapshot-based cloning operations to work correctly. Without a default `VolumeSnapshotClass`, virtual machine cloning and boot image management operations that rely on snapshots fail.
{: important}

##### Step 2: Configuring storage profiles for File Storage
{: #configure-storageprofiles}

The `vpc.file.csi.ibm.io` provisioner is not yet automatically recognized by the `Containerized Data Importer` (`CDI`). You must manually configure `StorageProfiles` to add the required specifications and optimize for IBM VPC File Storage characteristics.

#### Why StorageProfile configuration is needed?
{: #why-storageprofile-config}

`StorageProfile` configuration is required for the following reasons:

- Add required specifications such as `accessModes` and `volumeMode` that `CDI` does not automatically detect.
- Set the appropriate `cloneStrategy` for cross-storage-class virtual machine creation.
- Set the `minimumSupportedPvcSize` annotation to minimize PVC creation errors.

##### Patching storage profiles
{: #patch-storageprofiles}

Use the following commands to patch File Storage `StorageProfiles`. These patches configure the required specifications including access modes, volume mode, clone strategy, and the minimum PVC size annotation.

Patch all `StorageClasses` with less than 3000 IOPS with a 10 GiB minimum:

```bash
for sp in $(oc get storageprofile.cdi.kubevirt.io -o json | \
  jq -r '.items[] | select(.status.provisioner == "vpc.file.csi.ibm.io" and (.metadata.name | contains("3000") | not)) | .metadata.name'); do
  oc patch storageprofile.cdi.kubevirt.io "$sp" --type=merge -p '{
    "metadata": {
      "annotations": {
        "cdi.kubevirt.io/minimumSupportedPvcSize": "10Gi"
      }
    },
    "spec": {
      "claimPropertySets": [
        {
          "accessModes": ["ReadWriteMany"],
          "volumeMode": "Filesystem"
        }
      ],
      "cloneStrategy": "snapshot"
    }
  }'
done
```
{: codeblock}

Patch all 3000 IOPS `StorageClasses` with an 80 GiB minimum:

```bash
for sp in $(oc get storageprofile.cdi.kubevirt.io -o json | \
  jq -r '.items[] | select(.status.provisioner == "vpc.file.csi.ibm.io" and (.metadata.name | contains("3000"))) | .metadata.name'); do
  oc patch storageprofile.cdi.kubevirt.io "$sp" --type=merge -p '{
    "metadata": {
      "annotations": {
        "cdi.kubevirt.io/minimumSupportedPvcSize": "80Gi"
      }
    },
    "spec": {
      "claimPropertySets": [
        {
          "accessModes": ["ReadWriteMany"],
          "volumeMode": "Filesystem"
        }
      ],
      "cloneStrategy": "snapshot"
    }
  }'
done
```
{: codeblock}

These configurations:

- Set `minimumSupportedPvcSize` based on the IOPS tier.
- Set `accessModes` to `ReadWriteMany`, which is required for NFS.
- Set `volumeMode` to `Filesystem`, which is required for file storage.
- Use `cloneStrategy: snapshot` for efficient cloning.

When you create custom `StorageClasses` with different IOPS tiers, use the [minimum PVC size annotation table](#minimum-pvc-size-concept) to set the appropriate `minimumSupportedPvcSize` annotation for the corresponding `StorageProfile`.
{: tip}

##### Verifying storage profile configuration
{: #verify-storageprofile}

For a `StorageClass` with less than 3000 IOPS, verify that the minimum size is set to 10 GiB. For example,

```bash
oc get storageprofile.cdi.kubevirt.io ibmc-vpc-file-500-iops -o yaml
```
{: codeblock}

Check that the output includes:

```yaml
metadata:
  annotations:
    cdi.kubevirt.io/minimumSupportedPvcSize: 10Gi
spec:
  claimPropertySets:
  - accessModes:
    - ReadWriteMany
    volumeMode: Filesystem
  cloneStrategy: snapshot
```
{: codeblock}

For a 3000 IOPS `StorageClass`, verify that the minimum size is set to 80 GiB:

```bash
oc get storageprofile.cdi.kubevirt.io ibmc-vpc-file-3000-iops -o yaml
```
{: codeblock}

Check that the output includes:

```yaml
metadata:
  annotations:
    cdi.kubevirt.io/minimumSupportedPvcSize: 80Gi
spec:
  claimPropertySets:
  - accessModes:
    - ReadWriteMany
    volumeMode: Filesystem
  cloneStrategy: snapshot
```
{: codeblock}

### Working with virtual machines
{: #working-with-vms}

After you configure IBM VPC File Storage as the default `StorageClass` and set up `StorageProfiles` properly, you can create and manage virtual machines.

### Creating a virtual machine
{: #create-vm}

You can create VMs through the {{site.data.keyword.redhat_openshift_notm}} web console or by using YAML manifests.

#### By using the web console
{: #create-vm-console}

1. Browse to **Virtualization** > **virtual machines**
2. Click **Create** > **From template** or **From InstanceType**
3. Select your wanted template or instance type
4. Configure VM settings (name, namespace, resources)
5. The VM automatically uses the default File Storage class for its disks
6. Click **Create virtual machine**

#### By using a YAML manifest
{: #create-vm-yaml}

Example of a VM definition that uses File Storage:

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: rhel9-vm
  namespace: default
spec:
  running: true
  template:
    spec:
      domain:
        devices:
          disks:
          - disk:
              bus: virtio
            name: rootdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
        resources:
          requests:
            memory: 4Gi
            cpu: 2
      volumes:
      - name: rootdisk
        persistentVolumeClaim:
          claimName: rhel9-vm-rootdisk
      - name: cloudinitdisk
        cloudInitNoCloud:
          userData: |
            #cloud-config
            user: cloud-user
            password: changeme
            chpasswd: { expire: False }
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rhel9-vm-rootdisk
  namespace: default
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 30Gi
  storageClassName: ibmc-vpc-file-500-iops
```
{: codeblock}

### Using different storage classes for VM disks
{: #different-storage-classes}

You can use different `StorageClasses` for different VM disks based on performance requirements:

Boot disk
:   Use reduced IOPS (for example, `ibmc-vpc-file-500-iops`) for cost-effective storage, as booting is a one-time operation.

Data disks
:   Use higher IOPS (for example, `ibmc-vpc-file-3000-iops`) for better runtime performance where it matters most.

Example VM with multiple disks:

```yaml
spec:
  template:
    spec:
      domain:
        devices:
          disks:
          - disk:
              bus: virtio
            name: bootdisk
          - disk:
              bus: virtio
            name: datadisk
      volumes:
      - name: bootdisk
        persistentVolumeClaim:
          claimName: vm-boot-pvc
      - name: datadisk
        persistentVolumeClaim:
          claimName: vm-data-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: vm-boot-pvc
spec:
  storageClassName: ibmc-vpc-file-3000-iops
  accessModes: [ReadWriteMany]
  resources:
    requests:
      storage: 80Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: vm-data-pvc
spec:
  storageClassName: ibmc-vpc-file-500-iops
  accessModes: [ReadWriteMany]
  resources:
    requests:
      storage: 200Gi
```
{: codeblock}

### VM live migration
{: #vm-live-migration}

{{site.data.keyword.redhat_openshift_notm}} Virtualization supports live migration of VMs between worker nodes. With IBM VPC File Storage's ReadWriteMany access mode, multiple nodes can access VM disks simultaneously, enabling seamless live migration.
IBM VPC File Storage also supports storage migration.

Requirements for live migration:
- VM must be running
- Source and destination nodes must have network connectivity
- File storage must be accessible from both nodes (automatically satisfied with IBM VPC File Storage)

Live migration is triggered automatically during:
- Node maintenance or updates
- Node failures (if configured)
- Manual migration requests

### Customizing file storage
{: #customizing-file-storage}

After you deploy your virtual machines with IBM VPC File Storage, you can adjust storage performance and capacity to meet changing workload requirements. The section covers how to modify IOPS, bandwidth, and storage capacity for existing file shares.

### Finding your file share in the IBM Cloud UI
{: #finding-file-share}

To adjust IOPS or bandwidth through the {{site.data.keyword.cloud_notm}} console, you need to locate your file share by using its ID. You can retrieve the file share ID from your PVC:

1. Get the PersistentVolume name from your PVC:

   ```bash
   oc get pvc <pvc-name> -n <namespace> -o jsonpath='{.spec.volumeName}'
   ```
   {: screen}

2. Get the file share ID from the PersistentVolume:

   ```bash
   oc get pv <pv-name> -o jsonpath='{.spec.csi.volumeHandle}' | cut -d: -f1
   ```
   {: screen}

3. Use this file share ID to locate your file share in the {{site.data.keyword.cloud_notm}} VPC console under **Storage** > **File Shares**.

### Adjusting IOPS for zonal file shares
{: #adjusting-iops}

For zonal file shares (dp2 profile), you can increase or decrease IOPS at any time to optimize performance and cost. IOPS adjustments are made through the IBM Cloud console, not from the {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster.

Important considerations:

- IOPS adjustments must be performed through the IBM Cloud VPC UI
- You can both increase and decrease IOPS values
- IOPS must remain within the valid range for your file share capacity (see [IOPS and Capacity Relationship for Zonal File Shares](#iops-capacity))
- No downtime is required - VMs can remain running during IOPS adjustments
- Changes take effect immediately after applying

To adjust IOPS for a zonal file share, see [Adjusting IOPS for file shares](/docs/vpc?topic=vpc-file-storage-adjusting-iops&interface=ui#adjust-vpc-iops-ui-file).

### Expanding file share capacity
{: #expanding-capacity}

You can expand the capacity of your file shares to accommodate growing data requirements. Unlike IOPS and bandwidth adjustments, capacity expansion is run by updating the PVC from your {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster.

Important considerations:

- Capacity expansion must be performed by editing the PVC in your {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster.
- Do not resize the file share directly from the IBM Cloud VPC UI. Always update the PVC to ensure proper synchronization.
- You can only expand storage capacity. Reducing capacity is not supported.
- No downtime is required. VMs can remain running during capacity expansion.
- The file share must have been created with `allowVolumeExpansion: true` in the StorageClass. All default IBM StorageClasses support this setting.
- Expansion is performed online without requiring VM restarts.

To expand file share capacity, see [Expanding File Storage for VPC volumes](/docs/openshift?topic=openshift-storage-file-vpc-apps#vpc-file-volume-expansion).

Example: Expanding a PVC

1. Edit the PVC to increase the storage size:

   ```bash
   oc edit pvc my-vm-disk
   ```
   {: screen}

2. Update the `spec.resources.requests.storage` field to the new size:

   ```yaml
   spec:
     resources:
       requests:
         storage: 100Gi  # Increased from 50Gi
   ```
   {: codeblock}

3. Save the changes. The CSI driver automatically expands the underlying file share to match the new PVC size.

4. Verify the expansion:

   ```bash
   oc get pvc my-vm-disk
   ```
   {: screen}

### Snapshots and data protection
{: #snapshots}

IBM VPC File Storage supports volume snapshots for data protection and recovery.

### Understanding file storage snapshots
{: #understanding-snapshots}

- Incremental snapshots. The first snapshot is a full backup; subsequent snapshots capture only changes.
- Backend storage. Snapshots are stored in the File Storage backend and do not use cluster storage.
- Snapshot frequency. Snapshots can be taken as often as needed.
- Retention. Snapshots persist according to the VolumeSnapshotClass retention policy.

When a file share is deleted, its snapshots are deleted automatically.
{: important}

### Taking VM snapshots
{: #take-vm-snapshots}

#### By using the web console
{: #snapshot-console}

1. In the navigation menu, select **Virtualization** > **virtual machines**.
2. Select your VM.
3. Click **Actions** > **Take snapshot**.
4. Provide a snapshot name.
5. Click **Save**.

You might see a warning that the `cloudinitdisk` is not included in the snapshot. This behavior is expected because cloud-init disks are ephemeral configuration drives that are regenerated at boot.
{: note}

Example warning message:

```bash
The following disk will not be included in the snapshot
cloudinitdisk - Snapshot is not supported for this volumeSource type [cloudinitdisk]
Edit the disk or contact your cluster admin for further details.
```

#### By using the CLI
{: #snapshot-cli}

Create a VirtualMachineSnapshot:

```yaml
apiVersion: snapshot.kubevirt.io/v1beta1
kind: VirtualMachineSnapshot
metadata:
  name: rhel9-vm-snapshot-1
  namespace: default
spec:
  source:
    apiGroup: kubevirt.io
    kind: VirtualMachine
    name: rhel9-vm
```
{: codeblock}

Apply the snapshot:

```bash
oc apply -f vm-snapshot.yaml
```
{: codeblock}

Check snapshot status:

```bash
oc get vmsnapshot rhel9-vm-snapshot-1 -n default
```
{: codeblock}

### Restoring from snapshots
{: #restore-snapshots}

To restore a VM from a snapshot, you must first create a bootable volume from the snapshot, then create a new VM by using that volume.

#### Step 1: Create a bootable volume from a snapshot
{: #create-boot-volume}

1. In the {{site.data.keyword.redhat_openshift_notm}} web console, go to **Virtualization** > **Catalog**
2. Select the **InstanceTypes** tab
3. Click **Add volume**
4. Configure the volume:

   - **Source type:** Use existing Volume snapshot
   - **Snapshot namespace:** Select the namespace that contains your snapshot
   - **Volume snapshot:** Select your snapshot
   - **Destination StorageClass:** Choose a StorageClass (which can be different from original)
   - **Size:** Adjust if needed (must be valid for the IOPS tier)
   - **Destination project:** Select `openshift-virtualization-os-images`

5. Click **Save**

#### Step 2: Create a VM from the bootable volume
{: #create-vm-from-boot-volume}

1. Browse to **Virtualization** > **Catalog** > **InstanceTypes**
2. Find your newly created boot volume
3. Click **Create virtual machine**
4. Configure VM settings
5. Click **Create**

#### Step 3: Verify and clean up
{: #verify-cleanup}

1. Verify that the VM starts successfully
2. Verify that your data is present
3. If successful, you can delete temporary resources (Bootable Volume)

### Snapshot best practices
{: #snapshot-best-practices}

- Regular snapshots. Take snapshots before major changes or updates.
- Retention policy. Use `retain` policy for critical VM snapshots.
- Testing. Regularly test snapshot restoration procedures.
- Documentation. Document snapshot schedules and retention policies.
- Monitoring. Monitor snapshot creation and storage consumption.


### File share replication
{: #file-share-replication}

{{site.data.keyword.cloud_notm}} File Shares on VPC support zone-to-zone replication for disaster recovery scenarios.

### Understanding replication
{: #understanding-replication}

- Replication schedule. Replicas are updated regularly based on your specified schedule, as often as every 15 minutes.
- Replication scope. Within the same VPC, across different zones.
- Replica states. Replicas are read only by default in the VPC.
- Visibility. Replications are not visible within the {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service cluster.

For more information, see [Replication and failover documentation](/docs/vpc?topic=vpc-file-storage-vpc-about&interface=ui#fs-repl-failover-overview).

### Using replicas for VM recovery
{: #replica-recovery}

To create a VM from a replica file share, follow these steps:

#### Step 1: Remove the replication relationship
{: #remove-replication}

Before you can use a replica, you must remove the replication relationship to make it writable.

1. In the {{site.data.keyword.cloud_notm}} console, select **VPC Infrastructure** > **File Shares**.
2. Locate your replica file share.
3. Follow the instructions to [remove the replication relationship](/docs/vpc?topic=vpc-file-storage-manage-replication&interface=ui#fs-remove-replication).

#### Step 2: Create a mount target
{: #create-mount-target}

Create a mount target for the replica file share by using security group access mode:

1. In the {{site.data.keyword.cloud_notm}} console, select your replica file share
2. Click **Create mount target**
3. Configure the mount target:
   - **Access mode** - Security group
   - **Security group** - Select `kube-<clusterID>` (your cluster's security group)
   - **VNI** - Select the appropriate virtual network interface
4. Click **Create**

For detailed instructions, refer to [Creating a mount target in the console](/docs/vpc?topic=vpc-file-storage-create&interface=ui#fs-create-mount-target-ui).

#### Step 3: Create a static PV and PVC
{: #create-static-pv-pvc}

Create a static PersistentVolume and PersistentVolumeClaim for the replica file share.

Example static PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: replica-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ibmc-vpc-file-500-iops
  csi:
    driver: vpc.file.csi.ibm.io
    volumeHandle: <file-share-id>:<mount-target-id>
    volumeAttributes:
      nfsServerPath: <nfs-server-path>
```
{: codeblock}

Example static PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: replica-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Gi
  storageClassName: ibmc-vpc-file-500-iops
  volumeName: replica-pv
```
{: codeblock}

For detailed instructions on creating static PV/PVC, refer to [Attaching existing file storage to an app](/docs/openshift?topic=openshift-storage-file-vpc-apps#vpc-add-file-static).

#### Step 4: Create a bootable volume from the static PVC
{: #create-boot-from-static}

1. Browse to **Virtualization** > **Catalog** > **InstanceTypes**
2. Click **Add volume**
3. Configure:
   - **Source type** - Use existing Volume
   - **Namespace** - Select a namespace that contains your static PVC
   - **Volume name** - Select your static PVC
   - **Destination StorageClass** - Choose the appropriate class
   - **Destination project** - `openshift-virtualization-os-images`
4. Click **Save**

#### Step 5: Create and verify the VM
{: #create-verify-vm-replica}

1. Create a VM from the bootable volume
2. Verify that the VM starts successfully
3. Verify that your data from the original file share is present.
4. If successful, clean up temporary resources (static PV/PVC and Bootable Volume).

### Backup solutions
{: #backup-solutions}

While IBM VPC File Storage provides snapshots for point-in-time recovery, comprehensive backup solutions offer more protection.

Veeam&reg; Kasten&trade; is validated to work with IBM VPC File Storage on {{site.data.keyword.redhat_openshift_notm}}:
Kubernetes-native data protection with {{site.data.keyword.redhat_openshift_notm}} Virtualization support and incremental snapshot capabilities for {{site.data.keyword.cloud_notm}} File Shares. See the [Veeam Kasten reference architecture](https://www.veeam.com/solution-briefs/veeam-kasten-and-red-hat-openshift-virtualization-reference-architecture_wp.pdf){: external}.

## Additional configuration scenarios
{: #additional-configuration-scenarios}

The following section describes other cluster-level and storage configuration tasks that you might apply when you use the IBM VPC File Storage provisioner with {{site.data.keyword.redhat_openshift_notm}} Virtualization. These tasks depend on your environment and goals. Such as preparing a new cluster, enabling required cluster services, or making IBM VPC File Storage the default storage option for the cluster or for {{site.data.keyword.redhat_openshift_notm}} Virtualization.

### Cluster installation
{: #optional-cluster-preparation}

If you are creating the cluster, see [Setting up your first cluster in your Virtual Private Cloud (VPC)](/docs/openshift?topic=openshift-vpc_rh_tutorial&interface=ui).

While you create the cluster, you can ensure that you complete the following tasks:

- Disable outbound traffic protection.
- Enable the default `CatalogSource`.

#### Disable outbound traffic protection
{: #disable-outbound-protection}

Outbound protection must be disabled to help ensure that the cluster has outbound internet access to pull required images from sources such as `registry.redhat.io` and `quay.io`.

```bash
ibmcloud oc vpc outbound-traffic-protection disable -c <cluster-name>
```
{: codeblock}

#### Enable default CatalogSource
{: #enable-catalogsource}

Enable the default CatalogSource in your cluster to install {{site.data.keyword.redhat_openshift_notm}} Virtualization from OperatorHub:

```bash
oc patch operatorhub cluster --type json \
  -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": false}]'
```
{: codeblock}

Verify the CatalogSource status:

```bash
oc -n openshift-marketplace get catalogsource
```
{: codeblock}

You can see catalog sources such as `redhat-operators` in a `READY` state.

#### Install Red Hat OpenShift Virtualization Operator
{: #install-virt-operator}

To install the {{site.data.keyword.redhat_openshift_notm}} Virtualization Operator see [{{site.data.keyword.redhat_openshift_notm}} Virtualization installation documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/virtualization/installing#installing-virt-operator_installing-virt){: external}.

### Default storage class reconfiguration
{: #default-storageclass-reconfiguration}

Use the following tasks only if you want IBM VPC File Storage to become the default storage option for the cluster or for {{site.data.keyword.redhat_openshift_notm}} Virtualization.

#### Step 1: Disable ODF as default, if applicable
{: #disable-odf-default}

If {{site.data.keyword.redhat_openshift_notm}} Data Foundation isn’t installed, or you don’t want to set ODF as default storage for {{site.data.keyword.redhat_openshift_notm}} Virtualization, skip this step, and proceed to Step 2.

If {{site.data.keyword.redhat_openshift_notm}} Data Foundation (ODF) is installed, it can set its own StorageClasses as the default for both the cluster and {{site.data.keyword.redhat_openshift_notm}} Virtualization. To use IBM VPC File Storage as the primary storage provider for virtual machine workloads, update the storage cluster configuration so that ODF no longer enforces those defaults.

If you do not change the default KubeVirt `StorageClass` from ODF to IBM VPC File Storage, new virtual machines can be provisioned by using the `HostAssistedClone` strategy instead of snapshot-based cloning.
{: important}

To disable the ODF StorageCluster default StorageClass behavior:

1. Edit the StorageCluster resource.

   ```bash
   oc edit storagecluster ocs-storagecluster -n openshift-storage
   ```
   {: screen}

2. Set both `defaultStorageClass` and `defaultVirtualizationStorageClass` to `false`:

   ```yaml
   spec:
     managedResources:
       cephBlockPools:
         defaultStorageClass: false
         defaultVirtualizationStorageClass: false
   ```
   {: codeblock}

3. Save and exit the editor.
4. Wait a few seconds for the changes to take effect.

#### Step 2: Set the cluster default storage class
{: #set-cluster-default}

As explained in [Understanding default storage class annotations](#default-storage-annotations), the cluster uses two different default StorageClass annotations for different purposes. First, configure the cluster-wide default for general workloads.

IBM recommends using `ibmc-vpc-file-500-iops` as the cluster default `StorageClass`. Edit the File Storage add-on configuration:

```bash
oc edit cm -n kube-system addon-vpc-file-csi-driver-configmap
```
{: codeblock}

Set the `SET_DEFAULT_STORAGE_CLASS` field:

```yaml
SET_DEFAULT_STORAGE_CLASS: "ibmc-vpc-file-500-iops"
```
{: codeblock}

Apply the cluster default annotation to the StorageClass:

```bash
oc patch storageclass ibmc-vpc-file-500-iops -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
{: codeblock}

This StorageClass uses the Delete reclaim policy, which automatically cleans up storage when you delete PVCs. This behavior is appropriate for general application workloads.

This setting applies to general cluster workloads. In Step 3, you configure a different StorageClass with a Retain reclaim policy as the {{site.data.keyword.redhat_openshift_notm}} Virtualization default.
{: note}

#### Step 3: Set the Red Hat OpenShift Virtualization default storage class
{: #set-virt-default}

Before you configure the default StorageClass for {{site.data.keyword.redhat_openshift_notm}} Virtualization, review the clone strategy options to determine which approach best fits your virtual machine provisioning requirements. For detailed guidance, see [Clone strategy considerations for IBM VPC File Storage](#clone-strategy-considerations-for-ibm-cloud-storage).

The following example configures `ibmc-vpc-file-retain-500-iops` as the {{site.data.keyword.redhat_openshift_notm}} Virtualization default `StorageClass`. This `StorageClass` uses a `Retain` reclaim policy, which is required for `CDI` boot volume management when you use snapshot-based cloning with `dataImportCronSourceFormat`.

Apply the virtualization default annotation:

```bash
oc patch storageclass ibmc-vpc-file-retain-500-iops -p '{"metadata":{"annotations":{"storageclass.kubevirt.io/is-default-virt-class":"true"}}}'
```
{: codeblock}

The system uses this StorageClass for all new virtual machine creations unless you select a different StorageClass during virtual machine creation.

Storage classes with a `Retain` reclaim policy require manual cleanup of `PersistentVolumes` after virtual machines are deleted. Plan for ongoing storage management to prevent unused volumes from accumulating.
{: important}

By default, {{site.data.keyword.redhat_openshift_notm}} Virtualization creates `InstanceType` templates in the `openshift-virtualization-os-images` namespace with a default size of 30 GiB. The `ibmc-vpc-file-retain-500-iops` `StorageClass` can provision 30 GiB file shares with valid IOPS for that capacity.

#### Step 4: Wait for boot volume images to migrate
{: #wait-for-datasources}

After you change the {{site.data.keyword.redhat_openshift_notm}} Virtualization default StorageClass, the system migrates boot volume images to the new target StorageClass. You must wait for all DataSources to be ready before you can create virtual machines.

1. Get the list of all DataSources:

   ```bash
   oc get datasource -A
   ```
   {: codeblock}

2. For detailed status information, describe a specific DataSource:

   ```bash
   oc describe datasource <datasource-name> -n openshift-virtualization-os-images
   ```
   {: codeblock}

3. Confirm that the DataSource status shows it is ready to be used:

   ```yaml
   status:
     conditions:
     - lastHeartbeatTime: "2026-06-10T01:59:24Z"
       lastTransitionTime: "2026-06-10T01:59:24Z"
       message: DataSource is ready to be consumed
       reason: Ready
       status: "True"
       type: Ready
   ```
   {: screen}

When all DataSources are ready, you can proceed to create virtual machines using the new default StorageClass.

### Optional virtual machine persistent state configuration
{: #optional-vm-persistent-state-configuration}

Use the following optional tasks only if your virtual machines require persistent state storage for features such as vTPM or UEFI firmware.

#### Configuring storage for VM persistent state
{: #vm-state-storage}

Virtual machines with certain features enabled require dedicated storage for persistent state data. Including virtual machines with vTPM or UEFI firmware. Standard {{site.data.keyword.cloud_notm}} File Storage StorageClasses are not suitable for virtual machine state storage, so you must create a custom StorageClass with specific configuration.

#### Understanding virtual machine persistent state storage
{: #vm-state-storage-overview}

The system uses virtual machine persistent state storage for the following components:

vTPM (virtual Trusted Platform Module)
:   Provides a virtual security chip that stores cryptographic keys and measurements. It is required for Windows 11 and Windows Server 2022 virtual machines. For more information, see [Running Windows 11 and 2022 Server virtual machines with {{site.data.keyword.redhat_openshift_notm}} persistent vTPM](https://www.redhat.com/en/blog/running-windows-11-and-2022-server-virtual-machines-red-hat-openshift-persistent-vtpm){: external}.

UEFI firmware variables:
:   Stores UEFI firmware configuration and boot settings for virtual machines that use UEFI firmware instead of legacy BIOS.

When you create a virtual machine with persistent state enabled, {{site.data.keyword.redhat_openshift_notm}} Virtualization creates two PVCs for that virtual machine:

Boot disk PVC
:   Stores the operating system disk and application data for the virtual machine. This PVC uses the standard virtual machine StorageClass that you select for the boot volume.

Persistent state PVC
:   Stores the virtual machine persistent state data that is required for features such as vTPM and UEFI firmware variables. This PVC uses the `vmStateStorageClass` that you configure in the `HyperConverged` custom resource.

The persistent state PVC has different requirements than the standard virtual machine boot disk PVC:

- It must use the `ext4` file system type instead of the default NFS behavior.
- It requires `ReadWriteOnce` access mode instead of `ReadWriteMany`.
- It needs volume expansion that is enabled.
- It is typically small in size.

#### Live migration with ReadWriteOnce storage
{: #vm-state-rwo-live-migration}

Although virtual machine persistent state storage requires `ReadWriteOnce` access mode, this requirement does not prevent live migration of virtual machines. KubeVirt and {{site.data.keyword.redhat_openshift_notm}} Virtualization handle the orchestration of ReadWriteOnce (RWO) PVCs during live migration through a coordinated handoff process:

1. Migration initiation - When a live migration starts, KubeVirt creates a new target pod on the destination node while the source pod continues running.
2. Memory transfer - The virtual machine memory state is transferred from the source pod to the target pod while the virtual machine continues running on the source node.
3. Coordinated handoff - After memory transfer is complete, KubeVirt runs a coordinated handoff:
   - The source virtual machine is paused.
   - The RWO PVC is detached from the source pod.
   - The PVC is attached immediately to the target pod on the destination node.
   - The target virtual machine resumes execution with the transferred memory state.
4. Cleanup - After successful migration, the source pod is terminated.

This orchestration helps help ensure that the RWO PVC is attached to only one pod at a time, while still enabling live migration. The brief pause during the handoff is typically measured in milliseconds.

For more information about live migration in {{site.data.keyword.redhat_openshift_notm}} Virtualization, see [Virtual machine live migration](https://docs.openshift.com/container-platform/latest/virt/live_migration/virt-about-live-migration.html){: external}.

#### VM state StorageClass requirements
{: #vm-state-storage-requirements}

The VM state StorageClass must have the following configuration:

File system type
:   `ext4`, specified in the `csi.storage.k8s.io/fstype` parameter.

Volume expansion
:   Enabled by setting `allowVolumeExpansion: true`.

Access mode
:   `ReadWriteOnce` in the StorageProfile.

Profile type
:   Uses `dp2`, which is a zonal profile.

Special parameter
:   `vmState: true` to identify as a VM state StorageClass.

#### Creating the VM state StorageClass
{: #create-vm-state-storageclass}

Create a custom StorageClass for virtual machine persistent state storage by using the following template:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ibmc-vpc-file-vm-state
  labels:
    app.kubernetes.io/name: ibm-vpc-file-csi-driver
  annotations:
    revision: '1'
    version: v2.0
provisioner: vpc.file.csi.ibm.io
volumeBindingMode: Immediate
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  - hard
  - nfsvers=4.1
  - sec=sys
parameters:
  csi.storage.k8s.io/fstype: ext4
  vmState: 'true'
  classVersion: '1'
  primaryIPID: ''
  encrypted: 'false'
  profile: dp2
  iops: '500'
  isENIEnabled: 'true'
  billingType: hourly
```
{: codeblock}

Key parameters:

`csi.storage.k8s.io/fstype: ext4`
:   Required. Specifies the `ext4` file system instead of the default NFS behavior.

`vmState: 'true'`
:   Required. Identifies this StorageClass as intended for virtual machine state storage.

`profile: dp2`
:   Uses zonal file shares for virtual machine state storage.

`allowVolumeExpansion: true`
:   Allows the PVC to be expanded if more space is needed.

Save this YAML to a file such as `vm-state-storageclass.yaml`, and create the `StorageClass`:

```bash
oc apply -f vm-state-storageclass.yaml
```
{: codeblock}

Verify that the StorageClass was created:

```bash
oc get storageclass ibmc-vpc-file-vm-state
```
{: codeblock}

#### Configuring the VM state StorageProfile
{: #configure-vm-state-storageprofile}

After you create the StorageClass, CDI automatically creates a corresponding StorageProfile. You must update this StorageProfile to configure the correct access mode and clone strategy.

Patch the StorageProfile with the following configuration:

```bash
oc patch storageprofile.cdi.kubevirt.io ibmc-vpc-file-vm-state --type=merge -p '{
  "metadata": {
    "annotations": {
      "cdi.kubevirt.io/minimumSupportedPvcSize": "10Gi"
    }
  },
  "spec": {
    "claimPropertySets": [
      {
        "accessModes": ["ReadWriteOnce"],
        "volumeMode": "Filesystem"
      }
    ],
    "cloneStrategy": "snapshot"
  }
}'
```
{: codeblock}

This configuration:

- Sets `accessModes` to `ReadWriteOnce`, which is required for virtual machine state storage.
- Sets `volumeMode` to `Filesystem`.
- Uses `snapshot` clone strategy for efficient cloning.
- Sets the minimum PVC size to `10Gi`.

Verify the StorageProfile configuration:

```bash
oc get storageprofile.cdi.kubevirt.io ibmc-vpc-file-vm-state -o yaml
```
{: codeblock}

Check that the output includes:

```yaml
metadata:
  annotations:
    cdi.kubevirt.io/minimumSupportedPvcSize: 10Gi
spec:
  claimPropertySets:
  - accessModes:
    - ReadWriteOnce
    volumeMode: Filesystem
  cloneStrategy: snapshot
```
{: codeblock}

#### Configuring Red Hat OpenShift Virtualization to use VM state storage
{: #configure-hyperconverged-vm-state}

After you create and configure the VM state StorageClass, configure {{site.data.keyword.redhat_openshift_notm}} Virtualization to use it for virtual machine persistent state storage. If you skip this configuration, {{site.data.keyword.redhat_openshift_notm}} Virtualization uses the cluster default StorageClass for virtual machine state storage, which does not meet the required settings.

Edit the HyperConverged custom resource:

```bash
oc edit hyperconverged kubevirt-hyperconverged -n openshift-cnv
```
{: codeblock}

Add or update the `vmStateStorageClass` field in the spec:

```yaml
spec:
  vmStateStorageClass: ibmc-vpc-file-vm-state
```
{: codeblock}

Save and exit the editor. {{site.data.keyword.redhat_openshift_notm}} Virtualization now uses this StorageClass for all virtual machine persistent state PVCs, including vTPM and UEFI firmware storage.

If `vmStateStorageClass` is not set, {{site.data.keyword.redhat_openshift_notm}} Virtualization uses the cluster default `StorageClass` for virtual machine state storage, which might not work correctly because of the different requirements.
{: note}

Verify the configuration:

```bash
oc get hyperconverged kubevirt-hyperconverged -n openshift-cnv -o jsonpath='{.spec.vmStateStorageClass}'
```
{: codeblock}

The output can show `ibmc-vpc-file-vm-state`.

### Configuring the host-assisted clone strategy
{: #config-host-assisted-clone}

If you need to use the host-assisted copy clone strategy instead of snapshot-based cloning, patch the `StorageProfile` to set `cloneStrategy` to `copy`:

```bash
oc patch storageprofile.cdi.kubevirt.io <storage-profile-name> --type=merge -p '{
  "spec": {
    "cloneStrategy": "copy"
  }
}'
```
{: codeblock}

Replace `<storage-profile-name>` with the name of the `StorageProfile` you want to configure (for example, `ibmc-vpc-file-3000-iops`).

## Monitoring and troubleshooting
{: #monitoring-troubleshooting}

Use this decision tree to identify the right troubleshooting section:

- PVC is not binding? → See [PVC stuck in pending](#pvc-pending).
- VM is not starting? → See [VM creation fails with clone error](#vm-clone-error).
- Can't access file share? → See [File share mount failures](#mount-failures).
- Slow VM performance? → See [Performance issues](#performance-issues).

### Monitoring file storage
{: #monitoring-storage}

#### Check PVC status
{: #check-pvc-status}

```bash
oc get pvc -A
```
{: codeblock}

Look for PVCs in `Bound` status. PVCs stuck in `Pending` indicate provisioning issues.

#### Check File Storage CSI pods
{: #check-csi-pods}

```bash
oc get pods -n kube-system | grep vpc-file-csi
```
{: codeblock}

All CSI pods need to be in the `Running` state.

#### View PVC events
{: #view-pvc-events}

```bash
oc describe pvc <pvc-name> -n <namespace>
```
{: codeblock}

Check the events section for error messages.

### Common issues and solutions
{: #common-issues}

#### PVC stuck in pending
{: #pvc-pending}

Symptoms:

- PVC remains in `Pending` state
- VM fails to start

Common causes:

1. Invalid IOPS and capacity combination

Error message:

```bash
failed to provision volume with StorageClass "ibmc-vpc-file-3000-iops":
rpc error: code = InvalidArgument desc = {RequestID: xxx,
BackendError: {Code:shares_profile_capacity_iops_invalid,
Description:The capacity or IOPS specified in the request is not valid for the 'dp2' profile}}
```
{: screen}

Solution: Delete the PVC and re-create it with a valid capacity or IOPS combination according to the [dp2 profile specifications](/docs/vpc?topic=vpc-file-storage-profiles&interface=ui#dp2-profile).

2. CSI driver not running

Solution: Check CSI driver pods and restart them if necessary:
```bash
oc get pods -n kube-system | grep vpc-file-csi
oc delete pod <csi-pod-name> -n kube-system
```
{: codeblock}

3. Resource group mismatch

Solution: If your cluster and VPC are in different resource groups, create a custom StorageClass with the VPC resource group ID.

#### VM creation fails with clone error
{: #vm-clone-error}


Symptoms:
- VM creation fails when you use a different StorageClass than the template.

- Error mentions snapshot or clone issues.

Cause: `StorageProfile` `cloneStrategy` is set to `snapshot` instead of `copy`.

Solution: Fix the StorageProfile as described in [Configuring storage profiles](#configure-storageprofiles).

#### File share mount failures
{: #mount-failures}

Symptoms:
- Pods cannot mount file shares.
- Mount errors in pod events.

Common causes:

1. Security group misconfiguration

Solution: Verify that the file share is in the correct security group (`kube-<clusterID>`).

2. Network connectivity issues

Solution: Verify that worker nodes can reach the file share mount target IP address.

#### Performance issues
{: #performance-issues}

Symptoms:
- Slow VM boot times.
- High I/O latency.

Solutions:

1. Increase IOPS tier. Move to a higher IOPS StorageClass.
2. Check file share utilization. Verify that the file share is not at capacity.
3. Review VM resource allocation. Ensure that VMs have adequate CPU and memory.

## Best practices and recommendations
{: #best-practices}

### Storage class selection
{: #storageclass-selection}

- Data disks. Match IOPS to workload requirements based on application needs.
- Development and test. Use lower IOPS tiers for cost-effective nonproduction workloads.
- Production. Use `Retain` reclaim policy to prevent accidental data loss.

### Capacity planning
{: #capacity-planning}

- Size VMs appropriately. Plan VM disk sizes based on workload requirements and application data needs.
- Validate IOPS. Always verify that capacity and IOPS combinations are valid for the dp2 profile. For more information, see [IOPS and capacity relationship for zonal file shares](#iops-capacity).
- Plan for growth. Account for snapshot storage and data growth over time.
- Optimize costs. Use appropriate IOPS tiers to balance performance and cost.

### High availability
{: #high-availability}

- Multi-zone deployment. Deploy worker nodes across multiple zones for high availability.
- WaitForFirstConsumer. Use this binding mode to help ensure that VMs and storage are colocated in the same zone.
- Replication. Consider file share replication for critical workloads. For more information, see [File share replication](#file-share-replication).
- Regular backups. Implement automated backup policies by using tools such as Veeam Kasten.

### Security
{: #security}

- Security groups. Verify that file shares are in the correct security group (`kube-<clusterID>`).
- Access control. Use Kubernetes RBAC to control PVC access and permissions.
- Encryption at rest. IBM VPC File Storage provides encryption at rest by default.
- Network isolation. Use VPC network ACLs for additional network security.

## Migration from VMware
{: #vmware-migration}

For organizations that migrate from VMware&reg; to {{site.data.keyword.redhat_openshift_notm}} Virtualization with IBM VPC File Storage:

### Storage comparison
{: #storage-comparison}

| VMware vSphere | {{site.data.keyword.redhat_openshift_notm}} Virtualization with File Storage |
| ---------------- | ------------------------------------------- |
| VMFS / vSAN | NFS-based IBM VPC File Storage |
| Storage policies | StorageClasses |
| vMotion | Live Migration (with ReadWriteMany) |
| Storage vMotion | Supported by using Migration Toolkit for Containers Operator |
| Snapshots | VolumeSnapshots |
| Linked Clones | Not supported by File Storage |
{: caption="Storage feature comparison between VMware vSphere and OpenShift Virtualization" caption-side="bottom"}

### Migration considerations
{: #migration-considerations}

- Storage type. IBM VPC File Storage provides NFS-based storage versus block storage in vSAN.
- Performance. Evaluate IOPS requirements and select appropriate StorageClasses.
- Snapshots. Plan snapshot strategies for data protection. For more information, see [Snapshots and data protection](#snapshots).
- Backup tools. Veeam Kasten provides VMware-like backup capabilities.
- Testing. Thoroughly test migrated workloads for performance and function.

## Summary
{: #summary}

IBM VPC File Storage provides a robust, NFS-based storage solution for {{site.data.keyword.redhat_openshift_notm}} Virtualization workloads on {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service. Key takeaways:

- Easy deployment. Installation by using the {{site.data.keyword.cloud_notm}} console or CLI add-on.
- Flexible performance. Multiple IOPS tiers to match workload requirements.
- Data protection. Built-in snapshots and replication capabilities.
- High availability. Multi-zone access with `ReadWriteMany` support.

## Additional resources
{: #additional-resources}

- [File Storage for VPC documentation](/docs/openshift?topic=openshift-storage-file-vpc-install)
- [{{site.data.keyword.redhat_openshift_notm}} Virtualization documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/virtualization/index){: external}
- [Storage class reference](/docs/openshift?topic=openshift-storage-file-vpc-sc-ref)
- [File Storage profiles](/docs/vpc?topic=vpc-file-storage-profiles&interface=ui#dp2-profile)
- [Troubleshooting File Storage](/docs/containers?topic=containers-ts-storage-vpc-file-eit-pvc-fails)
- [IBM Developer tutorial](https://developer.ibm.com/tutorials/openshift-virtualization-ibm-cloud/){: external}
