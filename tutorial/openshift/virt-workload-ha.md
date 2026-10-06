---

copyright:
  years: 2026
lastupdated: "2026-10-06"
lasttested: "[{LAST_TESTED_DATE}]"

keywords: OpenShift Virtualization high availability IBM Cloud, Node Health Check OpenShift IBM Cloud, Self Node Remediation IBM Cloud, ROKS VM high availability, configure HA OpenShift Virtualization IBM Cloud, automatic VM recovery IBM Cloud, NHC SNR OpenShift IBM Cloud, OpenShift Virtualization workload HA, node failure recovery ROKS, high availability VM workloads IBM Cloud


subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization, VMware
account-plan: paid
completion-time: 15m

---

{{site.data.keyword.attribute-definition-list}}

# Configuring {{site.data.keyword.redhat_openshift_notm}} Virtualization high availability
{: #virt-workload-ha}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization, VMware"}
{: toc-completion-time="15m"}

Configure {{site.data.keyword.redhat_openshift_notm}} Virtualization high availability by using Node Health Check and Self Node Remediation operators for automatic failure detection and workload recovery on {{site.data.keyword.cloud_notm}}.

{: shortdesc}

## Objectives
{: #virt-workload-ha-objectives}

The tutorial covers the following tasks:

* Install and configure the Node Health Check Operator
* Install and configure the Self-Node Remediation Operator
* Create a NodeHealthCheck custom resource to define node health criteria
* Verify automatic node remediation

## Before you begin
{: #virt-workload-ha-prereqs}

Ensure that the following prerequisites are in place before you begin:

* An {{site.data.keyword.cloud_notm}} {{site.data.keyword.redhat_openshift_notm}} on VPC cluster running {{site.data.keyword.redhat_openshift_notm}} 4.12 or later
* cluster administrator privileges
* The `oc` command-line interface (CLI) installed and configured to access your cluster
* The `ibmcloud` CLI installed with the Kubernetes&reg; Service plug-in

For more information about setting up the CLI tools, see [Setting up the CLI](/docs/openshift?topic=openshift-cli-install){: external}.

## High availability for virtualization workloads
{: #virt-workload-ha-overview}

High availability (HA) for virtualization workloads on {{site.data.keyword.redhat_openshift_notm}} Kubernetes Service (ROKS) ensures that virtual machines (VMs) continue to operate reliably despite infrastructure failures such as worker node outages or hardware issues.


In a {{site.data.keyword.redhat_openshift_notm}} Virtualization environment running on ROVS, ROVS provides HA through tight integration among {{site.data.keyword.redhat_openshift_notm}} control plane services, Kubernetes scheduling, and {{site.data.keyword.redhat_openshift_notm}} Virtualization operators. The platform continuously monitors the health of worker nodes and VMs. When a worker node becomes unavailable, {{site.data.keyword.redhat_openshift_notm}} automatically detects the failure and triggers recovery actions.

Virtual machines affected by a node failure are automatically restarted or migrated to healthy worker nodes in the cluster. This process is enabled by shared, resilient storage, both {{site.data.keyword.redhat_openshift_notm}} Data Foundation (ODF) and NFS, which ensures that VM disks remain accessible across nodes during failover events.

Key capabilities that enable HA on {{site.data.keyword.cloud_notm}} ROKS include:


* Operator-driven node remediation through NHC and SNR to detect unhealthy nodes and recover VMs automatically
* Kubernetes-based scheduling to place recovered VMs on available worker nodes
* Resilient, shared storage by using ODF and NFS to support VM restart and migration
* Reduced operational complexity and recovery time through minimal manual intervention

By using OpenShift Virtualization on {{site.data.keyword.cloud_notm}} ROKS, you can run critical virtualization workloads with improved reliability, faster recovery, and built-in high availability aligned with enterprise availability and business continuity requirements.

### Node Health Check and Self-Node Remediation integration
{: #virt-workload-ha-nhc-snr}

To further enhance high availability for virtualization workloads on {{site.data.keyword.cloud_notm}} ROKS, Node Health Check (NHC) and Self Node Remediation (SNR) work together to automatically detect unhealthy worker nodes and recover them without manual intervention. This proactive remediation helps prevent prolonged outages and improves overall cluster resilience.

Node Health Check (NHC) continuously monitors the health of worker nodes by evaluating conditions such as node readiness, `kubelet` status, and heartbeat signals. When a node becomes unhealthy or unresponsive beyond a defined threshold, NHC determines that the node requires remediation and triggers corrective actions.

After remediation is triggered, Self-Node Remediation (SNR) performs node-level recovery operations. Based on the failure scenario and configuration, SNR can safely restart the affected node or perform fencing to isolate it from the cluster. When SNR isolates faulty nodes, they no longer impact running workloads or cause split-brain or data consistency issues.

NHC and SNR integration is especially critical for {{site.data.keyword.redhat_openshift_notm}} Virtualization workloads:

* The cluster quickly isolates unhealthy nodes so that VMs are not scheduled on unstable infrastructure.
* The cluster automatically migrates virtual machines from failed nodes to healthy nodes.
* The cluster allows remediated nodes to safely rejoin and resume hosting workloads after recovery.

By combining NHC for detection and SNR for automated remediation, OpenShift Virtualization on {{site.data.keyword.cloud_notm}} ROKS achieves faster failure response, reduced operational burden, and improved continuity for mission-critical virtualization workloads.

Red Hat also provides Fence Agents Remediation (FAR) to implement remediation by hardware or power-based fencing through BMC API, which is not currently supported on worker nodes of ROKS.

{: note}

## Disable outbound traffic protection and enable the operator catalog
{: #virt-workload-ha-enable-catalog}
{: step}

The Node Health Check and Self-Node Remediation operators are available from the Red Hat Operator catalog.

If outbound traffic protection is disabled on your cluster (as recommended), skip this step.

If outbound traffic protection is enabled (the default setting), the Red Hat Operator catalogs are disabled on ROVS. Use the following steps to enable the default catalog source on ROVS.

1. Disable outbound protection to help ensure that the cluster has outbound internet access to pull the required images from sources such as `registry.redhat.io` and `quay.io`.

   ```sh
   ibmcloud oc vpc outbound-traffic-protection disable -c <cluster-name>
   ```
   {: pre}

2. Get your cluster configuration.

   ```sh
   ibmcloud ks cluster config --admin -c <cluster-name>
   ```
   {: pre}

3. Enable the default `CatalogSource`.

   ```sh
   oc patch operatorhub cluster --type json -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": false}]'
   ```
   {: pre}

4. Check the `CatalogSource` status.

   ```sh
   oc -n openshift-marketplace get catalogsource
   ```
   {: pre}

5. Verify that the required operators are available.

   ```sh
   oc get packagemanifests -n openshift-marketplace | grep node-healthcheck-operator
   oc get packagemanifests -n openshift-marketplace | grep self-node-remediation
   ```
   {: pre}

You can view the list of operators in each CatalogSource from the {{site.data.keyword.redhat_openshift_notm}} Console by navigating to **Administration** > **CustomResourceDefinitions** > **OperatorHub details**.


## Install the Node Health Check and Self-Node Remediation operators
{: #virt-workload-ha-install-operators}
{: step}

Install both operators from the {{site.data.keyword.redhat_openshift_notm}} console OperatorHub.

1. Log in to the {{site.data.keyword.redhat_openshift_notm}} console.

2. Go to **Operators** > **OperatorHub**.

3. Search for and install the **Node Health Check Operator** with the following settings:
   * **Name**: Node Health Check
   * **Namespace**: `openshift-workload-availability`
   * **Approval strategy**: Automatic

4. Search for and install the **Self-Node Remediation Operator** with the following settings:
   * **Name**: Self-Node Remediation
   * **Namespace**: `openshift-workload-availability`
   * **Approval strategy**: Automatic

5. Wait until both operators show a status of **Succeeded**.

## Configure the Self-Node Remediation Operator
{: #virt-workload-ha-configure-snr}
{: step}

The Self-Node Remediation Operator creates a default `SelfNodeRemediationConfig` named `self-node-remediation-config` and a default `SelfNodeRemediationTemplate` named `self-node-remediation-automatic-strategy-template`.

The *Self-Node Remediation Config* (SNR Config) custom resource (CR) defines parameters for the Self-Node Remediation Operator, which automatically detects, fences, and restarts unhealthy Kubernetes nodes without requiring management interfaces such as Intelligent Platform Management Interface (IPMI). This agentless approach makes SNR Config useful for bare-metal or constrained environments where hardware management interfaces are unavailable.

The *Self-Node Remediation Template* (SNR Template) defines the restart strategy that the Self-Node Remediation Operator applies when it triggers recovery of an unhealthy node. By fencing failed nodes through software restarts or watchdog devices, SNR Template helps ensure safe workload recovery, particularly for stateful applications.

You can use the default `SelfNodeRemediationConfig` and `SelfNodeRemediationTemplate` resources that the Self-Node Remediation Operator automatically creates, or customize them to meet specific cluster requirements.

### Modify or create a Self-Node Remediation Config
{: #virt-workload-ha-snr-config}

Modify the SNR Config to adjust detection and fencing parameters, such as API server timeouts, peer check intervals, and the minimum peer count required before remediation is triggered. To modify the default Self-Node Remediation Config or create a new one:

1. In the {{site.data.keyword.redhat_openshift_notm}} console, go to **Operators** > **Installed Operators** (**Project:** ***openshift-workload-availability***) > **Self-Node Remediation Operator** > **Self-Node Remediation Config**.

2. To modify the default configuration:
   1. Open `self-node-remediation-config`
   1. Click **Actions** and select **Edit SelfNodeRemediationConfig**
   1. Update the YAML parameters as needed (for example, remediation strategy)
   1. Click **Save** to apply the changes

3. To create a new configuration:
   * Click **Create SelfNodeRemediationConfig** and provide the required parameters
   * Alternatively, create a YAML file named `snr-config.yaml`:

   ```yaml
   apiVersion: self-node-remediation.medik8s.io/v1alpha1
   kind: SelfNodeRemediationConfig
   metadata:
     name: customer-self-node-remediation-config
     namespace: openshift-workload-availability
   spec:
     apiServerTimeout: 5s
     peerApiServerTimeout: 5s
     hostPort: 30001
     isSoftwareRebootEnabled: true
     watchdogFilePath: /dev/watchdog
     peerDialTimeout: 5s
     peerUpdateInterval: 15m
     apiCheckInterval: 15s
     peerRequestTimeout: 7s
     maxApiErrorThreshold: 3
     minPeersForRemediation: 1
   ```
   {: codeblock}

4. Apply the configuration.

   ```sh
   oc apply -f snr-config.yaml
   ```
   {: pre}

For a detailed explanation of all available parameters, see [Understanding the Self-Node Remediation Operator configuration](https://docs.redhat.com/en/documentation/workload_availability_for_red_hat_openshift/26.1/html-single/remediation_fencing_and_maintenance/index#understanding-self-node-remediation-operator-config_self-node-remediation-operator-remediate-nodes){: external}.
{: tip}

### Modify or create a Self-Node Remediation Template
{: #virt-workload-ha-snr-template}

Modify the SNR Template to control the restart strategy that is applied when an unhealthy node is remediated, for example, to switch between `Automatic`, `ResourceDeletion`, or `OutOfServiceTaint` strategies. To modify the default Self-Node Remediation Template or create a new one:

1. In the {{site.data.keyword.redhat_openshift_notm}} console, go to **Operators** > **Installed Operators** (**Project:** ***openshift-workload-availability***) > **Self-Node Remediation Operator** > **Self-Node Remediation Template**.

2. To modify the default template:
   1. Open `self-node-remediation-automatic-strategy-template`
   1. Click **Actions** and select **Edit SelfNodeRemediationTemplate**
   1. Update the YAML parameters as needed (for example, remediation strategy)
   1. Click **Save** to apply the changes

3. To create a new template:
   * Click **Create SelfNodeRemediationTemplate** and provide the required parameters
   * Alternatively, create a YAML file named `snr-template.yaml`:

   ```yaml
   apiVersion: self-node-remediation.medik8s.io/v1alpha1
   kind: SelfNodeRemediationTemplate
   metadata:
     name: customer-self-node-remediation-template
     namespace: openshift-workload-availability
   spec:
     template:
       spec:
         remediationStrategy: Automatic
   ```
   {: codeblock}

4. Apply the template.

   ```sh
   oc apply -f snr-template.yaml
   ```
   {: pre}

For a detailed explanation of all available parameters, see [Understanding the Self-Node Remediation Template configuration](https://docs.redhat.com/en/documentation/workload_availability_for_red_hat_openshift/26.1/html-single/remediation_fencing_and_maintenance/index#understanding-self-node-remediation-remediation-template-config_self-node-remediation-operator-remediate-nodes){: external}.
{: tip}

## Configure the Node Health Check Operator
{: #virt-workload-ha-configure-nhc}
{: step}

The Node Health Check (NHC) Operator monitors the health of nodes in a {{site.data.keyword.redhat_openshift_notm}} cluster and identifies nodes that become unhealthy.

The `NodeHealthCheck` custom resource (CR) defines the specific criteria and thresholds to determine whether a node is unhealthy and requires remediation.

### Create a `NodeHealthCheck` resource
{: #virt-workload-ha-create-nhc}

To define node health criteria and configure remediation triggers by using the OpenShift console or CLI, perform the following steps:

1. In the OpenShift Console, navigate to **Operators** > **Installed Operators** (Project: `openshift-workload-availability`) > **Node Health Check Operator** > **Node Health Check**.

2. Click **Create NodeHealthCheck** and provide the required parameters.

3. Alternatively, create a YAML file named `nhc-node-notready.yaml`:

   ```yaml
   apiVersion: remediation.medik8s.io/v1alpha1
   kind: NodeHealthCheck
   metadata:
     name: customer-node-notready-check
     namespace: openshift-workload-availability
   spec:
     minHealthy: 51%
     unhealthyConditions:
       - type: Ready
         status: "False"
         duration: 300s
       - type: Ready
         status: "Unknown"
         duration: 300s
     remediationTemplate:
       apiVersion: self-node-remediation.medik8s.io/v1alpha1
       kind: SelfNodeRemediationTemplate
       name: self-node-remediation-automatic-strategy-template
       namespace: openshift-workload-availability
   ```
   {: codeblock}

4. Apply the configuration.

   ```sh
   oc apply -f nhc-node-notready.yaml
   ```
   {: pre}

The `remediationTemplate` must match the name of either the default `SelfNodeRemediationTemplate` or a custom `SelfNodeRemediationTemplate` created in the Self-Node Remediation Operator (as described in [Modifying or creating a Self-Node Remediation Template](#virt-workload-ha-snr-template)).
{: note}

For a detailed explanation of all available parameters, see [About the Node Health Check Operator](https://docs.redhat.com/en/documentation/workload_availability_for_red_hat_openshift/26.1/html/remediation_fencing_and_maintenance/node-health-check-operator#about-node-health-check-operator_node-health-check-operator){: external}.
{: tip}

## Verify the configuration
{: #virt-workload-ha-verify}
{: step}

After you configure the operators, verify that they work correctly.

1. Check the `NodeHealthCheck` resources.

   ```sh
   oc get nhc -n openshift-workload-availability
   ```
   {: pre}

2. Check the `SelfNodeRemediationConfig` resources.

   ```sh
   oc get selfnoderemediationconfig -n openshift-workload-availability
   ```
   {: pre}

3. Check the `SelfNodeRemediationTemplate` resources.

   ```sh
   oc get selfnoderemediationtemplate -n openshift-workload-availability
   ```
   {: pre}

## Next steps
{: #virt-workload-ha-next-steps}

After you configure automatic node health checking and remediation, consider the following actions:

* Monitor your cluster's node health and remediation events through the {{site.data.keyword.redhat_openshift_notm}} console
* Customize the health check thresholds based on your workload requirements
* Review the [Red Hat OpenShift Virtualization documentation](/docs/openshift?topic=openshift-virt-overview) for additional high availability features
* Explore [backup and disaster recovery options](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-backup) for your virtualization workloads
