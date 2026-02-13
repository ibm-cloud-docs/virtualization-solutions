---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-09"

keywords: IBM Cloud Monitoring, IBM Cloud Logs, Workload Protection

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Observability Design for VPC virtual servers
{: #virt-sol-vpc-observability-design-overview}

Observability in IBM Cloud provides the visibility and insights needed to monitor, troubleshoot, and optimize applications and infrastructure across hybrid and multicloud environments. It goes beyond traditional monitoring by offering end-to-end visibility into metrics, logs, and traces, enabling proactive detection of issues and faster root-cause analysis.

IBM Cloud observability solutions include IBM Cloud Monitoring and IBM Cloud Logs, which together deliver a comprehensive view of system health and performance. These services help organizations ensure application reliability, security compliance, and operational efficiency by providing real-time dashboards, alerting, and deep analytics.

The key observability architecture elements are shown in the following diagram.

![IBM Cloud VPC VSI Observability](../../images/vpc-vsi/vpc-vsi-high-level-observability.svg "IBM Cloud VPC VSI Observability"){: caption="IBM Cloud VPC VSI Observability" caption-side="bottom"}

## IBM Cloud Security and Compliance Center Workload Protection
{: #virt-sol-vpc-observability-design-scc-wpp}

IBM Cloud Security and Compliance Center Workload Protection provides comprehensive security monitoring and threat detection for workloads running on IBM Cloud, including virtual machines on VPC and OpenShift Virtualization environments.

The Workload Protection agent discovers and prioritizes software vulnerabilities, detects and responds to runtime threats, and manages configurations, permissions, and compliance requirements for hosted virtual machines and containerized workloads.

See [Getting started with IBM Cloud Security and Compliance Center Workload Protection](/docs/workload-protection?topic=workload-protection-getting-started)

### Deployment and Capabilities
{: #virt-sol-vpc-observability-design-scc-wpp-deployment}

To enable Workload Protection, provision an instance of the IBM Cloud Security and Compliance Center Workload Protection service in IBM Cloud. After provisioning, deploy the agent to collect security and compliance data across your infrastructure.

The following table details the capabilities the agent provides.

| Feature | Description |
| -------------- | -------------- |
| Vulnerability scanning | Identify security vulnerabilities in images, packages, and applications |
| Intrusion detection | Detect runtime threats and anomalous behavior |
| Posture management | Validate security configurations and compliance policies |
| Incident response | Investigate and respond to security events with forensic data |
| Compliance validation | Assess compliance against regulatory frameworks and industry standards |
{: caption="IBM Cloud Security and Compliance Center Workload Protection features" caption-side="bottom"}

This unified approach enables organizations to accelerate hybrid cloud adoption while addressing security and regulatory compliance requirements across cloud, on-premises, virtual machines, containers, and Kubernetes environments.

See [Protecting Linux hosts](/docs/workload-protection?topic=workload-protection-protecting-linux-hosts) and [Managing the Workload Protection agent on Windows Servers](/docs/workload-protection?topic=workload-protection-agent-deploy-windows)

## IBM Cloud Monitoring and Logs
{: #virt-sol-vpc-observability-design-mon-and-logs}

IBM Cloud Monitoring and IBM Cloud Logs provide cloud-native observability for applications and infrastructure running on IBM Cloud, including virtual machines on VPC and OpenShift Virtualization.

| Service | Description | Agent deployment metrics collection |
| -------------- | -------------- | -------------- |
| IBM Cloud Monitoring | IBM Cloud Monitoring is a cloud-native, container-intelligence management system that provides operational visibility into the performance and health of applications, services, and platforms. It offers administrators, DevOps teams, and developers full-stack telemetry with advanced features for monitoring, troubleshooting, alerting, and custom dashboard creation. | To monitor infrastructure, networks, and applications, deploy Monitoring agents on supported hosts. The agent type depends on the host platform and determines which metrics are automatically collected. When a Monitoring agent is configured, default metrics are collected automatically, including metadata for labeling, segmentation, and filtering. <br><br> No additional instrumentation is required to gain insights from automatically collected metrics. <br><br> For more information, see [Getting started with IBM Cloud Monitoring](/docs/monitoring?topic=monitoring-getting-started), [Monitoring a Windows environment](/docs/monitoring?topic=monitoring-windows) and [Monitoring an Ubuntu Linux VPC server instance](/docs/monitoring?topic=monitoring-ubuntu). |
| IBM Cloud Logs | IBM Cloud Logs is an observability service designed to help organizations monitor, troubleshoot, analyze, and alert on application and infrastructure performance in real time and over extended periods. By collecting and analyzing logs from cloud-native applications, servers, databases, and IT systems, IBM Cloud Logs provides actionable insights into system behavior. | IBM Cloud Logs supports log collection from: <br> - IBM Cloud services and resources <br> - On-premises infrastructure <br> - Third-party cloud providers <br> - Security and audit logs generated in IBM Cloud <br><br> To monitor infrastructure, networks, and applications, deploy Monitoring agents on supported hosts. The agent type depends on the host platform and determines which metrics are automatically collected. When a Monitoring agent is configured, default metrics are collected automatically, including metadata for labeling, segmentation, and filtering. No additional instrumentation is required to gain insights from automatically collected metrics. <br><br> For more information, see [Getting started with IBM Cloud Monitoring](/docs/monitoring?topic=monitoring-getting-started), [Monitoring a Windows environment](/docs/monitoring?topic=monitoring-windows) and [Monitoring an Ubuntu Linux VPC server instance](/docs/monitoring?topic=monitoring-ubuntu). |
{: caption="IBM Cloud Monitoring and IBM Cloud Logs details" caption-side="bottom"}

Be aware that for IBM Cloud Linux VSI and IBM Cloud Windows VSI	both Service ID API key and Trusted Profiles authentication methods are supported by the agent with the IBM Cloud Logs service.

### Combined Observability Benefits
{: #virt-sol-vpc-observability-design-combined-benefits}

IBM Cloud uses a single unified agent that can collect both security data (for Workload Protection) and metrics data (for Cloud Monitoring). Key points:

* Multiple instances of the agent cannot be deployed on the same host, but by creating a connection between instances, a single agent can collect both security and metrics data
* You can connect only one Monitoring instance to one Workload Protection instance, and both instances must be in the same region

The following table details the unified agent components.

| Component | Description |
| -------------- | -------------- |
| For Monitoring (Metrics) | - Agent: Collects metrics from containers, pods, nodes, and Kubernetes resources  \n - Prometheus integration: Custom metrics collection  \n  - Cluster metadata: Automatic tagging with cluster name and context
| For Workload Protection (Security) | - Node Analyzer: Includes host scanner and KSPM (Kubernetes Security Posture Management) analyzer  \n Host Scanner: Detects vulnerabilities and identifies resolution priority based on available fixed versions and severity  \n - KSPM Analyzer: Kubernetes Security Posture Management for compliance and configuration analysis  \n - Cluster Shield: Security runtime component |
{: caption="Components of unified agent" caption-side="bottom"}

The following table details the comprehensive observability provided when deploying both the unified agent (for IBM Cloud Monitoring and Workload Protection) and the IBM Cloud Logs agent in VPC virtual server instances.

| Observability | Description |
| -------------- | -------------- |
| Full-stack visibility | Monitor from infrastructure through application layers |
| Correlated insights | Correlate metrics and logs for faster root cause analysis |
| Unified dashboards | View metrics and logs in integrated IBM Cloud console |
| Custom alerting | Configure alerts based on metric thresholds and log patterns |
| Long-term retention | Store historical data for trend analysis and compliance |
| Centralized management | Manage observability across hybrid and multicloud environments from a single platform |
| Vulnerability scanning | Identify security vulnerabilities in images, packages, and applications |
| Intrusion detection | Detect runtime threats and anomalous behavior |
| Posture management | Validate security configurations and compliance policies |
| Incident response | Investigate and respond to security events with forensic data |
| Compliance validation | Assess compliance against regulatory frameworks and industry standards |
{: caption="Observability provided by unified agent" caption-side="bottom"}

## Next steps
{: #virt-sol-vpc-observability-design-next-steps}

Now that you understand the observability design for VPC virtual servers, explore these related topics:

- **Security**: Review [security design considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-security-design-overview) including compliance monitoring
- **Resiliency**: Learn about [backup and disaster recovery](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vpc-resiliency-design) strategies
- **Networking**: Explore [networking design patterns](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-design) for VPC connectivity
- **Reference architecture**: Review the complete [VPC virtual server reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vsi-architecture)
