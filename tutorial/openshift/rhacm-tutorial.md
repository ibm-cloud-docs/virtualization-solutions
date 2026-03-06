---

copyright:
  years: 2026, 2026
lastupdated: "2026-03-06"

keywords: ROKS, OpenShift Data Foundation, ODF, observability, monitoring, logging, alerting, metrics, dashboards, ACM, LokiStack, IBM Cloud Logs

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization
account-plan: paid
completion-time: 60m
use-case: ApplicationModernization
industry: Software and platform applications
compliance: HIPPA

---

{{site.data.keyword.attribute-definition-list}}

# Observability with Red Hat Advanced Cluster Management

{: #rhacm-overview}
{: #vsphere-openshift-rhacm}
{: #tutorial-rhacm-overview}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}
{: toc-use-case="ApplicationModernization"}
{: toc-industry="Software and platform applications"}
{: toc-compliance="HIPPA"}

Learn how to implement comprehensive observability for {{site.data.keyword.redhat_openshift_full}} Virtualization environments, including monitoring, logging, alerting, and metrics collection by using Red Hat Advanced Cluster Management (ACM).
{: shortdesc}

## Introduction to {{site.data.keyword.redhat_openshift_notm}} Virtualization
{: #roks-virt-observ-intro}

With Red Hat Advanced Cluster Management (RHACM), you can manage and control multiple clusters under one {{site.data.keyword.redhat_openshift_notm}} instance. RHACM provides capabilities such as remote clustering, workloads across nodes and regions, disaster recovery, and more. Each managed cluster reports back to the designated hub cluster. Often, this configuration means that the hub cluster contains the control-plane nodes, while the managed clusters consist of worker nodes. A Klusterlet is installed on each managed cluster and controls communication with the hub cluster. The term "local cluster" refers to a hub cluster that also acts as a managed cluster.

However, RHACM installs through a separate operator on a {{site.data.keyword.redhat_openshift_notm}} instance. This installation requires a license and subscription agreement. After you install RHACM, you can use more multi-cluster features with extra configurations to expand capabilities.

By default, {{site.data.keyword.redhat_openshift_notm}} provides several observability options. Without extra operators, you can access basic alerting, monitoring, and logging. {{site.data.keyword.redhat_openshift_notm}} provides a Prometheus-based platform that focuses on critical cluster functions and expands through customization. The following components support observability:

- Cluster Monitoring Operator (CMO) - Manages the observability components
- Prometheus - Monitors data that is received from the CMO and creates a database engine for metrics
- Metrics server - Collects metrics and set up the API service for Thanos
- Thanos Querier - Provides an interface to collect query metrics
- Alertmanager - Receives alerts from Prometheus and sends them to external systems

{{site.data.keyword.redhat_openshift_notm}} automatically deploys these components in the **Openshift-monitoring namespace** during the Red Hat OpenShift Container Platform deployment. You can access dashboards and configuration of these components from the **Observe** section in the **Administration view**.

Alerting displays the current alerts, alerting rules, and supports viewing and adding alert silencing rules. You use the metrics page to run PromQL queries and to receive metrics data. Dashboards display graphs from a preset list. Targets are the defined list of endpoints that the metrics server calls to receive data from different components. The logs section appears after you install and configure {{site.data.keyword.redhat_openshift_notm}} Logging. For more information about the design and components, see [Monitoring stack architecture]( https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/monitoring/about-openshift-container-platform-monitoring#monitoring-stack-architecture){: external}

RHACM adds extra observability functions that help support multiple host clusters. RHACM expands existing Prometheus, Thanos, and Alertmanager infrastructure but with extra components.

- Multicluster observability operator - Monitors and gathers managed cluster metrics.
- Grafana - A new customizable dashboard for metrics, including more node and virtual machine metrics.
- Observability add-on controller - A new apiserver that handles the managed clusters.
- Thanos Compactor - Allows for long-term storage of metrics data.

The RHACM observability feature is available after you install the RHACM operator because it is not enabled by default. The following guide assumes that ACM observability is enabled, especially regarding metrics and dashboards.

For more information, see [RHACM Observability architecture](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#observing-environments-intro_observability){: external}.

## Installing Red Hat Advanced Cluster Management
{: #roks-virt-observ-installing}
{: step}

To install Red Hat Advanced Cluster Management on an Red Hat OpenShift Container Platform instance, use the following steps.

1. Log in to your Red Hat OpenShift Container Platform instance.
1. Go to **Ecosystem > Software catalog** and select a **namespace**.
1. Search for **Advanced cluster management for Kubernetes**.
1. Click **Install** if it is not installed.
1. Open the operator from **Ecosystem > Installed Operators**.
1. In the **MultiClusterHub** tab, click **Create MultiClusterHub**.
1. Enter an appropriate name and click **Create**.
1. Go to **Storage > StorageClasses** and verify that the **StorageClass** is set as the **Default** for infrastructure deployment.
1. Follow the steps to [enable the Observability modules of RHACM](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#enabling-observability){: external}.
1. Go to the **Observability** tab under **MultiClusterHub**.
1. In the **Observability** tab, click **Create observability**
1. Enter an appropriate name and click **Create**
1. After the Observability instance is created, go to the **Observability** tab and click **Observability instance**.

If you use ODF, verify that the **ocs-storagecluster-ceph-rdb-virtualization** class is set as the default. If it is not, click the three dots on the **StorageClass required** and select **Set as default**.
{: note}



If you configure Thanos with IBM Cloud Object Storage, the `thanos-object-storage.yaml` looks similar to the following example.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: thanos-object-storage
  namespace: open-cluster-management-observability
type: Opaque
stringData:
  thanos.yaml: |
    type: s3
    config:
      bucket: <bucket-name>
      endpoint: s3.us-east.cloud-object-storage.appdomain.cloud
      insecure: true
      access_key: XXXX
      secret_key: XXXX
```
{: codeblock}

Create service credentials with HMAC keys for the COS instance. Use the provided HMAC values for `access_key` and `secret_key`.
{: note}

## Setting up and configuring Red Hat Advanced Cluster Management Observability 
{: #roks-virt-observ-setup-config}
{: step}

Use the following information to set up and configure Red Hat Advanced Cluster Management Observability.

### System alerts overview
{: #roks-virt-observ-setup-config-alerting-overview}
{: step}

**Default system alerts** are managed by {{site.data.keyword.redhat_openshift_notm}} built-in monitoring stack at the platform-level.

- Scope: Single cluster infrastructure monitoring
- Namespace: `openshift-monitoring` on each managed cluster
- **Alert components**:
   - Cluster Monitoring Operator
   - Prometheus Operator
   - Prometheus for cluster metrics
   - Alertmanager for cluster alerts

You can set up **Custom alerts** if default system alerts. The Red Hat Advanced Cluster Management observability stack manages these alerts and supports multi-cluster scenarios.

- Scope: Multi-cluster application and custom resource monitoring
- Namespace: `open-cluster-management-observability` on the hub cluster
- Components:
   - Thanos Query for a global metrics view
   - Thanos Ruler for centralized alert evaluation
   - Alertmanager for multi-cluster alerting

### Metrics and dashboards
{: #roks-virt-observ-setup-config-metrics-dashboards}
{: step}

ACM provides a set of preconfigured dashboards that monitor virtual server-specific metrics. For example, in the **ACM/OpenShift Virtualization dashboards** folder, the **Executive dashboards and single virtual machine view dashboard** displays CPU, Memory, Network (Transmit and Receive, Packets Dropped), Storage (IOPS/Traffic). It also displays the file system usage for each virtual server.

#### Extra monitoring features
{: #observability-design-min-features}

ACM monitors essential metrics like CPU, memory, network, storage usage, pod, cluster node, or entire cluster. {{site.data.keyword.redhat_openshift_notm}} base observability monitors many of the same metrics, but ACM extends monitoring across multiple managed clusters.

For a brief overview of the metrics that are displayed in each pre-configured provided by ACM and base {{site.data.keyword.redhat_openshift_notm}} observability, refer to the following information:

#### Add and customize dashboards 
{: #observability-design-custom-dashboards}
{: step}

The default Grafana instance doesn't support creating dashboards. To create and customize dashboards, first create a `grafana-dev instance`. Follow the instructions that are in the [RHACM Observability documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#using-grafana-dashboards){: external} to set up the instance by using the scripts that are found in [multicluster-observability-operator](https://github.com/open-cluster-management/multicluster-observability-operator){: external}.

To create a dashboard, use PromQL queries from a wide selection of [KubeVirt Components Metrics](https://kubevirt.io/monitoring/metrics.html){: external} and other [Kubernetes Metrics](https://kubernetes.io/docs/reference/instrumentation/metrics/){: external}.

To export custom metrics from either [Platform](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#adding-platform-metrics){: external} or [User Workload](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#adding-user-workload-metrics){: external} sources, follow the instructions in the [Advanced Observability Configuration](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#adv-config-obs){: external} section.

View the list of available platforms or user workload metrics on the **Observe > Targets** page. Filter the **Source** by **Platform** for platform metrics or **User** for user workload metrics.
{: tip}

#### Right-sizing recommendation dashboard
{: #observability-design-dashboards-rightsize}

The right-sizing recommendation dashboard provides recommendations to optimize CPU and memory allocations that are based on resource usage. Enable the [Right-sizing](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#optimize-work-right-size){: external} feature in the MultiClusterObservability custom resource on the hub cluster to access the dashboard.

The dashboard provides the following information:

- CPU Recommendation: Recommended workload CPU allocation 
{: #observability-design-custom-alerts}
{: step}

You create custom alerts by defining a `PrometheusRule` custom resource in the **open-cluster-management-observability** namespace. The following example creates a custom alert that triggers when a pod CPU usage exceeds 50% for more than one minute:

When default alert rules do not cover a scenario, configure the custom alert rules.

Consider the following scenario: Trigger a Slack alert when a virtual machine in the **“default“namespace** is not in a running state. If ACM manages the local hub cluster, it monitors system namespaces and **default** namespaces by default. However, default alert rules might not cover this scenario. So, you must set up a custom alert rule in ACM to trigger an alert based on the custom rule. Follow these steps:

1. Verify whether the Prometheus metrics `kubevirt_vm_info` exist or not. Log in to `observability-thanos-query` pod under **namespace “open-cluster-management-observability“**, and run the following curl command:

    `curl -s 'http://localhost:9090/api/v1/query' --data-urlencode 'query=kubevirt_vm_info'`, and make sure that the kubevirt_vm_info metrics exist there.

    ```bash
    $ curl -s 'http://localhost:9090/api/v1/query' --data-urlencode 'query=kubevirt_vm_info'

    $ curl -s 'http://localhost:9090/api/v1/query' --data-urlencode 'query=kubevirt_vm_info'

    $ curl -s 'http://localhost:9090/api/v1/query' --data-urlencode 'query=kubevirt_vm_info'

    {"status":"success","data":{"resultType":"vector","result":[{"metric":{"__name__":"kubevirt_vm_info","cluster":"local-cluster","clusterID":"110b2032-b6b9-455a-abd5-9949482b61df","container":"virt-controller","endpoint":"metrics","flavor":"small","instance":"10.129.0.198:8443","job":"kubevirt-prometheus-metrics","machine_type":"pc-q35-rhel9.6.0","name":"centos-stream9-white-mink-62","namespace":"acm-test","os":"centos-stream9","pod":"virt-controller-7744b59dd8-dk7tw","receive":"true","service":"kubevirt-prometheus-metrics","status":"running","status_group":"running","tenant_id":"2c1cf409-966c-4c2d-bd57-f24c4c5a9393","workload":"server"},"value":[1762574206.765,"1"]},{"metric":{"__name__":"kubevirt_vm_info","cluster":"local-cluster","clusterID":"110b2032-b6b9-455a-abd5-9949482b61df","container":"virt-controller","endpoint":"metrics","instance":"10.129.0.198:8443","instance_type":"u1.medium","job":"kubevirt-prometheus-metrics","machine_type":"pc-q35-rhel9.6.0","name":"rhel-10-rose-damselfly-10","namespace":"acm-test","pod":"virt-controller-7744b59dd8-dk7tw","preference":"rhel.10","receive":"true","service":"kubevirt-prometheus-metrics","status":"running","status_group":"running","tenant_id":"2c1cf409-966c-4c2d-bd57-f24c4c5a9393"},"value":[1762574206.765,"1"]}],"analysis":{}}}
     ```
    {: codeblock}

1. To apply the custom alert rule, create and **Apply the configmap** to the cluster.

    ```bash
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: thanos-ruler-custom-rules
        # need to ensure namespace is "open-cluster-management-observability"
      namespace: open-cluster-management-observability
      labels:
        # need this label to ensure custom alert rules are automatically mounted into Thanos Ruler pods
        thanos-ruler-rule: "true"
    data:
      # name has to be custom_rules.yaml, custom alert rules are automatically mounted into Thanos Ruler pods successfully
      custom_rules.yaml: |
        groups:
          - name: VirtualMachineAlerts
            interval: 30s
          rules:
            - alert: VirtualMachineNotRunning
            expr: kubevirt_vm_info{status!="running"} > 0
            for: 2m
            annotations:
              summary: "VM {{ $labels.name }} is Not in running status on {{ $labels.cluster }}"
              description: "VM {{ $labels.name }} in namespace {{ $labels.namespace }} is not running."
          labels:
            severity: critical
            alert_type: vm_not_running
      ```
      {: codeblock}

    Use the existing Prometheus metrics `kubevirt_vm_info` and PromQL `kubevirt_vm_info{status!="running"} > 0` to trigger an alert when any virtual server isn't in the running status.

    To help ensure that Thanos Ruler pods automatically mount the custom alert rules, you must meet the following conditions:
    
    For scenarios that default alerting rules can't cover, you need to set up custom alert rules.
      - The **configmap** must be labeled with: `thanos-ruler-rule: "true"`
      - The **key** must be set to `"custom_rules.yaml"`

    The `alert_type: vm_not_running` must match the secret that you create in the next step.

1. Apply the **configmap** by running `oc -n open-cluster-management-observability apply -f thanos-ruler-custom-rules.yaml`. Then, validate the output `/etc/thanos/rules/thanos-ruler-custom-rules from thanos-ruler-custom-rules` to confirm that the pod `observability-thanos-rule` mounts the custom rule.

  ```bash
      oc describe pod observability-thanos-rule-0

     Mounts:

        /etc/thanos/config/thanos-ruler-config from thanos-ruler-config (ro)
  ```
  {: codeblock}

1. Create and apply Apply the configmap to the cluster, for the custom alert rule to be applied.

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: thanos-ruler-custom-rules
  # need to ensure namespace is "open-cluster-management-observability"
  namespace: open-cluster-management-observability
  labels:
    # need this label to ensure custom alert rules are automatically mounted into Thanos Ruler pods
    thanos-ruler-rule: "true"
data:
  # name has to be custom_rules.yaml, custom alert rules are automatically mounted into Thanos Ruler pods successfully
  custom_rules.yaml: |
    groups:
    - name: VirtualMachineAlerts
      interval: 30s
      rules:
      - alert: VirtualMachineNotRunning
        expr: kubevirt_vm_info{status!="running"} > 0
        for: 2m
        annotations:
          summary: "VM {{ $labels.name }} is Not in running status on {{ $labels.cluster }}"
          description: "VM {{ $labels.name }} in namespace {{ $labels.namespace }} is not running."
        labels:
          severity: critical
          alert_type: vm_not_running
```
{: codeblock}

```
   /etc/thanos/configmaps/Alertmanager-ca-bundle from Alertmanager-ca-bundle (rw)

   /etc/thanos/rules/thanos-ruler-custom-rules from thanos-ruler-custom-rules (rw)

   /etc/thanos/rules/thanos-ruler-default-rules from thanos-ruler-default-rules (rw)

   /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-c55nd (ro)

   /var/thanos/rule from data (rw)
```
    {: codeblock}

1. Create a secret yaml file that is named `Alertmanager-config.yaml`, which configures the receiver, and Slack webhook URL, Slack channel, and so on.

Make sure that the `alert_type` matches the value that is defined in the **configmap** that you create later.
{: important}

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: Alertmanager-config
      # namespace has to be open-cluster-management-observability
      namespace: open-cluster-management-observability
    type: Opaque
    stringData:
    Alertmanager.yaml: |
      global:
        resolve_timeout: 5m
        # your slack webhook url
        slack_api_url: 'https://hooks.slack.com/services/\<replace_with_your_webhook\>'

    route:
      receiver: 'default-receiver'
      group_by: ['alertname', 'cluster', 'namespace', 'name']
      group_wait: 10s
      group_interval: 5m
      repeat_interval: 12h

      # Specific sub-route for VM not running alerts
      routes:
        - match:
            # ensure the alert_type is consistent with the one defined in configmap
            alert_type: vm_not_running
          receiver: 'slack-vm-not-running'
          group_wait: 5s
          repeat_interval: 10m

    receivers:
      # default receiver (no-op)
      - name: 'default-receiver'

      # Slack receiver for VM not running alerts
      - name: 'slack-vm-not-running'
        slack_configs:
          # slack channel name, do NOT miss "#" before you type the channel name
          - channel: '#roks-virt-observability-alerts'   # <-- adjust to your Slack channel
            title: '🚨 VM NOT RUNNING ALERT'
            text: |
              *Alert:* {{ .CommonLabels.alertname }}
              *VM Name:* {{ .CommonLabels.name }}
              *Namespace:* {{ .CommonLabels.namespace }}
              *Cluster:* {{ .CommonLabels.cluster }}
              *Severity:* {{ .CommonLabels.severity | toUpper }}
              *Status:* {{ .Status | toUpper }}

              {{ range .Alerts }}
              *Description:* {{ .Annotations.description }}
              {{ end }}
            send_resolved: true
            color: '{{ if eq .Status "firing" }}danger{{ else }}good{{ end }}'
            actions:
              - type: button
                text: 'View in Console'
                url: 'https://console-openshift-console.apps.{{ .CommonLabels.cluster }}/k8s/ns/{{ .CommonLabels.namespace }}/virtualmachines/{{ .CommonLabels.name }}'
    ```
{: codeblock}

1. Apply the secret by running `oc -n open-cluster-management-observability apply -f Alertmanager-config.yaml`.
1. Stop the virtual server by running:  virtctl stop `<vm-name>` and wait until the server stops.
1. After the server stops, check the Slack channel for the notification of the resolved alert.

   You can verify the generated alert through Alertmanager console. To open the Alertmanager console, run the following command:

   ```bash
       oc -n open-cluster-management-observability port-forward pod/observability-Alertmanager-0 9093:9093
    ```
   {: codeblock}

   Access http://localhost:9093/#/alerts to check whether the alert is generated there. The following example shows that the `slack-vm-not-running` alert is generated.

1. Run virtctl start `<vm-name>` and wait until the VM starts.
1. Start a virtual machine by running:  virtctl start `<vm-name>`, and wait for the vm started.
1. After the VM starts, check the Slack channel for the notification of the resolved alert.
   You can also verify that the alert is resolved through the Alertmanager console. Access [localhost alerts](http://localhost:9093/#/alerts) to confirm whether the alert is resolved. The following example shows that the `slack-vm-not-running` alert is resolved.

##### Observability references
{: #observability-design-ref-links}

See the following links for extra Observability information.

- For more information about configuring Alertmanager, see [Configure the Alertmanager in ACM](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html/observability/observing-environments-intro#configuring-Alertmanager){: external}
- For more information about creating custom rules, see [How to create a custom rules](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html/observability/observing-environments-intro#creating-custom-rules){: external}
- For more information about Kubervirt metrics, see [Kubevirt related prometheus metrics](https://kubevirt.io/monitoring/metrics.html){: external}

## Next steps
{: #observability-next-steps}

After you implement Observability for your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment, consider the following information. 

- Backup and recovery: [Implement backup solutions for your virtual servers](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-backup)
- Migration: [Learn about migrating workloads to Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-migration-design-mtv)
- Design considerations: Review comprehensive design guidance for production deployments:
   - [Compute design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-compute-design)
   - [Networking design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-network-design)
   - [Storage design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-storage-design-overview)
   - [Security design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-security-design-overview)
   - [Resiliency design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-resiliency-design)
- Reference architecture: [Explore the complete reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-rove-architecture)
