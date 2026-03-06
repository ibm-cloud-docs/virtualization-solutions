---

copyright:
  years: 2026
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

# Observability with Red Hat Advance Cluster Management
{: #rhacm-overview}
{: #vsphere-openshift-rhacm}
{: #tutorial-rhacm-overview}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}
{: toc-use-case="ApplicationModernization"}
{: toc-industry="Software and platform applications"}
{: toc-compliance="HIPPA"}

Learn how to implement comprehensive observability for Red Hat OpenShift Virtualization environments, including monitoring, logging, alerting, and metrics collection using Red Hat Advanced Cluster Management (ACM).
{: shortdesc}


## Red Hat OpenShift Virtualization
{: #roks-virt-observ}

### General Introduction
{: #roks-virt-observ-intro}


Red Hat Advanced Cluster Management (RHACM) provides the ability to manage and control multiple clusters under a single OpenShift instance. This allows for such capabilities as remote clustering, workloads across nodes and regions, disaster recovery and more. Each managed cluster will report back to the designated hub cluster. Often this will mean the hub cluster contains the control-plane nodes while the managed clusters are all composed of worker nodes. A klusterlet is installed on each managed cluster and is used to control the communication to the hub cluster. The term local cluster is used for a hub cluster that is also a managed cluster.

RHACM is installed through a separate operator on a Red Hat OpenShift instance, however note that this does require licensing and subscription agreement. Once installed, a number of multi-cluster features appear with some additional configuration to further expand the capabilities.

By default, Red Hat OpenShift provides a number of observability options. Without additional operators some basic alerting, monitoring, and logging are available. OpenShift provides a Prometheus-based platform that initially focuses on critical functionality of the cluster but can be expanded with customization. Some examples of the components that are created to handle observability:

- Cluster Monitoring Operator (CMO) - overall manager of the observability components
- Prometheus - monitors data received from CMO and creates a database engine for metrics
- Metrics Server - collects metrics and sets up API service for Thanos
- Thanos Querier - provides interface for querying metrics collected
- Alertmanager - receives alerts from Prometheus and can send externally

These are automatically deployed under the openshift-monitoring namespace during OCP deployment. The dashboards and configuration of these are available under the **Observe section** in the Administration view.

Alerting will display the current alerts, alerting rules, and the viewing/addition of any alert silencing rules. The Metrics page allows for the user to run PromQL queries and receive metrics data and graphs. Dashboards will display metric graphs for a preset list that can be selected from multiple dropdowns and different time periods. Targets is the defined list of endpoints that are called by the Metrics Server to receive data from different components. The Logs section will only appear after installing and configuring OpenShift Logging (see further below in this guide). [For more info on the design and components]( https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/monitoring/about-openshift-container-platform-monitoring#monitoring-stack-architecture) {: external}


RHACM adds additional observability functionality focused on the support of multiple host clusters. This largely expands existing Prometheus, Thanos, and Alertmanager infrastructure but with a few new pieces:

- Multicluster observability operator - manager of the monitoring and metrics gathering of managed clusters
- Grafana - new customizable dashboard for metrics including additional node and virtual machine metrics
- Observability add-on controller - new API server that handles the managed clusters
- Thanos Compactor - allows for long-term saving of metrics data

The observability feature of RHACM must be enabled after the RHACM operator has been installed as it is not enabled by default. A heavy focus of this guide is assuming ACM observability is enabled, especially with regard to metrics and dashboards.

For further information on [RHACM Observability architecture](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#observing-environments-intro_observability){: external}


### Installing Red Hat Advanced Cluster Management
{: #roks-virt-observ-installing}
{: step}

1. Log in to your OCP instance.
2. Under Ecosystem → Software Catalog.
3. Pick a namespace.
4. In the search look for Advanced Cluster Management for Kubernetes
5. Select install if not already installed
6. Once installed, pull it up under Ecosystem →  Installed Operators
7. Under MultiClusterHub tab, click Create MultiClusterHub
8. Give it an appropriate name and click Create
9. Go to Storage → StorageClasses
10. Make sure a StorageClass is set to default for deploying the infrastructure.
11. Follow the steps to [enable the Observability modules of RHACM](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#enabling-observability){: external}
12. Once the Observability modules are enabled, go to the Observability tab under the MultiClusterHub
13. Click on the Observability tab and click Create Observability
14. Give it an appropriate name and click Create
15. Once the Observability instance is created, go to the Observability tab and click on the Observability instance

If using ODF, make sure the “ocs-storagecluster-ceph-rdb-virtualization” class is set to the default. If it is not, click the 3 dots on the StorageClass desired and choose “Set as default”
{: note}

If configuring Thanos with IBM COS, the thanos-object-storage.yaml will look something like

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

Service credentials should be created HMAC keys in the COS instance. The values for access_key and secret_key will be the HMAC ones provided.
{: note}

### Setup and Configuration of Red Hat Advanced Cluster Management Observability
{: #roks-virt-observ-setup-config}
{: step}

#### Alerting
{: #roks-virt-observ-setup-config-alerting}
{: step}

#### Overview
{: #observability-design-overview}

**Default System Alerts** are managed by OpenShift's built-in monitoring stack and cover platform-level concerns:

- **Scope**: Single cluster infrastructure monitoring
- **Namespace**: `openshift-monitoring` on each managed cluster
- **Components**:
    - Cluster Monitoring Operator
    - Prometheus Operator
    - Prometheus (for cluster metrics)
    - AlertManager (for cluster alerts)

**Custom Alerts** can be set up if default system alerts cannot cover your business scenario. These are managed by Red Hat Advanced Cluster Management (RHACM) observability stack and support multi-cluster scenarios:

- **Scope**: Multi-cluster application and custom resource monitoring
- **Namespace**: `open-cluster-management-observability` on the hub cluster
- **Components**:
    - Thanos Query (global metrics view)
    - Thanos Ruler (centralized alert evaluation)
    - AlertManager (multi-cluster alerting)

### Metrics and Dashboards
{: #roks-virt-observ-setup-config-metrics-dashboards}
{: step}

ACM provides a set of pre-configured dashboards that are designed to monitor VM specific metrics. For instance, within the “ACM / OpenShift Virtualization” dashboards folder, the “Executive dashboards / Single Virtual Machine View” dashboard displays CPU Usage, Memory Usage, Network Usage (Transmit/Receive, Packets Dropped), Storage Usage (IOPS/Traffic), and Filesystem Usage for an individual VM.

#### Additional monitoring features
{: #observability-design-min-features}

ACM can be used to monitor essential metrics like CPU, memory, network, storage usage for a VM, pod, cluster node, or an entire cluster. Most of the same metrics can be monitored with the base observability included in OpenShift, but ACM allows the user to monitor these metrics across multiple managed clusters.

ACM adds pre-configured Grafana dashboards designed to monitor VM specific metrics. For instance, within the “ACM / OpenShift Virtualization” dashboards folder, the “Executive dashboards / Single Virtual Machine View” dashboard displays CPU Usage, Memory Usage, Network Usage (Transmit/Receive, Packets Dropped), Storage Usage (IOPS/Traffic), and Filesystem Usage for an individual VM.

For a brief assessment of the metrics displayed in every pre-configured dashboard provided through ACM as well as the base OpenShift observability, refer to this chart:

#### Adding/Customizing Dashboards
{: #observability-design-custom-dashboards}
{: step}

New dashboards can not be added with the default grafana instance, but new dashboards can be created and customized by first creating a grafana-dev instance. This can be done by following the instructions outlined in the [RHACM Observability Documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#using-grafana-dashboards) {: external) for setting up the grafana dev instance using the scripts found in https://github.com/open-cluster-management/multicluster-observability-operator.

New dashboards can be created using PromQL queries from a wide selection of [KubeVirt Components Metrics](https://kubevirt.io/monitoring/metrics.html) and other [Kubernetes Metrics](https://kubernetes.io/docs/reference/instrumentation/metrics/).

Custom metrics can be exported from either [Platform](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#adding-platform-metrics) or [User Workload](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#adding-user-workload-metrics) sources as described in the [Advanced Observability Configuration](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#adv-config-obs) section. A list of available platform or user workload metrics targets can be viewed by navigating to the Observe → Targets page and filtering the Source by Platform or User, respectively.

#### RightSizing Recommendation Dashboard
{: #observability-design-dashboards-rightsize}

The RightSizing Recommendation Dashboard provides recommendations for optimizing CPU and Memory allocations based on resource usage. The dashboard is available if the [Right-Sizing](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html-single/observability/index#optimize-work-right-size) feature is enabled in the MultiClusterObservability custom resource on the hub cluster.

The dashboard provides the following information:
- **CPU Recommendation**: The recommended CPU allocation for each workload.
- **Memory Recommendation**: The recommended memory allocation for each workload.
- **Current CPU Usage**: The current CPU usage for each workload.
- **Current Memory Usage**: The current memory usage for each workload.
- **CPU Savings**: The estimated CPU savings if the recommended CPU allocation is applied.
- **Memory Savings**: The estimated memory savings if the recommended memory allocation is applied.

#### Creating System Alerts
{: #observability-design-system-alerts}
{: step}

For System alerts (handled by “openshift-monitoring“ namespace), you can following the below steps to create alerting, which supports integration with Pagerduty, Slack, Email, etc.

1. Login to your OCP instance
2. Under Administration → Cluster Settings → Alertmanager
3. Either configure one of the existing Receivers listed at the bottom table or click Create Receiever
4. Give an appropriate name
5. Under Receiver type select one from the dropdown
    1. If selecting PagerDuty, you will need a routing key and URL
    2. If selecting Email you will need an SMTP server
    3. If selecting Slack you will need the webhook URL and channel name (can be a username for direct DMs) with no @ symbol
6. For Routing labels you can select based on which alerts you want to fire out a notification
    1. For example, “severity = critical” will be all critical alerts, and “severity = warning“ will be all warning alerts
    2. You can leave this field blank to have all alerts send out a notification.

You can click “Alerting rules“ tab, which will show you the existing default alerting rules to cover the system alerts.  Take alerting rules for ceph storage cluster as an example, as following shows, which means there should be alert triggered if the storage usage is higher than 75%.

#### Creating Custom Alerts
{: #observability-design-custom-alerts}
{: step}


You can create custom alerts by creating a `PrometheusRule` custom resource (CR) in the **open-cluster-management-observability** namespace. The following example creates a custom alert that fires when the CPU usage of a pod is greater than 50% for more than 1 minute:

For scenarios where default alerting rules could not cover, you need to setup custom alerting rules.

Take this scenario as an example: Firing a slack alert if a virtual machine created under “default“ namespace (if the local hub cluster is managed by ACM, ACM can monitor system namespaces and “default” namespace by default) is NOT in running state, this is something which could not be covered by default alerting rules, and we have to setup a custom alerting rule in ACM side, so that ACM can help fire an alert based on the custom rule. Basically you need to follow the below steps:


1. Verify if the Prometheus metrics “kubevirt_vm_info“ exists or not. Login to “observability-thanos-query“ pod under namespace “open-cluster-management-observability“, and run the following curl command: `curl -s 'http://localhost:9090/api/v1/query' --data-urlencode 'query=kubevirt_vm_info'`, and ensure kubevirt_vm_info metrics exist there.

```bash
$ curl -s 'http://localhost:9090/api/v1/query' --data-urlencode 'query=kubevirt_vm_info'

{"status":"success","data":{"resultType":"vector","result":[{"metric":{"__name__":"kubevirt_vm_info","cluster":"local-cluster","clusterID":"110b2032-b6b9-455a-abd5-9949482b61df","container":"virt-controller","endpoint":"metrics","flavor":"small","instance":"10.129.0.198:8443","job":"kubevirt-prometheus-metrics","machine_type":"pc-q35-rhel9.6.0","name":"centos-stream9-white-mink-62","namespace":"acm-test","os":"centos-stream9","pod":"virt-controller-7744b59dd8-dk7tw","receive":"true","service":"kubevirt-prometheus-metrics","status":"running","status_group":"running","tenant_id":"2c1cf409-966c-4c2d-bd57-f24c4c5a9393","workload":"server"},"value":[1762574206.765,"1"]},{"metric":{"__name__":"kubevirt_vm_info","cluster":"local-cluster","clusterID":"110b2032-b6b9-455a-abd5-9949482b61df","container":"virt-controller","endpoint":"metrics","instance":"10.129.0.198:8443","instance_type":"u1.medium","job":"kubevirt-prometheus-metrics","machine_type":"pc-q35-rhel9.6.0","name":"rhel-10-rose-damselfly-10","namespace":"acm-test","pod":"virt-controller-7744b59dd8-dk7tw","preference":"rhel.10","receive":"true","service":"kubevirt-prometheus-metrics","status":"running","status_group":"running","tenant_id":"2c1cf409-966c-4c2d-bd57-f24c4c5a9393"},"value":[1762574206.765,"1"]}],"analysis":{}}}
```
{: codeblock}

2. Create and apply Apply the configmap to the cluster, for the custom alerting rule to be applied.

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

The essence is to leverage the existing Prometheus metrics "kubevirt_vm_info" and PromQL `kubevirt_vm_info{status!="running"} > 0`, which can fire an alert if it detects any virtual machines that are not in the running status.

To ensure the custom alert rules are automatically mounted into Thanos Ruler pods successfully, the following conditions must be met:
- The configmap must be labeled with: `thanos-ruler-rule: "true"`
- The "key" must be set to `"custom_rules.yaml"`
{: important}

The `alert_type: vm_not_running` must match with the secret that will be created in the next step.
{: note}

3.  Apply the configmap by running “oc -n open-cluster-management-observability apply -f thanos-ruler-custom-rules.yaml“Then validate the output “/etc/thanos/rules/thanos-ruler-custom-rules from thanos-ruler-custom-rules “, which means the custom rule has been mounted to the pod  “observability-thanos-rule“.

```bash
oc describe pod observability-thanos-rule-0

Mounts:

      /etc/thanos/config/thanos-ruler-config from thanos-ruler-config (ro)

      /etc/thanos/configmaps/alertmanager-ca-bundle from alertmanager-ca-bundle (rw)

      /etc/thanos/rules/thanos-ruler-custom-rules from thanos-ruler-custom-rules (rw)

      /etc/thanos/rules/thanos-ruler-default-rules from thanos-ruler-default-rules (rw)

      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-c55nd (ro)

      /var/thanos/rule from data (rw)
```
{: codeblock}

4. Create a secret yaml file named as alertmanager-config.yaml, which configures receiver, and  slack webhook url, slack channel, etc.

Attention: ensure the alert_type is consistent with the one defined in the configmap which will be created later.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-config
  # namespace has to be open-cluster-management-observability
  namespace: open-cluster-management-observability
type: Opaque
stringData:
  alertmanager.yaml: |
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

5. Apply the secret by running: oc -n open-cluster-management-observability apply -f alertmanager-config.yaml

6. Stop a virtual machine by running:  virtctl stop `<vm-name>`, and wait for the vm stopped
7. Check slack channel for the corresponding alert message.

You can verify the generated alert through alert manager console too. To open the alert manager console, you can run:

```bash
oc -n open-cluster-management-observability port-forward pod/observability-alertmanager-0 9093:9093
```
{: codeblock}

You can verify the alert is generated through the alter manager console. Access http://localhost:9093/#/alerts, to check if the alert is generated there. An example is as following, which shows “slack-vm-not-running“ alert is generated.

8. Start a virtual machine by running:  virtctl start `<vm-name>`, and wait for the vm started.

Once vm get started back, check the slack channel for the notification of the alert is resolved.

You can verify the alert is resolved through alert manager console too. Access http://localhost:9093/#/alerts, to check if the alert is resolved there. An example is as following, which shows “slack-vm-not-running“ alert is resolved.

##### Reference links:
{: #observability-design-ref-links}

- [Configure alert manager in ACM](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html/observability/observing-environments-intro#configuring-alertmanager) {: external}

- [How to create a custom rules](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.14/html/observability/observing-environments-intro#creating-custom-rules) {: external}

- [Kubevirt related prometheus metrics](https://kubevirt.io/monitoring/metrics.html) {: extenral}

## Next steps
{: #observability-next-steps}

After implementing observability for your Red Hat OpenShift Virtualization environment, consider these next steps:

- **Backup and Recovery**: Implement backup solutions for your virtual machines. See [Backup solution for Red Hat OpenShift Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-backup-solution).
- **Migration**: Learn about migrating workloads to OpenShift Virtualization. See [Migration Toolkit for Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-migration-toolkit).
- **Design Considerations**: Review comprehensive design guidance for production deployments:
    - [Compute design](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-compute-design)
    - [Networking design](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-networking-design)
    - [Storage design](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-storage-design)
    - [Security design](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-security-design)
    - [Resiliency design](/docs/virtualization-solutions?topic=virtualization-solutions-openshift-resiliency-design)
- **Reference Architecture**: Explore the complete reference architecture. See [Red Hat OpenShift Virtualization reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-reference-architecture-openshift).
