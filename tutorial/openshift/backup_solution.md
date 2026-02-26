---

copyright:
  years: 2026, 2026
lastupdated: "2026-02-26"
lasttested: "[{LAST_TESTED_DATE}]"

keywords: Red Hat OpenShift Virtualization, virtual servers, ROKS, VSI, File Storage, Backup, Kasten, Veeam, volumes

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 120m

---

{{site.data.keyword.attribute-definition-list}}

# IBM Cloud virtualization backup solutions
{: #virt-sol-openshift-backup}
{: #tutorial-backup-vms}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="120m"}

Use the following tutorial as a guide to install and configure a Veeam&reg; Kasten backup solution on an {{site.data.keyword.IBM}} {{site.data.keyword.redhat_openshift_full}} cluster.
{: shortdesc}



The tutorial uses Kasten version 8.5.1. However, the Kasten version that is available at the time of use might differ. New releases might include extra features or changes to the installation process and user interface.



## Veeam Kasten prerequisites
{: #virt-sol-openshift-backup-kasten-prereqs}


The following sections are an overview of the prerequisites of a Veeam Kasten installation.

### Before you begin the Veeam Kasten installation
{: #virt-sol-openshift-backup-kasten-before-installation}

Check that an IBM Red Hat OpenShift Container Platform cluster is deployed and configured with a Ceph&reg; RADOS block device StorageClass.

### Veeam Kasten prerequisite checker
{: #virt-sol-openshift-backup-kasten-prerequisite-checker-tool}

Veeam Kasten provides a prerequisite checker, Veeam Kasten Primer, that you use to verify that the Kubernetes cluster and StorageClasses meet the installation requirements. If a Container Storage Interface (CSI) provisioner exists, it conducts a basic validation check.

The download location includes a path to the version that you want to install. Version 8.5.1 is used in this example. Change these values to a specific version or use the value `latest`. The primer tool also expects the helm package to be installed on your system with the Veeam Kasten Helm charts repository. For more information, see [Veeam Kasten documentation](https://docs.kasten.io/latest/install/requirements/#install-prereqs){: external}.
{: note}

To start the Pre-Flight checker, run the following command after you create a command line session to the Red Hat OpenShift cluster:

```bash
curl https://docs.kasten.io/downloads/8.5.1/tools/k10_primer.sh | bash
```

For sample output for the checker, see [Veeam Kasten Primer for Pre-Flight Checks](https://docs.kasten.io/latest/operating/k10tools/#veeam-kasten-primer-for-pre-flight-checks){: external} link.
In general, you can expect a combination of `Success` and `OK` messages in the output. If errors occur, correct them before you install Veeam Kasten.

## Installing Veeam Kasten
{: #virt-sol-openshift-backup-kasten-install}

You can install Kasten by using Kubernetes OperatorHub or Helm.

A demo to install OperatorHub is available from the [Veeam overview](https://veeam.storylane.io/share/hfr76e40ea6y){: external}.

A Helm installation provides extra customization options, but the process is more elaborate.
For more information, see [Helm-based installation](https://docs.kasten.io/latest/install/openshift/helm){: external}.

### Installing Veeam Kasten through OperatorHub
{: #virt-sol-openshift-backup-kasten-install-operator}

Use the following information to install Veeam Kasten through OperatorHub.

1. From the Red Hat OpenShift Container Platform (Red Hat OpenShift Container Platform) web console, go to **Operators > OperatorHub**.
2. Search for **Kasten** in the filter and click the resulting **Veeam Kasten** entry. Installation packages are available with different licensing options. Click **Tile** for the appropriate licensing.
3. Click **Install**.
4. In the **Install operator**, you can use the default values. You can also enable the **Console plug-in**, which enables the **Veeam Kasten** tab of the Red Hat OpenShift web console navigation window.
5. Click **Install**.

### Configuring the environment for Veeam Kasten
{: #virt-sol-openshift-backup-kasten-conf}

Configuring the environment to use Veeam Kasten involves the use of the Red Hat OpenShift command line interface (CLI). Verify that the command line session to the Red Hat OpenShift cluster is active. If not, create a new command line session before you proceed.

If your storage class name differs from the example, then you must adjust the commands.
{: note}

Set the Red Hat OpenShift container storage class as the default for Kasten deployment. You can change the default storage class after you install Kasten.

1. From the Red Hat OpenShift Container Platform (Red Hat OpenShift Container Platform) web console go to **Storage > StorageClasses**
2. Search for the **ocs-storagecluster-ceph-rbd** entry. In the entry row, click **Vertical ellipsis** > **Set as default**.

Veeam Kasten uses annotations on StorageClass and VolumeSnapshotClass resources in Kubernetes to configure data protection behaviors and integrate with underlying storage providers. Use the Kubernetes CLI and the Red Hat OpenShift CLI to create the required annotations.

Run the following commands from the command line session to the Red Hat OpenShift cluster.

- `oc annotate volumesnapshotclass ocs-storagecluster-rbdplugin-snapclass k10.kasten.io/is-snapshot-class=true`
- `oc annotate storageclass ocs-storagecluster-ceph-rbd k10.kasten.io/sc-supports-block-mode-exports=true`
- `oc annotate storageclass ocs-storagecluster-ceph-rbd-virtualization k10.kasten.io/sc-supports-block-mode-exports=true`
- `oc annotate storageclass ocs-storagecluster-ceph-rbd k10.kasten.io/block-mode-export=true`
- `oc annotate storageclass ocs-storagecluster-ceph-rbd-virtualization k10.kasten.io/block-mode-export=true`

### Deploying Veeam Kasten
{: #virt-sol-openshift-backup-kasten-deploy}

After you install Veeam Kasten and annotate StorageClasses, complete the configuration by deploying and configuring more Veeam Kasten pods into the environment.

1. From the Red Hat OpenShift Container Platform web console, go to **Operators > Installed Operators > Kasten** and verify that Project: **All projects** or **kasten-io** is selected.
2. Click **Veeam Kasten**.
3. Click the **K10** tab in the Veeam Kasten Operator.
4. Click **Create K10**.
5. In the **Create K10** page, enable the following settings:
   1. Enable basic authentication.
   2. Enable the **K10 dashboard** to display through the route.
   3. Create secured edge route for the displayed K10.
6. In the **Specify StorageClassName to be used for PVCs** field, enter `ocs-storagecluster-ceph-rbd`.
7. Click **Create**.
8. Go to **Workload > Pods**. View the **Project: kasten-io** and wait for the pods to show **Running**.

### Accessing the Veeam Kasten web interface
{: #virt-sol-openshift-backup-kasten-web}

1. Verify that **Project: kasten-io** is selected.
2. From the web console, go to **Networking > Routes**.
3. Locate the **K10-route** entry and click the link in **Location** value. The path for Kasten UI is `/K10/#/dashboard`. For example, `https://<openshift-console-fqdn>/k10/#/dashboard`.
4. Log in to Kasten by using the Red Hat OpenShift authentication token. You can use the web console or CLI. For more information about authentication, see the [Authentication](https://docs.kasten.io/latest/access/authentication/){: external}.
5. From the Red Hat OpenShift Container Platform web console, click the dropdown menu in the page header that is next to the **IAM username**.

Run the Veeam Kasten primer tool by using the following command.

```bash
% curl https://docs.kasten.io/downloads/8.5.1/tools/k10_primer.sh | bash /dev/stdin blockmount -s ${STORAGE_CLASS_NAME}
```

The Veeam Kasten primer tool can also be used to verify that the CSI StorageClass has the required configuration to support block VolumeMode exports in Veeam Kasten by running the following command. For more information, see [Veeam Kasten Primer for storage integration checks](https://docs.kasten.io/latest/operating/k10tools/#primer_storage_int_checks){: external}.

```bash
% curl https://docs.kasten.io/downloads/8.5.1/tools/k10_primer.sh | bash /dev/stdin csi -s ${STORAGE_CLASS_NAME}
```

The two checks that are described in the previous section can be run against both storage classes for Kasten use, in this example, `ocs-storagecluster-ceph-rbd` and `ocs-storagecluster-ceph-rbd-virtualization`. If errors occur, you need to correct them before Veeam Kasten can protect your data.




## Applying for a Veeam Kasten license
{: #virt-sol-openshift-backup-kasten-license}

Veeam Kasten comes with an embedded, without charge edition license, which allows nonproduction use without support for up to five worker nodes. This license is good for proof-of-concept work.

For production use, a Veeam Kasten enterprise license is recommended, which you can obtain through Veeam. You can apply licenses by using the web interface or by using the Kubernetes command line `kubectl`. Both options are explained in detail in [Advanced installation options](https://docs.kasten.io/latest/install/advanced){: external}.

## Creating a data protection location profile
{: #virt-sol-openshift-backup-kasten-location}

In Veeam Kasten, location profiles define where backup data is stored or from where the backup data is restored. Supported storage targets typically include S3-compatible object storage, Network File System (NFS) share, or Veeam Scale-Out Backup Repository (SOBR).

For details on location configuration options, see [Veeam Kasten location configuration documentation](https://docs.kasten.io/latest/usage/configuration/#locked-bucket-setup){: external}.

### Creating a S3-compatible location profile
{: #virt-sol-openshift-backup-kasten-location-s3}

Before you create an IBM Cloud Object Storage bucket location profile, you have the following prerequisites:

- IBM Cloud Object Storage bucket that is already defined with a networking path for access from Veeam Kasten on Red Hat OpenShift
- S3 access key
- S3 secret
- IBM Cloud Object Storage bucket endpoint URL
- Bucket region

Enable the object-locking option on the IBM Cloud Object Storage bucket. This option is required for immutable backups.
{: note}

1. From the Veeam Kasten web interface, go to **Profiles > Location** and click **Create profile**.
1. In the **Name and Provider** page, enter the `Location Profile Name` and in the **Storage Provider** dropdown menu, select **S3 Compatible**
1. Click **Next** and in the **Configure S3 Generic Profile** page, enter the following information:
   - S3 access key
   - S3 secret
   - Endpoint
   - Choose whether to **Skip certificate chain and hostname verification**
   - Region
   - Bucket name
   - Enable immutable backups, if applicable. Click **Validate bucket**, if enabled. If immutable backup is enabled, set the **Protection period**. The minimum protection period is 1 day and the maximum is 90 days.
1. Click **Next**
1. Review the **Summary** and click **Submit**.

### Creating an NFS location profile
{: #virt-sol-openshift-backup-kasten-location-nfs}

Creating an NFS location profile requires the following prerequisites:

- An existing NFS share with a networking path for access from Veeam Kasten on Red Hat OpenShift.
- The name of the PersistentVolumeClaim (PVC) mounting the NFS export location. For setup instructions, see [Veeam Kasten documentation](https://docs.kasten.io/latest/usage/configuration/#filestore-location-profile){: external}.
- A path suffix to use inside the NFS export path (optional unless supplemental groups are set).
- A supplemental group that has access to the path (optional).

Use the following steps to create an NFS location profile.

1. From the Veeam Kasten web interface, go to **Profiles > Location** and click **Create profile**.
1. In the **Name and provider** page, enter the `Location profile name` and in the **Storage provider** dropdown menu select **NFS/SMB**.
1. Click **Next**
1. In the **Configure NFS/SMB profile** page, enter the following information:
   1. Claim name
   2. Path (optional unless you specify a supplemental group)
   3. Supplemental group (optional)
1. Click **Next**
1. Review the **Summary** and click **Submit**.

### Creating a Veeam Scale-Out Backup Repository location profile
{: #virt-sol-openshift-backup-kasten-location-vbr}

Creating a Veeam Scale-Out Backup Repository (SOBR) location profile requires the following prerequisites:

- A Veeam SOBR that is configured with a networking path for access from Veeam Kasten on Red Hat OpenShift
- Veeam Backup & Replication (VBR) server name and location details
- Access to credentials with adequate permissions on the VBR
- In the Veeam VBR console, confirm that access permissions in the repository are configured to allow Veeam Kasten access.

- On the Backup Infrastructure page, select your **SOBR** under Scale-Out Repositories and select **Access permissions**.
- To make the setup simple, select **Allow** for everyone

For a secure setup, configure access by using only a service user.
{: tip}

- In the VPC that hosts Kasten, confirm that the relevant security group allows traffic between the VBR and Kasten.

Use the following steps to create a Veeam Scale-Out Backup Repository location profile.

1. From the Veeam Kasten web interface, go to **Profiles > Location**.
1. Click **Create New Profile**
1. In the **Name and Provider** page, enter the `Location profile name` and in the **Storage provider** menu select **Veeam Repository**.
1. Click **Next**
1. In the **Configure Veeam Profile** page, enter the following information:
   - The Veeam backup server
   - TVeeam Backup Server API port
   - Backup repository
   - Username
   - Password
   - Choose whether to **Skip certificate chain and hostname verification**.
1. Click **Next**
1. Review the **Summary** and click **Submit**.

## Creating policies for data protection
{: #virt-sol-openshift-backup-kasten-policies}

To protect applications and data, you must understand and plan for the following basic concepts:

- The requirements for snapshots and exports
- Scheduling snapshots and exports to meet the recovery goals and timelines
- Selecting the correct applications and identifying resources with finer-grained protection requirements
- Backup export location (optional but recommended)
- You can apply actions before or after snapshots (optional)

Two types of policies are available.

- Snapshot: These policies typically store storage snapshots and metadata both on and off the storage cluster by using defined location profiles.
- Import: These policies require an import action to bring in restore points and a restore action that defines how to process the restore points.

Kasten allows the policies to be run on a user-defined schedule or can be run on demand. Users also define how long to retain the data before it is removed from the inventory. Retention can be defined in days, weeks, months, or years. Depending on the type of data, your retention plan depends on business needs or business security policy.

You need to configure the following retention settings:

- Snapshot retention that is related to storage snapshots.
- Retention of exported snapshots that is for snapshots that are exported to a backup target location.

A best practice is to set a storage snapshot retention at a minimal level as possible and use the export retention for backup retention. If an export retention is not set, the snapshot retention is used for both values.

Kasten provides the option to export backup data to an external location by using location profiles by toggling the **Enable backups through snapshots exports** option. After the option is enabled, you can select export frequency, export location profile, and retention of exported snapshots.

It is recommended that you enable the backup export option. Because having only a single backup copy (and local) is not a best practice, it doesn't satisfy the industry standard 3-2-1 backup rule.
{: tip}

Veeam Kasten encryption is always enabled for data and metadata for object storage and NFS/SMB files storage. Kasten uses the AES-256-GCM encryption algorithm.

You can apply Kasten policies at a Red Hat OpenShift project level, which works at a Kubernetes namespace level that includes applications, metadata, and PVCs. You can apply them based on processing labels that different asset types use. Labels can be used to include or exclude assets from a policy.

Lastly, Kasten provides option to run pre-snapshot and post-snapshot hooks (also known as blueprint actions) that you can apply before or after the snapshot completes. The hooks are useful and must be considered for application-aware backups.

For more information about blueprints, see the [FAQ](#virt-sol-openshift-backup-kasten-faq). 

The option to make protected data immutable is set in the location profile, not in the policy. If immutability is a requirement, you must configure it when you create location profiles.
{: note}

### Creating a data protection policy in the Veeam Kasten web UI
{: #virt-sol-openshift-backup-kasten-policies-create}

The policy that is used in a data protection backup action is determined by the following questions:

- What data is backed up?
- How is the data backed up?
- Where is the data is copied?
- When does the policy run?
- How long is the data stored?

Every data protection restore option must be tested frequently to verify that the requirements are being met.

After the requirements are defined, you can create a data protection policy by following these steps:

1. From the Veeam Kasten web interface, go to **Policies** > **Policies**.
1. Click **Create a policy**.
1. Provide a **Name** for the policy. Optionally, you can add **Comments** to identify, describe, or share helpful information about the policy.
1. In the **Action** entry, choose **Snapshot** and define the following options.
   1. Use the preset option (optional).
       - Enabled: select a pre-defined schedule and export settings from the settings menu.
       - Disabled: define custom schedule and export settings in the **Backup frequency** section.
   1. Set the back up frequency (if the preset option is disabled).
       1. Select the back up schedule (hourly, daily, weekly, monthly, yearly, or on demand).
   1. Define the advanced frequency options.
       1. Enabled:
          - Select **Use local** or **UTC time**.
          - Select the **Hours of the day** for daily snapshots.
       1. Disabled: nothing to set
   1. Select a back up window. If the backup creation is not finished within the selected interval, the back up cancels.
      1. Enabled:
         - Use staggering:
            - Enabled: You can stagger when policies run to help reduce peak loads on the system.
            - Disabled: Staggering doesn't run.
         - Define the **From** and **To** time for the backup interval.
   1. Snapshot retention: You can customize the snapshot retention schedule. Use the user interface to set daily, weekly, monthly, and yearly snapshots.
   1. Enable **Backups** through **Snapshot exports** (optional but recommended)
      1. Enabled:
         - Set an export frequency of daily, weekly, monthly, or yearly.
         - Export location profile: You can create or select an existing location profile.
         - Retention of exported snapshots: You define or use an existing one.
         - Storage class exceptions are used to define exceptions for specific storage classes
      1. Disabled: nothing to define.
   1. Selection Type: Namespaces or virtual servers
      1. Namespaces:
         - Select applications: if **namespaces** is selected you must select applications.
         - Snapshot cluster-scoped resources (optional)
            - All cluster-scoped resources
            - Filter cluster-scoped resources
               - Include filters - based inclusion rules, define Group, Version, Resource, Name, or Label.
               - Exclude filters - based exclusion rules, define Group, Version, Resource, Name, or Label.
      1. Virtual servers:
         1. Specify virtual servers: if virtual servers are selected, you must specify the virtual servers by selecting one or more namespaces.
         1. Select **VM resources** to create filters that include and exclude resources that are dependencies for each virtual server.
1. Advanced settings (optional)
   1. Pre and post-snapshot action hooks: These actions hooks are blueprint actions that you can apply before or after a snapshot success, or after a snapshot failure.
   1. Location profile for Kanister actions: Specify a location profile for Kanister Blueprint.
   1. Location profile for container images: If the application snapshot includes `ImageStreams` specify a container image location profile.
   1. Pre and post-export action hooks: These action hooks are blueprint actions that you can apply before or after an export success, or after an export failure.
   1. Ignore exceptions and continue if possible. This action is useful in environments when applications are in an unhealthy state, but the policy actions can continue.

## Restoring a namespace or virtual server
{: #virt-sol-openshift-backup-kasten-policies-restore}

A restore is run when there you need to replace or compare the existing application to a previous application backup. Use the following steps to restore a namespace or virtual server.

1. From the Veeam Kasten web interface, go to **Restore points**
1. Select the application type and whether you want to restore a namespace or virtual server.
1. Find the appropriate restore point from the list.
1. In the entry row, click **Vertical ellipsis** and click **Restore**.
   1. Application name: the restore can run in the originating namespace or a different namespace. Whichever name is selected, that namespace is overwritten. Review the namespace carefully.
   1. Optional restore settings:
      1. Pre and post-snapshot action hooks: These action hooks are blueprint actions that you can apply before or after a restore success, or after a restore failure.
      1. Volume-clones restore: Provides mountable copies of backup data that you can use to retrieve individual files without disrupting workloads.
      1. Data-only restoration: Application data must exist. Restore doesn't overwrite workload manifests.
      1. Alternative location profile: is used to copy a profile to a different location profile.
      1. Overwrite existing (optional): Workloads are always restored. ServiceAccounts and nonnamespaced resources (such as StorageClass) are restored only when they are missing and is best used to overwrite nonworkload resources.
      1. Preserve virtual server MAC addresses: You can restore the original MAC address for all virtual servers that are in the application.
      1. Apply transforms to restored resources: If selected, specify a transform. This specification is useful when you move an application to a different environment.
   1. Artifacts: You can specify which restore point artifacts are restored. The default is all artifacts.
   1. Snapshot: Select from existing snapshots.
   1. Specs: You select the artifacts that you want to restore. By default, all specs are selected.
   1. Kanister: Veeam Kasten defaults to volume snapshot operations. Kanister uses blueprints to define database-specific workflows and open source blueprints are available for several popular applications.

## Running a file-level restore
{: #virt-sol-openshift-backup-kasten-flr}

Veeam Kasten provides the FileRecoverySession Custom Resource (CR) to request network access to files from exported restore points without requiring full restore of volume data.

For more information, see [Restoring Individual Files](https://docs.kasten.io/latest/usage/restorefiles){: external}.

FileRecoverySession doesn't support backups that are exported to Veeam Scale-Out Backup Repository, which can be done through Veeam Backup & Restore (VBR) server. See [Veeam documentation](https://helpcenter.veeam.com/docs/vbr/kasten_integration/guest_file_recovery.html?ver=13){: external} on limitations and [Veeam KB4759](https://www.veeam.com/kb4759){: external} on step-by-step procedure.

### Virtual servers that were migrated by using the Red Hat Migration Toolkit for Virtualization
{: #virt-sol-openshift-backup-kasten-limitations-mtv}

If you use Kasten to back up and restore virtual servers, some issues can occur. These issues occur for virtual servers that are migrated by using the Red Hat Migration Toolkit for Virtualization, if the migration plan is not archived and deleted after the migration. These issues can also occur because of references to storage resources that are not released after the migration. It is documented in [PVCs are stuck in a stopping state](https://access.redhat.com/solutions/7018107) {: external}

### The Federal Information Processing Standard (FIPS)
{: #virt-sol-openshift-backup-kasten-limitations-fips}

You can deploy Kasten in a FIPS-compliant mode through only the helm chart with the correct parameters selected. Because components that are connected to Kasten might not all be FIPS-compliant, carefully configure FIPS mode to reduce the risk of features that might not work.

For more information, see [Installing Kasten in FIPS mode](https://docs.kasten.io/latest/install/fips/){: external}.

### Veeam Backup and Replication (VBR) Repository backup
{: #virt-sol-openshift-backup-kasten-limitations-vbr-backup}

Backups that are exported to VBR repositories might have limitations.

The VBR repository can store only the snapshot data itself and not the metadata that is a part of a backed-up application in Red Hat OpenShift.

When you configure a backup policy that exports to a VBR repository, you also need to select a secondary export location like NFS or an object storage bucket. This means that to successfully restore from a VBR repository, Veeam Kasten needs to pull the snapshot data from the VBR repository and the metadata from the other export location.

If immutability is required, you must configure immutability on both the VBR repository and the secondary immutable object storage bucket. The immutability period that is configured for the object storage bucket location in Veeam Kasten must be less than or equal to the immutability period that is defined in the VBR repository.

For more information, see [VBR Considerations](https://docs.kasten.io/latest/install/storage/#considerations){: external}.

## Veeam Kasten best practices
{: #virt-sol-openshift-backup-kasten-best-practices}

Review [Veeam Kasten Best Practices](https://docs.kasten.io/latest/references/best-practices/){: external} for the most recent release, and verify that the guidance supports your implementation details and use case.

### Recommendations to follow the 3-2-1 backup rule
{: #virt-sol-openshift-backup-kasten-best-practices-3-2-1}

See the following best practices to follow the 3-2-1 backup rule.

- Retain three copies of the data. These copies can be original data in the persistent volume claim (PVC), the snapshot data, and the snapshot data that is exported to an external location.

- Storing the copies on two different media. Veeam Kasten supports multiple storage infrastructures, including block, file, and object storage. For example, you store your data on PVC and external S3. Having two copies of data, one on PVC and the other as a local snapshot in the Kubernetes cluster, does not count as two different media. In a disaster, the cluster is beyond recoverable and both copies of data are lost.

- Keep one backup copy off-site. Set up backup copy jobs to transfer your backups offsite to another location, such as a public cloud provider or auxiliary storage in a separate site. Exporting backups to a locally hosted profile through NFS file storage is not considered offsite.

You can follow the 3-2-1 rule in several ways. The simplest is to export your backup data to an IBM Cloud Object Storage bucket. You can also use Veeam SOBR for enhanced 3-2-1 rule. You have the same backup copy on performance tier and capacity tier.

### Immutability
{: #virt-sol-openshift-backup-kasten-best-practices-immutability}

It is recommended that you enable immutability wherever possible to protect against ransomware attacks. You can configure immutability on S3-compatible buckets and on Veeam SOBR when you use a Linux Hardened Repository.

When you use immutable buckets as an export location, Kasten adds a hidden buffer period to the retention period. This buffer affects the actual immutability duration. According to Kasten documentation, the following points apply when you create an S3 immutable bucket profile:

- The maximum protection period is 90 days.
- A safety buffer is automatically added to the selected protection period to help safeguard ongoing protection for new objects that are written to the bucket before their retention expires.
- The minimum protection period is 1 day.

For more information, see [Setting Up Immutable Backup Protection](https://docs.kasten.io/latest/usage/immutable/#setup_immutable){: external}.

### Use Kasten Disaster Recovery (DR) to backup configurations
{: #virt-sol-openshift-backup-kasten-best-practices-kasten-dr}

Kasten can help back up and restore Kasten configuration through Kasten DR feature, which can be configured as part of best practices to recover your Kasten instance from any unintended actions.

Kasten cannot be recovered without a few pieces of information that enable an administrator to restore the Kasten environment.

- Depending on how Kasten DR is configured, you need the passphrase information that was used to set up Kasten DR.
- One or more export locations where the DR backups were exported.
- Cluster ID, an auto-generated ID when Kasten DR is set up.

Make a note of the value shown in the Kasten DR tab on the Kasten UI for successful recovery.
{: tip}

For more information, see [Kasten DR documentation](https://docs.kasten.io/latest/operating/dr/#dr-enable){: external}.


## Next steps
{: #virt-sol-openshift-backup-next-steps}

After you configure Veeam Kasten for back up and recovery, consider these next steps:

- **Explore migration**: Learn how to [migrate workloads from VMWARE to Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-vsphere-openshift-migration)
- **Review architecture**: Understand the [Red Hat OpenShift Virtualization reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-rove-architecture)
- **Design considerations**: Explore [resiliency design patterns](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-resiliency-design) for your environment
- **Veeam documentation**: Access the complete [Veeam Kasten documentation](https://docs.kasten.io/latest/){: external} for advanced features
- **Veeam documentation**: Access the [Veeam Kasten documentation](https://docs.kasten.io/latest/){: external} for advanced features
