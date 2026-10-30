---

copyright:
   years: 2026
lastupdated: "2026-10-29"

keywords: Red Hat Advanced Cluster Management Hub, IBM Cloud, IBM Cloud Object Storage, Red Hat OpenShift Virtualization, RHACM, OADP, Velero, MultiClusterHub recovery

subcollection: virtualization-solutions

content-type: howto

---

{{site.data.keyword.attribute-definition-list}}

# Backing up and restoring the Red Hat Advanced Cluster Management Hub with IBM Cloud Object Storage
{: #acm-hub-backup-and-restore}

Red Hat&reg; Advanced Cluster Management (RHACM) provides the management plane for a fleet of managed {{site.data.keyword.redhat_openshift_notm}} clusters, including clusters that run {{site.data.keyword.redhat_openshift_notm}} Virtualization. Protecting the RHACM Hub helps preserve cluster-management configuration, policies, placements, application metadata, and the information that is required to reconnect managed clusters after a Hub failure.
{: shortdesc}

Back up and restore RHACM Hub management-plane resources by using the RHACM `cluster-backup` component, Red Hat OpenShift APIs for Data Protection (OADP), Velero, and {{site.data.keyword.cos_full_notm}} (COS).

Scope: RHACM Hub backup protects RHACM management-plane resources. It does not replace workload-level backup or disaster recovery for virtual machine disks, persistent volumes, applications, or data that runs on managed clusters.
{: note}

## What is covered in the guide
{: #acm-hub-backup-what-covered}

- Enabling RHACM `cluster-backup` and the OADP infrastructure that it manages.
- Configuring {{site.data.keyword.cos_short}} as the backup repository.
- Selecting COS resiliency that matches the recovery objective.
- Configuring the `DataProtectionApplication` (DPA).
- Enabling Managed Service Account support for automatic managed-cluster reconnection.
- Creating and validating an RHACM `BackupSchedule`.
- Validating RHACM backup outputs and repository access.
- Recovering selected RHACM resources on the same Hub by using the RHACM-native `Restore` resource.
- Restoring RHACM Hub metadata to a replacement Hub.
- Handling both an unavailable original Hub and a planned recovery test where the original Hub remains available.
- Validating Managed Service Account and `auto-import-account` prerequisites.
- Troubleshooting managed clusters that remain in `Pending Import`.
- Establishing the replacement Hub as the active Hub and safely resuming scheduled backups.


## Guide applicability
{: #acm-hub-backup-when-to-use}

The following table describes when to use each set of steps.

| Situation | Applicability and Usage |
| --- | --- |
| Initial RHACM Hub backup setup | Follow steps 1–8 |
| Validate an existing RHACM backup configuration | Follow steps 5–8 |
| Recover selected RHACM resources on the same Hub | Follow step 9 |
| Recover RHACM on a replacement Hub | Follow step 10 |
| Back up managed-cluster workloads or persistent application data | No - use an appropriate workload-level backup or disaster recovery (DR) solution |
{: caption="When is the guide applicable" caption-side="bottom"}


## Before you begin
{: #acm-hub-backup-prereqs}

Before you configure RHACM Hub backup and restore, confirm the following prerequisites:

- You have `cluster-admin` access to the RHACM Hub and can use the `oc` CLI.
- The RHACM Hub is working correctly before you create the recovery baseline.
- The `cluster-backup` component is supported by the RHACM version that is installed in your environment.
- You have a {{site.data.keyword.cos_short}} instance and permission to create or use a bucket and {{site.data.keyword.cloud_notm}} Identity and Access Management (IAM) Service ID credentials.
- You understand the recovery objective for the backup repository:
    - For recovery from a complete regional failure, use a Cross Region COS bucket or another design that keeps the backup repository available outside the failed region. This might be bucket replication to a recovery region.
    - A Regional COS bucket provides resilience across zones within one region, but it should not be treated as independently available from that region during a complete regional outage.
    - If you plan to restore to a replacement Hub, record the following configuration from the source Hub:
    - RHACM operator version and installation namespace.
    - `MultiClusterHub` configuration.
    - `MultiClusterEngine` configuration.
    - OADP configuration and version.
    - Additional operators that RHACM resources depend on, such as Red Hat OpenShift GitOps, cert-manager, or other extensions.

The replacement Hub must initially use the same RHACM operator version as the Hub that created the backup. If you want to use a newer RHACM version, restore the backup first with the matching version, then upgrade the restored Hub.

The command examples align with RHACM 2.16 behavior. Verify fields and supported workflows against the RHACM version that is installed in your environment.
{: note}

## Step 1: Enable RHACM cluster backup
{: #acm-hub-backup-step1-enable}

Enable the RHACM `cluster-backup` component. For a standard installation where AWS Security Token Service (STS) is not enabled, the backup component automatically installs the required OADP Operator in the `open-cluster-management-backup` namespace.

Do not manually install a second OADP Operator as part of the default backup setup. OADP and Velero custom resource definitions are cluster-scoped, and incompatible duplicate OADP versions can cause an unsupported or conflicting configuration.

Edit the `MultiClusterHub` resource:

```bash
oc edit multiclusterhub multiclusterhub -n open-cluster-management
```
{: pre}

Find the `cluster-backup` component and set `enabled` to `true`:

```yaml
- configOverrides: {}
  enabled: true
  name: cluster-backup
```
{: codeblock}

Verify the backup namespace and components:

```bash
oc get namespace open-cluster-management-backup
oc get pods -n open-cluster-management-backup
```
{: pre}

Verify the OADP installation that RHACM uses:

```bash
oc get csv -n open-cluster-management-backup
oc get pods -n open-cluster-management-backup
```
{: pre}

Expected result:

- `open-cluster-management-backup` exists.
- RHACM backup controllers are running.
- The OADP Operator that is required by RHACM backup is installed in `open-cluster-management-backup`.

If the Hub requires a manually installed OADP Operator, such as in a supported STS scenario, follow the Red Hat procedure for that environment and help ensure the installed OADP version is compatible with the RHACM backup component. Do not install multiple incompatible OADP versions on the same cluster.
{: important}

## Step 2: Create and authorize the COS backup repository
{: #acm-hub-backup-step2-cos-repo}

Create or select the {{site.data.keyword.cos_short}} bucket that stores RHACM backups.

1. In the {{site.data.keyword.cloud_notm}} console, open **Cloud Object Storage**.
2. Create or select a {{site.data.keyword.cos_short}} instance.
3. Create or select a bucket whose resiliency matches the recovery objective.
4. Record the bucket name, location, and the S3-compatible endpoint you plan to use.
5. Create a Service ID credential with **Include HMAC Credential** enabled.
6. Grant the Service ID the required {{site.data.keyword.cloud_notm}} IAM access to list, read, write, and delete objects in the bucket. Use bucket-scoped Writer access or an equivalent policy that provides the required permissions.

Example configuration values:

```text
Bucket:   <ACM_BACKUP_BUCKET>
Location: <COS_LOCATION>
Endpoint: <COS_S3_ENDPOINT>
```
{: screen}

Examples of endpoint patterns include:

```text
Regional:     https://s3.us-south.cloud-object-storage.appdomain.cloud
Cross Region: https://s3.us.cloud-object-storage.appdomain.cloud
```
{: screen}

Hash-based Message Authentication Code (HMAC) credentials alone are not sufficient. The Service ID must also have {{site.data.keyword.cloud_notm}} IAM authorization to access the bucket. Missing authorization can result in `403 AccessDenied (ListObjectsV2)` and an unavailable `BackupStorageLocation`.
{: important}

The RHACM restore workflow depends on access to the backup repository. For recovery from a complete regional failure, choose a {{site.data.keyword.cos_short}} resiliency model or replication design that keeps the repository reachable from the recovery environment.
{: note}

## Step 3: Create the COS credentials secret
{: #acm-hub-backup-step3-credentials-secret}

Create a local credentials file by using the COS HMAC credentials. Restrict access to the file while it exists:

```bash
umask 077
vi credentials-velero
```
{: pre}

Add the credentials:

```text
[default]
aws_access_key_id=<YOUR_ACCESS_KEY_ID>
aws_secret_access_key=<YOUR_SECRET_ACCESS_KEY>
```
{: screen}

Create the Kubernetes&reg; `Secret` in the RHACM backup namespace:

```bash
oc create secret generic cloud-credentials \
  --namespace open-cluster-management-backup \
  --from-file=cloud=credentials-velero
```
{: pre}

Verify that the `Secret` exists:

```bash
oc get secret cloud-credentials -n open-cluster-management-backup
```
{: pre}

Expected result: an `Opaque` `Secret` named `cloud-credentials` exists in `open-cluster-management-backup`.

After the `Secret` is created and verified, delete the local plain text credentials file:

```bash
rm -f credentials-velero
```
{: pre}

Do not place HMAC credentials in source control, terminal history, screenshots, tickets, or documentation. Rotate the HMAC credential according to your organization security policy and verify backup access after rotation.
{: important}

## Step 4: Configure the DataProtectionApplication
{: #acm-hub-backup-step4-dpa}

Create a `DataProtectionApplication` (DPA). The DPA manages the Velero deployment and `BackupStorageLocation` that RHACM Hub backup uses.

These steps cover RHACM management-plane resources only and do not back up persistent workload volumes. Therefore, the sample configures object backup storage only and does not add a volume snapshot location or Node Agent configuration.

Create `dpa.yaml`:

```yaml
apiVersion: oadp.openshift.io/v1alpha1
kind: DataProtectionApplication
metadata:
  name: velero
  namespace: open-cluster-management-backup
spec:
  backupImages: false
  configuration:
    velero:
      defaultPlugins:
        - openshift
        - aws
  backupLocations:
    - name: default
      velero:
        provider: aws
        default: true
        objectStorage:
          bucket: <ACM_BACKUP_BUCKET>
          prefix: velero
        config:
          region: <COS_LOCATION_OR_PROVIDER_REGION_VALUE>
          profile: default
          s3Url: <COS_S3_ENDPOINT>
          s3ForcePathStyle: "true"
        credential:
          name: cloud-credentials
          key: cloud
```
{: codeblock}

Apply the DPA to the cluster:

```bash
oc apply -f dpa.yaml
```
{: pre}

Verify the DPA and the resources that it manages:

```bash
oc get dpa -n open-cluster-management-backup
oc get backupstoragelocation -n open-cluster-management-backup
oc get deployment,pods -n open-cluster-management-backup
oc describe dpa velero -n open-cluster-management-backup
```
{: pre}

Expected result:

- DPA status is `Reconciled`.
- `BackupStorageLocation/default` is created.
- The Velero `Deployment` is running.

Do not edit the generated `BackupStorageLocation` directly. It is reconciled by the DPA controller. Modify the `DataProtectionApplication` (DPA) when you need to change the bucket, endpoint, credentials, prefix, or storage configuration.
{: important}

If the same Hub uses OADP for a separate workload-level backup use case, design the configuration per applicable OADP documentation and verify compatibility with the RHACM-managed OADP installation before you add Node Agent or volume snapshots.
{: note}

## Step 5: Verify BackupStorageLocation is available
{: #acm-hub-backup-step5-verify-bsl}

Run the following command:

```bash
oc get backupstoragelocation -n open-cluster-management-backup
```
{: pre}

Expected result:

```text
NAME      PHASE       LAST VALIDATED   AGE   DEFAULT
default   Available   <time>           <age> true
```
{: screen}

If the phase is `Unavailable`, review the DPA, credentials `Secret`, COS bucket, endpoint, and Service ID IAM policy before you continue.

Do not create a `BackupSchedule` until `BackupStorageLocation/default` is `Available`.
{: important}

## Step 6: Enable the Managed Service Account component
{: #acm-hub-backup-step6-msa}

Managed Service Account support enables RHACM to automatically reconnect imported managed clusters after their activation data is restored to a replacement Hub.

Setting `useManagedServiceAccount: true` on the `BackupSchedule` is not sufficient by itself. The `managedserviceaccount` component must also be enabled in `MultiClusterEngine`.

### 6.1 Enable `managedserviceaccount` in MultiClusterEngine
{: #acm-hub-backup-step6-1-enable-msa-mce}

List the `MultiClusterEngine` resource:

```bash
oc get multiclusterengine
```
{: pre}

Edit the resource:

```bash
oc edit multiclusterengine <MULTICLUSTERENGINE_NAME>
```
{: pre}

Under `spec.overrides.components`, help ensure that the following component is enabled:

```yaml
spec:
  overrides:
    components:
      - name: managedserviceaccount
        enabled: true
```
{: codeblock}

Verify that the component is enabled:

```bash
oc get multiclusterengine <MULTICLUSTERENGINE_NAME> -o yaml
```
{: pre}

Confirm the following output:

```yaml
name: managedserviceaccount
enabled: true
```
{: codeblock}

### 6.2 Validate the Managed Service Account and automatic-import credentials
{: #acm-hub-backup-step6-2-validate-msa}

Before you rely on a backup for replacement-Hub recovery, verify the automatic-import resources for each imported managed cluster that must reconnect.

List managed service accounts:

```bash
oc get managedserviceaccount -A
```
{: pre}

For a managed cluster, inspect `auto-import-account`:

```bash
oc get managedserviceaccount auto-import-account \
  -n <MANAGED_CLUSTER_NAME> -o yaml
```
{: pre}

Check the token `Secret` reference:

```bash
oc get managedserviceaccount auto-import-account \
  -n <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{.status.tokenSecretRef.name}{"\n"}'
```
{: pre}

Verify that the referenced `Secret` exists:

```bash
oc get secret <TOKEN_SECRET_NAME> -n <MANAGED_CLUSTER_NAME>
```
{: pre}

Verify the backup label on the token `Secret`:

```bash
oc get secret <TOKEN_SECRET_NAME> \
  -n <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{.metadata.labels.cluster\.open-cluster-management\.io/backup}{"\n"}'
```
{: pre}

Also, verify that the managed cluster contains API server information:

```bash
oc get managedcluster <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{.spec.managedClusterClientConfigs[*].url}{"\n"}'
```
{: pre}

Confirm the following items before you take a recovery backup:

- `auto-import-account` exists.
- `status.tokenSecretRef` is populated.
- The referenced `Secret` exists.
- The token is valid for the intended recovery window and has not expired.
- The managed cluster was reachable when the token was created or refreshed.
- The token `Secret` carries the RHACM backup label. Verify that it is included in a completed backup after the backup is generated.
- `ManagedCluster.spec.managedClusterClientConfigs` contains the managed-cluster API server information that automatic import requires.

The default Managed Service Account token lifetime is tied to the backup time to live (TTL). Do not override `managedServiceAccountTTL` unless you have a specific requirement and understand the effect on token expiration across the backup retention period.
{: important}

## Step 7: Create the RHACM BackupSchedule
{: #acm-hub-backup-step7-backupschedule}

RHACM Hub backup uses the RHACM `BackupSchedule` custom resource. Do not manually create generic Velero `Backup` resources as the normal RHACM Hub backup workflow.

Create `backup-schedule.yaml`:

```yaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: BackupSchedule
metadata:
  name: acm-backup
  namespace: open-cluster-management-backup
spec:
  veleroSchedule: "0 */6 * * *"
  veleroTtl: 720h0m0s
  useManagedServiceAccount: true
  paused: false
```
{: codeblock}

Apply the backup schedule:

```bash
oc apply -f backup-schedule.yaml
```
{: pre}

Verify the `BackupSchedule` and generated Velero schedules:

```bash
oc get backupschedule acm-backup \
  -n open-cluster-management-backup -o yaml
oc get schedule -n open-cluster-management-backup
```
{: pre}

Expected result:

- `BackupSchedule/acm-backup` is `Enabled`.
- `spec.useManagedServiceAccount` is `true`.
- `spec.paused` is `false` during normal active-Hub operation.
- RHACM creates the underlying Velero schedules that are required by the installed RHACM version.

### Core RHACM backup outputs
{: #acm-hub-backup-step7-core-outputs}

When you validate backup content, confirm completed backup outputs for these core data groups:

- `acm-credentials-schedule-<timestamp>`
- `acm-managed-clusters-schedule-<timestamp>`
- `acm-resources-generic-schedule-<timestamp>`
- `acm-resources-schedule-<timestamp>`

Additional helper or validation schedules can appear depending on the installed RHACM version and configuration. Treat the RHACM BackupSchedule status and the applicable RHACM documentation as authoritative for the installed version.

### Validation-policy backup
{: #acm-hub-backup-step7-validation-policy}

If `acm-validation-policy-schedule` exists, validate it separately from the core management-plane backup outputs. The RHACM backup validation policy uses it to confirm that scheduled backup activity is occurring.

Use one active RHACM `BackupSchedule` per active Hub and backup repository. Multiple Hubs writing scheduled backups to the same repository can cause a `BackupCollision` condition and make the latest-backup source ambiguous.
{: important}

## Step 8: Execute and validate an RHACM backup
{: #acm-hub-backup-step8-execute-validate}

The `BackupSchedule` controller automatically creates the RHACM backup objects according to the configured cron schedule.

### 8.1 Validate the normal schedule
{: #acm-hub-backup-step8-1-validate-schedule}

Check the schedule and current backup objects:

```bash
oc get backupschedule acm-backup -n open-cluster-management-backup
oc get backup -n open-cluster-management-backup
```
{: pre}

For functional testing, you can temporarily use a shorter interval. Choose an interval longer than the expected backup duration for your Hub. For example:

```bash
oc patch backupschedule acm-backup \
  -n open-cluster-management-backup \
  --type merge \
  -p '{"spec":{"veleroSchedule":"*/5 * * * *"}}'
```
{: pre}

Do not use a short schedule that starts new backups before earlier backups can finish. Large Hubs can require significantly more time than small test environments. This is a test-only setting.
{: important}

After the test interval, verify the generated backups:

```bash
oc get backup -n open-cluster-management-backup
```
{: pre}

Verify backup status, storage location, and TTL:

```bash
oc get backup -n open-cluster-management-backup \
  -o custom-columns=NAME:.metadata.name,PHASE:.status.phase,LOCATION:.spec.storageLocation,TTL:.spec.ttl
```
{: pre}

For each expected backup, confirm:

- `PHASE`: `Completed`
- `LOCATION`: `default`
- `TTL`: matches the configured retention

Verify that backup objects are present in the configured COS bucket under the configured prefix.

### 8.2 Restore the normal schedule
{: #acm-hub-backup-step8-2-restore-schedule}

After testing, restore the production schedule:

```bash
oc patch backupschedule acm-backup \
  -n open-cluster-management-backup \
  --type merge \
  -p '{"spec":{"veleroSchedule":"0 */6 * * *"}}'
```
{: pre}

### 8.3 Validate the recovery prerequisites
{: #acm-hub-backup-step8-3-validate-recovery-prereqs}

Before you treat a backup as disaster-recovery ready, also verify:

- `BackupStorageLocation/default` is `Available`.
- The core RHACM backup outputs are `Completed`.
- The Managed Service Account token Secret exists and is valid.
- The token Secret is included in the completed backup for each imported managed cluster that must reconnect automatically.
- The backup repository is reachable from the intended recovery environment.

To inspect the Managed Service Account and its token Secret reference, run:

```bash
oc get managedserviceaccount auto-import-account \
  -n <MANAGED_CLUSTER_NAME> -o yaml
```

Retrieve the referenced token Secret name:

```bash
oc get managedserviceaccount auto-import-account \
  -n <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{.status.tokenSecretRef.name}{"\n"}'
```

Verify the Secret exists and carries the expected backup label:

```bash
oc get secret <TOKEN_SECRET_NAME> \
  -n <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{.metadata.name}{"\t"}{.metadata.labels.cluster\.open-cluster-management\.io/backup}{"\n"}'
```

Use the completed RHACM backup outputs and the applicable backup-scope configuration to confirm that the token Secret was actually captured. The label alone does not prove that the Secret is present in the backup.

## Step 9: Recover RHACM resources on the same Hub
{: #acm-hub-backup-step9-same-hub-recovery}

RHACM supports restoring data from an earlier backup on the same Hub. Use the RHACM-native `Restore.cluster.open-cluster-management.io` resource so that RHACM orchestrates the underlying Velero restore operations.

Before starting the restore, pause the BackupSchedule and verify that it is paused. An enabled schedule prevents the restore from proceeding.

Pause the schedule:

```bash
oc patch backupschedule acm-backup \
  -n open-cluster-management-backup \
  --type merge \
  -p '{"spec":{"paused":true}}'
```
{: pre}
Verify that the schedule is paused:

```bash
oc get backupschedule acm-backup \
  -n open-cluster-management-backup \
  -o jsonpath='{.spec.paused}{"\n"}'
```
{: pre}
Expected result: true.

For a controlled object-recovery test, use a noncritical resource in a dedicated test namespace so the restore can be narrowly scoped.

### 9.1 Prepare a validation resource
{: #acm-hub-backup-step9-1-prepare-resource}

Create or identify a noncritical Governance `Policy` in a dedicated test namespace. Record its YAML before deletion:

```bash
oc get policy <POLICY_NAME> \
  -n <TEST_POLICY_NAMESPACE> \
  -o yaml > policy-before-restore.yaml
```
{: pre}

Ensure that the resource is included in a completed backup before you delete it.
Delete the test resource:

```bash
oc delete policy <POLICY_NAME> -n <TEST_POLICY_NAMESPACE>
```
{: pre}

### 9.2 Identify a completed resources backup
{: #acm-hub-backup-step9-2-identify-backup}

List the available resource backups:

```bash
oc get backup -n open-cluster-management-backup \
  -l velero.io/schedule-name=acm-resources-schedule
```
{: pre}

Choose the completed backup that contains the resource state you want to recover.

### 9.3 Create a scoped RHACM Restore resource
{: #acm-hub-backup-step9-3-create-restore}

Create `restore-same-hub.yaml`:

```yaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Restore
metadata:
  name: restore-acm-same-hub
  namespace: open-cluster-management-backup
spec:
  cleanupBeforeRestore: None
  veleroManagedClustersBackupName: skip
  veleroCredentialsBackupName: skip
  veleroResourcesBackupName: <RESOURCES_BACKUP_NAME>
  includedNamespaces:
    - <TEST_POLICY_NAMESPACE>
```
{: codeblock}

Apply the restore configuration:

```bash
oc apply -f restore-same-hub.yaml
```
{: pre}

Monitor the RHACM restore progress:

```bash
oc get restore restore-acm-same-hub \
  -n open-cluster-management-backup -w
```
{: pre}

Inspect status and events when needed:

```bash
oc describe restore restore-acm-same-hub \
  -n open-cluster-management-backup
```
{: pre}

Expected result: the RHACM `Restore` phase reaches `Finished` and the underlying Velero restore operations complete successfully.

### 9.4 Validate the recovered resource
{: #acm-hub-backup-step9-4-validate-recovery}

Verify that the deleted policy is restored:

```bash
oc get policy <POLICY_NAME> -n <TEST_POLICY_NAMESPACE>
```
{: pre}

Also, verify that normal Hub operations continue to function correctly:

- RHACM console access.
- Managed-cluster status.
- Governance policy processing.
- Placements and bindings.

Use scoped restore filters for same-Hub object recovery when possible. Restoring a historical resources backup without filters can change more Hub resources than the single object that you intended to recover.
{: important}
After validating recovery, resume the backup schedule only if this Hub is intended to remain active.

If the `BackupSchedule` is affected by `BackupCollision`, do not simply unpause it. Follow the BackupCollision troubleshooting procedure, resolve the collision, and delete and recreate the affected schedule on the active Hub before resuming backups.
{: important}

Resume the schedule:
```bash
oc patch backupschedule acm-backup \
  -n open-cluster-management-backup \
  --type merge \
  -p '{"spec":{"paused":false}}'
```
{: pre}

Verify that the schedule is no longer paused:

```bash
oc get backupschedule acm-backup \
  -n open-cluster-management-backup \
  -o jsonpath='{.spec.paused}{"\n"}'
```
{: pre}

The expected output is false. If the schedule is affected by a BackupCollision, follow the collision troubleshooting procedure before resuming backups.

## Step 10: Restore to a replacement RHACM Hub
{: #acm-hub-backup-step10-replacement-hub}

Restore RHACM management-plane data to a replacement Hub by using the same backup repository.

### 10.1 Prepare the replacement Hub
{: #acm-hub-backup-step10-1-prepare-hub}

Before you restore data, configure the replacement Hub to match the source Hub closely enough for the RHACM resources in the backup to be valid.

1. Install RHACM on the replacement {{site.data.keyword.redhat_openshift_notm}} cluster.
2. Use the same RHACM operator version installed when the backup was created.
3. Install RHACM in the same namespace as the source Hub.
4. Install any additional operators that RHACM resources in the backup depend on.
5. Configure `MultiClusterHub` to match the source Hub configuration.
6. Configure `MultiClusterEngine` to match the source Hub configuration, including `managedserviceaccount` when automatic import is required.
7. Enable RHACM `cluster-backup`.
8. Use the RHACM-managed OADP installation unless your environment requires a documented custom or manual OADP configuration.
9. Create the `cloud-credentials` `Secret` with access to the backup repository.
10. Apply the DPA configuration and point it to the same COS bucket and prefix that contain the source-Hub backups.
11. Wait for `BackupStorageLocation/default` to become `Available`.

Verify repository access and backup discovery:

```bash
oc get dpa -n open-cluster-management-backup
oc get backupstoragelocation -n open-cluster-management-backup
oc get backup -n open-cluster-management-backup
```
{: pre}

Expected result: the replacement Hub shows the completed source-Hub backups.

If you want the recovered Hub to run a newer RHACM version, complete the restore with the matching source-Hub version first. Upgrade only after recovery validation is complete.
{: important}

### 10.2 Verify the RHACM Restore custom resource
{: #acm-hub-backup-step10-2-verify-restore-crd}

Verify the RHACM-native Restore custom resource definition (CRD):

```bash
oc get crd restores.cluster.open-cluster-management.io
```
{: pre}

Expected result: the CRD exists and the RHACM `Restore` resource is available.

### 10.3 Determine the state of the original Hub
{: #acm-hub-backup-step10-3-original-hub-state}

The source-Hub actions differ depending on whether you are recovering from an actual outage or running a planned DR test or migration.

#### The original hub is unavailable
{: #acm-hub-backup-step10-3-hub-unavailable}

In a typical disaster-recovery event, the original Hub is unavailable. Continue to step 10.4 after you confirm that the failed Hub cannot resume active management or scheduled backups unexpectedly during recovery.

If the original Hub later returns, do not allow it to manage the same cluster fleet or write scheduled backups to the same repository while the replacement Hub is active.

#### The original hub remains available for a planned DR test or migration
{: #acm-hub-backup-step10-3-hub-available}

If the original Hub remains active, prepare it before you restore managed-cluster activation data to the replacement Hub.

First, pause the source-Hub `BackupSchedule`:

```bash
oc patch backupschedule acm-backup \
  -n open-cluster-management-backup \
  --type merge \
  -p '{"spec":{"paused":true}}'
```
{: pre}

Verify that the schedule is paused:

```bash
oc get backupschedule acm-backup \
  -n open-cluster-management-backup \
  -o jsonpath='{.spec.paused}{"\n"}'
```
{: pre}

Expected result:

```text
true
```
{: screen}

Then, disable automatic reimport on the source Hub for each managed cluster that moves to the replacement Hub:

```bash
oc annotate managedcluster <MANAGED_CLUSTER_NAME> \
  import.open-cluster-management.io/disable-auto-import="" \
  --overwrite
```
{: pre}

Repeat for each managed cluster that moves.

Pause the source-Hub backup schedule before you set the `disable-auto-import` annotation. Do not create a new recovery backup that contains this annotation because that backup can prevent automatic import on the replacement Hub.
{: important}

If you plan to return management to the original Hub after the test, follow the complete RHACM procedure for running a restore while the primary Hub remains active. This includes the source-Hub preparation and failback steps. The steps in section 10.3 cover only the minimum safeguards and are not a substitute for a full failback workflow.
{: note}

### 10.4 Create the RHACM Restore resource
{: #acm-hub-backup-step10-4-create-restore}

Create `restore-acm.yaml`:

```yaml
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Restore
metadata:
  name: restore-acm
  namespace: open-cluster-management-backup
spec:
  cleanupBeforeRestore: None
  veleroManagedClustersBackupName: latest
  veleroCredentialsBackupName: latest
  veleroResourcesBackupName: latest
```
{: codeblock}

Apply the restore configuration:

```bash
oc apply -f restore-acm.yaml
```
{: pre}

Monitor the RHACM restore progress:

```bash
oc get restore restore-acm \
  -n open-cluster-management-backup -w
```
{: pre}

Inspect status and events when needed:

```bash
oc describe restore restore-acm \
  -n open-cluster-management-backup
```
{: pre}

### 10.5 Validate RHACM management-plane recovery
{: #acm-hub-backup-step10-5-validate-mgmt-plane}

After the restore completes, verify the recovered management-plane state:

- RHACM console is accessible.
- Managed clusters are restored and visible.
- ManagedClusterSets and bindings are restored.
- Cluster labels and metadata are restored.
- Placements and placement decisions are restored where applicable.
- Applications are restored.
- Governance policies and policy placement are restored.
- Observability configuration is restored where applicable to the Hub backup scope.
- The DPA and `BackupStorageLocation` are healthy.

Restoring the RHACM Hub does not restore application workloads or persistent workload data on managed clusters.
{: note}

### 10.6 Validate Managed Service Account recovery before you diagnose automatic import
{: #acm-hub-backup-step10-6-validate-msa-recovery}

Before you conclude that automatic managed-cluster reconnection has failed, validate the restored prerequisites on the replacement Hub.

For each imported managed cluster, verify `auto-import-account`:

```bash
oc get managedserviceaccount auto-import-account \
  -n <MANAGED_CLUSTER_NAME> -o yaml
```
{: pre}

Verify:

- `ManagedServiceAccount/auto-import-account` exists.
- `status.tokenSecretRef` is populated.
- The referenced `Secret` exists on the replacement Hub.
- The restored token is still valid.
- The token `Secret` was included in the backup.
- `ManagedCluster.spec.managedClusterClientConfigs` contains the required API server information.
- The managed-cluster API is reachable from the replacement-Hub recovery path.

Also, inspect the RHACM `Restore` resource for token or auto-import errors:

```bash
oc describe restore restore-acm \
  -n open-cluster-management-backup
```
{: pre}

### 10.7 Validate managed-cluster reconnection
{: #acm-hub-backup-step10-7-validate-reconnection}

For each managed cluster, inspect its conditions:

```bash
oc get managedcluster <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{range .status.conditions[*]}{.type}{"\t"}{.status}{"\t"}{.reason}{"\t"}{.message}{"\n"}{end}'
```
{: pre}

The wanted recovered state includes:

```text
ManagedClusterImportSucceeded=True
ManagedClusterJoined=True
ManagedClusterConditionAvailable=True
```
{: screen}

For imported clusters, the preferred recovery path is automatic reconnection that uses the restored Managed Service Account token.

### 10.8 if an imported cluster remains in Pending Import
{: #acm-hub-backup-step10-8-pending-import}

If a managed cluster remains in `Pending Import`, do not immediately delete `Klusterlet`, agent, namespace, or other OCM resources as a generic recovery action.

Complete these checks first:

1. Verify `managedserviceaccount` is enabled in `MultiClusterEngine`.
2. Verify `useManagedServiceAccount: true` was enabled on the `BackupSchedule` that produced the recovery backup.
3. Verify `auto-import-account` and its token `Secret` are present on the replacement Hub.
4. Verify that the token was included in the backup and has not expired.
5. Verify the managed-cluster API server URL is present on the `ManagedCluster` resource.
6. Verify that the managed cluster is reachable.
7. Verify that the original Hub is not reclaiming the cluster.
8. Review the RHACM Restore status and relevant controller logs for automatic-import errors.

If the prerequisites are valid and automatic import still does not complete, follow the RHACM version-specific import troubleshooting procedure or open a Red Hat support case before you perform destructive managed-cluster cleanup.

Collect at least the following evidence:

```bash
oc get managedcluster <MANAGED_CLUSTER_NAME> -o yaml
oc get managedserviceaccount auto-import-account \
  -n <MANAGED_CLUSTER_NAME> -o yaml
oc get secret <TOKEN_SECRET_NAME> \
  -n <MANAGED_CLUSTER_NAME> \
  -o jsonpath='{.metadata.name}{"\t"}{.metadata.labels}{"\n"}'
oc describe restore restore-acm \
  -n open-cluster-management-backup
oc get pods -n multicluster-engine
```
{: pre}

Do not retrieve or share the secret data payload when collecting diagnostic evidence. Metadata, labels, and token-reference status are sufficient for initial triage.

If Red Hat guidance for your version directs you to reimport the cluster, generate a fresh import configuration from the active replacement Hub and follow the documented RHACM import troubleshooting procedure. Do not publish or standardize a destructive `Klusterlet` cleanup sequence unless the affected RHACM version explicitly supports it.
{: note}

### 10.9 Make the replacement Hub active and resume backups
{: #acm-hub-backup-step10-9-resume-backups}

After recovery validation, designate the replacement Hub as the active Hub.

The RHACM `BackupSchedule` resource itself is excluded from RHACM Hub backup. Therefore, do not assume that a `BackupSchedule` was restored to the replacement Hub.

Check whether a `BackupSchedule` exists:

```bash
oc get backupschedule -n open-cluster-management-backup
```
{: pre}

If no `BackupSchedule` exists, create it on the new active Hub by applying the configuration from step 7:

```bash
oc apply -f backup-schedule.yaml
```
{: pre}

If a valid `BackupSchedule` exists and is intentionally paused, enable it:

If the `BackupSchedule` is affected by `BackupCollision`, do not simply unpause it. Follow the BackupCollision troubleshooting procedure, resolve the collision, and delete and recreate the affected schedule on the active Hub before resuming backups.
{: important}

```bash
oc patch backupschedule acm-backup \
  -n open-cluster-management-backup \
  --type merge \
  -p '{"spec":{"paused":false}}'
```
{: pre}

Verify that the schedule is active:

```bash
oc get backupschedule acm-backup \
  -n open-cluster-management-backup
```
{: pre}

Wait for a new backup set from the replacement Hub to complete successfully.

Only the designated active Hub should generate scheduled backups to the shared repository.

If the original Hub still exists, keep its `BackupSchedule` paused or removed while it is inactive. Do not allow both Hubs to write scheduled backups to the same repository.

## Troubleshooting and workarounds
{: #acm-hub-backup-troubleshooting}

### DPA does not reconcile: `backupImages` validation
{: #acm-hub-backup-ts-dpa-backupimages}

Example error:

```text
BackupLocation must have velero prefix when backupImages is not set to false
```
{: screen}

Resolution: Set:

```yaml
spec:
  backupImages: false
```
{: codeblock}

Image backup is not required for RHACM Hub management-plane backup.

### OADP version or duplicate installation problem
{: #acm-hub-backup-ts-oadp-duplicate}

OADP and Velero CRDs are cluster-scoped. Multiple incompatible OADP versions on one cluster can cause one operator instance to run against CRDs from another version.

Check OADP installations and versions:

```bash
oc get subscription -A | grep -i oadp
oc get csv -A | grep -i oadp
oc get dpa -A
```
{: pre}

Use the OADP installation in `open-cluster-management-backup` that the RHACM backup component supports unless your environment requires a documented custom installation.

### BackupStorageLocation is `Unavailable` and Velero shows `403 AccessDenied`
{: #acm-hub-backup-ts-bsl-403}

Likely cause: The HMAC credential is valid, but the Service ID does not have the required {{site.data.keyword.cloud_notm}} IAM access to list or write objects in the COS bucket.

Resolution: Verify the bucket, endpoint, credentials `Secret`, and Service ID IAM policy. Wait for the location to return to `Available`:

```bash
oc get backupstoragelocation -n open-cluster-management-backup
```
{: pre}

### BackupSchedule shows `FailedValidation`
{: #acm-hub-backup-ts-failed-validation}

Example message:

```text
Backup storage location is not available. Check velero.io.BackupStorageLocation
and validate storage credentials.
```
{: screen}

Likely cause: Invalid credentials, a repository-access failure, or an unavailable `BackupStorageLocation`.

Resolution: Restore valid repository access and verify `BackupStorageLocation/default` is `Available` before you continue.

### Direct BackupStorageLocation changes are reverted
{: #acm-hub-backup-ts-bsl-reverted}

Cause: `BackupStorageLocation/default` is managed by the DPA.

Resolution: Modify the `DataProtectionApplication` instead of editing the generated `BackupStorageLocation` directly.

### BackupCollision
{: #acm-hub-backup-ts-backupcollision}

A `BackupCollision` can occur when more than one Hub writes backup data to the same repository.

Check the backup schedule status:

```bash
oc get backupschedule -A
```
{: pre}

Before resuming backups, identify the valid active Hub and stop backup scheduling on the inactive Hub. Resolve the collision, then delete and recreate the affected `BackupSchedule` on the active Hub. Verify that the recreated schedule is healthy before resuming backups. Simply unpausing the existing collided schedule is not sufficient to resolve the condition.

### Managed cluster remains in `Pending Import`
{: #acm-hub-backup-ts-pending-import}

Verify, in order:

1. `managedserviceaccount` is enabled in `MultiClusterEngine`.
2. `useManagedServiceAccount: true` was enabled when the recovery backup was created.
3. `auto-import-account` exists after restore.
4. `status.tokenSecretRef` points to an existing `Secret`.
5. The token is valid and not expired.
6. The token `Secret` was included in the backup.
7. The managed cluster contains the required API server information.
8. The managed cluster is reachable.
9. The original Hub is not reclaiming the cluster.
10. The RHACM `Restore` events do not report token or automatic-import errors.

If these prerequisites are valid and the cluster still does not reconnect, use the RHACM version-specific troubleshooting procedure or open a Red Hat support case.

### The replacement Hub is running a newer RHACM version
{: #acm-hub-backup-ts-newer-version}

Do not restore a backup directly into a replacement Hub that is already running a newer RHACM operator version than the Hub that created the backup.

Install the matching RHACM version first, restore and validate the backup, then upgrade the recovered Hub.

### Replacement Hub has no BackupSchedule after restore
{: #acm-hub-backup-ts-no-schedule}

This is expected because the RHACM `BackupSchedule` resource is excluded from Hub backup.

After the replacement Hub is designated active, create a new `BackupSchedule` using the approved configuration and confirm that a fresh backup set completes successfully.

### Backups remain `InProgress` for too long
{: #acm-hub-backup-ts-inprogress}

Large Hub environments can require more Velero CPU or memory than small test environments.

Check backup status and Velero pod health:

```bash
oc get backup -n open-cluster-management-backup
oc get pods -n open-cluster-management-backup
```
{: pre}

If backups consistently exceed the expected window or Velero shows resource pressure, review the RHACM and OADP guidance for Velero resource requests and limits before you reduce the backup interval.

## Final readiness checklist
{: #acm-hub-backup-readiness-checklist}

Before you consider RHACM Hub backup and restore ready for use, confirm all applicable items in the following table.

| Check | Expected state |
| --- | --- |
| RHACM `cluster-backup` component | Enabled |
| RHACM-managed OADP | Installed and running in `open-cluster-management-backup` |
| Incompatible duplicate OADP versions | Not present |
| COS bucket resiliency | Matches the intended recovery scope |
| `cloud-credentials` `Secret` | Exists in `open-cluster-management-backup` |
| DPA `velero` | Reconciled |
| `BackupStorageLocation/default` | Available |
| Velero `Deployment` | Running |
| `MultiClusterEngine` `managedserviceaccount` component | Enabled when automatic import is required |
| `auto-import-account` | Present for imported clusters that require automatic reconnection |
| Managed Service Account token `Secret` | Present, valid, and included in backup |
| Managed-cluster API server information | Present for automatic import |
| `BackupSchedule` on active source Hub | Enabled |
| `useManagedServiceAccount` | `true` when automatic import is required |
| Normal `paused` state | `false` on the active Hub |
| Core RHACM backup outputs | Completed |
| Backup objects stored in {{site.data.keyword.cos_short}} | Verified |
| Repository reachable from recovery environment | Verified |
| Same-Hub recovery | RHACM-native `Restore` validated with scoped test resource |
| Replacement-Hub RHACM version before restore | Matches source-Hub backup version |
| Replacement-Hub configuration | Required operators, MultiClusterHub (MCH), MultiClusterEngine (MCE), DPA, and repository access configured |
| RHACM-native replacement-Hub restore | Validated |
| Managed-cluster automatic reconnection | Validated or troubleshooting evidence captured |
| Active Hub after recovery | Explicitly designated |
| BackupSchedule on replacement Hub | Created or enabled after promotion |
| Inactive/original Hub | Not managing the same fleet or writing backups to the shared repository |
{: caption="Final readiness checklist" caption-side="bottom"}

## Operational guidance
{: #acm-hub-backup-operational-guidance}

### Before a Hub backup intended for disaster recovery
{: #acm-hub-backup-ops-before-backup}

- Confirm `cluster-backup` is enabled.
- Confirm that the RHACM-managed OADP infrastructure is healthy.
- Confirm `BackupStorageLocation/default` is `Available`.
- Confirm the {{site.data.keyword.cos_short}} bucket resiliency and location match the intended recovery scenario.
- Confirm that the recovery environment can access the {{site.data.keyword.cos_short}} repository.
- Confirm `managedserviceaccount` is enabled in `MultiClusterEngine` when imported clusters must reconnect automatically.
- Confirm `useManagedServiceAccount: true` on the `BackupSchedule` when automatic import is required.
- Confirm `auto-import-account` and its token `Secret` exist, are valid, and are included in backup.
- Confirm core RHACM backups complete within the expected backup window.

### Before you restore to a replacement Hub
{: #acm-hub-backup-ops-before-restore}

- Record or verify the RHACM version that created the backup.
- Install the matching RHACM version on the replacement Hub.
- Match the source-Hub RHACM namespace and required operator configuration.
- Match the `MultiClusterHub` and `MultiClusterEngine` configuration that is required by the restored resources.
- Verify that the replacement Hub can access the {{site.data.keyword.cos_short}} repository.
- Verify `BackupStorageLocation/default` is `Available`.
- Verify the RHACM Restore CRD/controller is available.
- If the original Hub is still active, pause its `BackupSchedule` before you move managed clusters.
- Verify the Managed Service Account and token prerequisites before you diagnose an automatic-import failure.

### After recovery
{: #acm-hub-backup-ops-after-recovery}

- Designate exactly one Hub as active.
- Verify managed-cluster conditions: `ManagedClusterImportSucceeded`, `ManagedClusterJoined`, and `ManagedClusterConditionAvailable`.
- Create or enable the `BackupSchedule` on the active replacement Hub.
- Verify that a new backup set from the active Hub completes successfully.
- Keep inactive Hubs from writing scheduled backups to the same repository.
- Upgrade RHACM only after the matching-version restore is complete and validated.

## Related information
{: #acm-hub-backup-related-info}

- [{{site.data.keyword.cloud_notm}} Virtualization Solutions overview](/docs/virtualization-solutions?topic=virtualization-solutions-overview)
- [{{site.data.keyword.cos_full_notm}} high availability and disaster recovery](/docs/cloud-object-storage?topic=cloud-object-storage-cos-ha-dr)
- [{{site.data.keyword.cos_full_notm}} endpoints and storage locations](/docs/cloud-object-storage?topic=cloud-object-storage-endpoints)
- [Red Hat Advanced Cluster Management 2.16 business continuity](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.16/html-single/business_continuity/index){: external}
