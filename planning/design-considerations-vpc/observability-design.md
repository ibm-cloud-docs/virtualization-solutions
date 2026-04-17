---
copyright:
  years: 2026, 2026
lastupdated: "2026-04-17"
keywords: IBM Cloud Monitoring, IBM Cloud Logs, Workload Protection
subcollection: virtualization-solutions
---
{{site.data.keyword.attribute-definition-list}}

# Observability design for Virtual Private Cloud (VPC) virtual server instances
{: #virt-sol-vpc-observability-design-overview}

Observability in {{site.data.keyword.cloud}} provides the visibility and insights that are needed to monitor, troubleshoot, and optimize applications and infrastructure across hybrid and multicloud environments. It extends beyond traditional monitoring by offering end-to-end visibility into metrics, logs, and traces, enabling proactive detection of issues and faster root-cause analysis.

{{site.data.keyword.cloud_notm}} observability solutions, including {{site.data.keyword.monitoringfull_notm}} and {{site.data.keyword.logs_full_notm}}, deliver a comprehensive view of system health and performance. These services help organizations ensure application reliability, maintain security compliance, and improve operational efficiency through real-time dashboards, alerting, and advanced analytics. The following diagram shows the key observability architecture elements.
![IBM Cloud VPC VSI Observability](../../images/vpc-vsi/vpc-vsi-high-level-observability.svg "IBM Cloud VPC VSI Observability"){: caption="IBM Cloud VPC VSI Observability" caption-side="bottom"}

## {{site.data.keyword.sysdigsecure_full_notm}}
{: #virt-sol-vpc-observability-design-scc-wpp}

{{site.data.keyword.sysdigsecure_full_notm}} provides security monitoring and threat detection for workloads on {{site.data.keyword.cloud_notm}}, including virtual machines on Virtual Private Cloud (VPC) and {{site.data.keyword.redhat_openshift_notm}} Virtualization environments.

The workload protection agent discovers and prioritizes software vulnerabilities, detects and responds to runtime threats, and manages configurations, permissions, and compliance requirements for hosted virtual machines and containerized workloads.
For more information, see [Getting started with {{site.data.keyword.sysdigsecure_full_notm}}](/docs/workload-protection?topic=workload-protection-getting-started)

### Deployment and capabilities
{: #virt-sol-vpc-observability-design-scc-wpp-deployment}

To enable workload protection, provision an instance of the {{site.data.keyword.sysdigsecure_full_notm}} service in {{site.data.keyword.cloud_notm}}. After provisioning, deploy the agent to collect security and compliance data across your infrastructure.
The following table details the capabilities that the agent provides.

| Feature | Description |
| -------------- | -------------- |
| Vulnerability scanning | Identify security vulnerabilities in images, packages, and applications |
| Intrusion detection | Detect runtime threats and anomalous behavior |
| Posture management | Validate security configurations and compliance policies |
| Incident response | Investigate and respond to security events with forensic data |
| Compliance validation | Assess compliance against regulatory frameworks and industry standards |
{: caption="{{site.data.keyword.sysdigsecure_full_notm}} features" caption-side="bottom"}

This unified approach enables organizations to accelerate hybrid cloud adoption while meeting security and regulatory compliance requirements across cloud, on-premises, virtual machines, containers, and Kubernetes environments.

See [Protecting Linux hosts](/docs/workload-protection?topic=workload-protection-protecting-linux-hosts) and [Managing the Workload Protection agent on Windows Servers](/docs/workload-protection?topic=workload-protection-agent-deploy-windows).

For {{site.data.keyword.vpc_short}} virtual server instance with an IBM&reg; provided Linux OS image, follow the guide in [Protecting Linux hosts](/docs/workload-protection?topic=workload-protection-protecting-linux-hosts) to install and deploy the workload protection agent and connect it to the workload protection instance.

For {{site.data.keyword.vpc_short}} virtual server instance with an {{site.data.keyword.IBM_notm}} provided Windows OS image, follow the guide in [Managing the workload protection agent on Windows Servers](/docs/workload-protection?topic=workload-protection-agent-deploy-windows) to complete the installation. However, some OS images that are provided by {{site.data.keyword.vpc_short}} virtual server instance have limitations. You might need to follow the best practices or workarounds to facilitate agent deployment. For images not covered in the following sections, the general guide works as expected.

For a customer-provided custom image that is uploaded to the {{site.data.keyword.vpc_short}} virtual server instance platform, first follow the default guide steps and then address any gaps individually. If necessary, open a support ticket for the workload protection instance.
{: note}

#### ibm-centos-stream-9-amd64-13
{: #virt-sol-vpc-observability-design-scc-vsi-centos9}

The default kernel version of the CentOS OS with this image does not have a corresponding `kernel-devel` package. As a result, the installation step in the guide, `sudo yum -y install kernel-devel-$(uname -r)`, fails. Use the following workaround:

- Run `sudo dnf list --showduplicates kernel-devel` to list the kernel patch versions for the current CentOS version. Then, run `sudo yum install kernel-<LATEST_KERNEL_PATCH_VERSION>` to upgrade the kernel version (for example, as of February 2026, use `5.14.0-665.el9.x86_64`, and the command to run it is `sudo yum install kernel-5.14.0-665.el9`). Then, restart the virtual server instance.
- Run `uname -r` to verify that the kernel version is updated. Follow the general guide to install the kernel-devel package, then download and install the agent to the virtual server instance, which must pass without issues.

#### ibm-centos-stream-10-amd64-5
{: #virt-sol-vpc-observability-design-scc-vsi-centos10}

The same issue that affects CentOS 9 also apply to the CentOS 10 image. In addition to the workaround described earlier, an extra issue affects the CentOS 10 OS agent installation: the required `dkms` package has a GPG key that is incompatible with CentOS 10, and no newer version provides a compatible one. Before you follow the general guide to download and install the agent, run `sudo dnf install --nogpgcheck dkms` to install `dkms` without a GPG verification.

#### ibm-debian-13-2-minimal-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-debian13}

The general guide mostly works for the Debian 13 OS image. However, you must run `sudo apt-get update` to update packages to the latest versions and avoid archive fetch failures, such as:

```sh
E: Failed to fetch http://mirrors.adn.networklayer.com/debian-security/pool/updates/main/c/curl/libcurl4_7.74.0-1.3%2bdeb11u15_amd64.deb  404  Not Found [IP: 161.26.0.6 80]
E: Failed to fetch http://mirrors.adn.networklayer.com/debian-security/pool/updates/main/c/curl/curl_7.74.0-1.3%2bdeb11u15_amd64.deb  404  Not Found [IP: 161.26.0.6 80]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```
{: pre}

#### ibm-debian-12-12-minimal-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-debian12}

The general guide mostly works for the Debian 12 OS image. However, you must run `sudo apt install curl` to install the `curl` package, which is required to download the agent installer.

#### ibm-debian-11-11-minimal-amd64-5
{: #virt-sol-vpc-observability-design-scc-vsi-debian11}

The issue that affects Debian 13 and Debian 12 also apply to the Debian 11 OS image. Before you follow the general guide, run `sudo apt-get update` and `sudo apt install curl` to address these gaps.

#### ibm-fedora-coreos-43-testing-1 and ibm-fedora-coreos-43-stable-1
{: #virt-sol-vpc-observability-design-scc-vsi-coreos43}

The Fedora CoreOS images that are offered by {{site.data.keyword.vpc_short}} virtual server instance use a read-only Fedora Bootc (immutable) system. According to the [Fedora CoreOS FAQ](https://docs.fedoraproject.org/en-US/fedora-coreos/faq/#_how_do_i_run_custom_applications_on_fedora_coreos){: external}.

Fedora CoreOS is primarily targeted for containers and discourages additional installation on the base OS. As a result, SCC integrations on virtual server instances using these images are not supported.

#### ibm-redhat-ai-nvidia-1-5-2-amd64-1 and ibm-redhat-ai-intel-1-5-1-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-redhatai}

Both Red Hat AI OS images are not registered with an entitled repository server because current Red Hat releases do not include the required internal infrastructure. They are offered as CoreOS images, and do not support custom application installation, similar to Fedora CoreOS.

It is unclear whether the repository is intentionally not attached to Red Hat AI to avoid kernel modification. However, with the currently supported agent installation method requires a repository. Therefore, Cloud Security and Compliance Center agent injection is not supported for these OS images until a repository is registered.

#### ibm-rocky-linux-9-6-minimal-amd64-3
{: #virt-sol-vpc-observability-design-scc-vsi-rocky9}

Issues affecting CentOS 9 also apply to Rocky Linux 9. Use the same workaround by running `sudo dnf list --showduplicates kernel-devel` to obtain the latest patch version and then running `sudo dnf install kernel-<LATEST_KERNEL_PATCH_VERSION>` to upgrade the kernel (for example, by February 2026, the latest patch version is `5.14.0-611.27.1.el9_7`, and the command to issue is `sudo dnf install kernel-5.14.0-611.27.1.el9_7`). After the kernel upgrade, restart the virtual server instance.

#### ibm-rocky-linux-10-minimal-amd64-3
{: #virt-sol-vpc-observability-design-scc-vsi-rocky10}

Issues affecting CentOS 10 also apply to Rocky Linux 10. After you upgrade the kernel to the latest patch version (for example, `6.12.0-124.31.1.el10_1` as of February 2026, and the command to issue is `sudo dnf install kernel-6.12.0-124.31.1.el10_1`) and restart the virtual server instance. Then, install the `dkms` package by running `sudo dnf install --nogpgcheck dkms` before you download and deploy the agent.

#### ibm-sles-16-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-sles16}

Similar issues that affect CentOS 9 also apply to the SUSE Linux Enterprise Server 16 image. Follow these steps as a workaround:

- Run `sudo zypper search -s kernel-*-devel` to list and obtain the latest kernel patch version (for example, `6.12.0-160000.9.1` as of February 2026).
- Run `sudo zypper install kernel-<LATEST_KERNEL_PATCH_VERSION>` to upgrade the kernel (for example, the command is `sudo zypper install kernel-default-6.12.0-160000.9.1` as of February 2026).
- Restart the virtual server instance.
- Run `uname -r` to make sure that the kernel is upgraded.
- Run `sudo zypper install kernel-devel` to install the `kernel-devel` package as a replacement for the first step in the general guide.
Then, follow the remaining steps.

#### ibm-sles-15-7-amd64-3
{: #virt-sol-vpc-observability-design-scc-vsi-sles157}

Issues affecting SUSE Linux Enterprise Server 16 also apply to the SUSE Linux Enterprise Server 15.7 image. In addition to the earlier steps (by February 2026, the kernel patch version to upgrade to `6.4.0-150700.53.31.1`), an extra issue must be resolved: the current SLES 15 image does not include the required local repositories for the `kernel-syms` package, which causes agent installation to fail. Before you download and install the agent, perform the following step:

- Run the following commands to configure two local repositories that contain the `kernel-syms` package:

```sh
sudo SUSEConnect -p sle-module-desktop-applications/15.7/x86_64
sudo SUSEConnect -p sle-module-development-tools/15.7/x86_64
```
{: pre}

#### ibm-sles-15-6-amd64-6
{: #virt-sol-vpc-observability-design-scc-vsi-sles156}

The same issues that affect SUSE Linux Enterprise Server 15.7 also apply to the SLES 15.6 image. Follow the same steps to resolve the gaps. Consider the following differences:

- The kernel patch version to be upgraded for the 15.6 patch stream. For example, by February 2026, the recommended version is `6.4.0-150600.23.81.3`.
- By February 2026, the `kernel-devel` package version to upgrade to version `6.4.0-150600.23.81.2`, and the same patch version as the kernel is not currently available. It might become consistent again in a later version.
- The two commands to run to install the `kernel-syms` package are:

```sh
sudo SUSEConnect -p sle-module-desktop-applications/15.6/x86_64
sudo SUSEConnect -p sle-module-development-tools/15.6/x86_64
```
{: pre}

#### ibm-sles-15-7-amd64-sap-hana-3 and ibm-sles-15-7-amd64-sap-applications-3
{: #virt-sol-vpc-observability-design-scc-vsi-sles157-sap}

The SUSE Linux Enterprise Server 15.7 for SAP Hana and SUSE Linux Enterprise Server 15.7 for SAP Applications share the exact same workaround with SLES 15.7 as described earlier.

#### ibm-sles-15-6-amd64-sap-hana-6 and ibm-sles-15-6-amd64-sap-applications-6
{: #virt-sol-vpc-observability-design-scc-vsi-sles156-sap}

The images for SUSE Linux Enterprise Server 15.6 for SAP Hana and SUSE Linux Enterprise Server 15.6 for SAP Applications use a similar workaround that is described for SLES 15.6. However, the following key differences are there:

- These two OS images contain extra local repositories for SAP packages. As a result, the latest available kernel patch version differs from the general SLES 15.6 image. For example, by February 2026, the upgrade patch version is `6.4.0-150600.23.87.1`.
- The extra local repositories also include the `kernel-syms` package, so no additional commands are required to install local repositories.

#### ibm-sles-15-5-amd64-sap-hana-11 and ibm-sles-15-5-amd64-sap-applications-11
{: #virt-sol-vpc-observability-design-scc-vsi-sles155-sap}

The images for SUSE Linux Enterprise Server 15.5 for SAP Hana and SUSE Linux Enterprise Server 15.5 for SAP Applications use a similar workaround as described for SLES 15.6, with minor differences:

- The images currently provide the latest version for the 15.5 patch stream (for example, by February 2026, it is `5.14.21-150500.55.136.1`).
- You do not need to upgrade the kernel version.
- You can install the `kernel-devel` package directly before you run the commands to install and deploy the SCC agent.

#### ibm-sles-15-4-amd64-sap-hana-16 and ibm-sles-15-4-amd64-sap-applications-16
{: #virt-sol-vpc-observability-design-scc-vsi-sles154-sap}

The images for SUSE Linux Enterprise Server 15.4 for SAP Hana and SUSE Linux Enterprise Server 15.4 for SAP Applications use a similar workaround as described for SLES 15.6, with minor differences:

- The images currently provide the latest version for the 15.4 patch stream (for example, by February 2026, it is `5.14.21-150400.24.194.1`).
- You do not need to upgrade the kernel version.
- You can install the `kernel-devel` package directly before you run the commands to install and deploy the SCC agent.

#### ibm-sles-15-3-amd64-sap-hana-14 and ibm-sles-15-3-amd64-sap-applications-15
{: #virt-sol-vpc-observability-design-scc-vsi-sles153-sap}

The SUSE Linux Enterprise Server 15.3 for SAP Hana and SUSE Linux Enterprise Server 15.3 for SAP Applications share a similar workaround as described for SLES 15.6 for SAP, with minor differences:

- Use the latest Kernel patch from the 15.3 version stream. For example, by February 2026, the version is `5.3.18-150300.59.229.3`.

```sh
The description for Event ID <ID> from source SysdigHostShield cannot be found. Either the component that raises this event is not installed on your local computer or the installation is corrupted. You can install or repair the component on the local computer.
If the event originated on another computer, the display information had to be saved with the event.
The following information was included with the event:
unable to load the configuration: sysdig_endpoint.api_url: '<SCC_API_HOST>' is not valid 'uri'
The message resource is present but the message was not found in the message table.
```
{: pre}

## {{site.data.keyword.monitoringfull_notm}} and Logs
{: #virt-sol-vpc-observability-design-mon-and-logs}

{{site.data.keyword.monitoringfull_notm}} and Logs provide cloud-native observability for applications and infrastructure running on the {{site.data.keyword.cloud_notm}}, including virtual machines on VPC and {{site.data.keyword.redhat_openshift_notm}} Virtualization.

| Service | Description | Agent deployment metrics collection |
| -------------- | -------------- | -------------- |
| {{site.data.keyword.monitoringfull_notm}} | {{site.data.keyword.monitoringfull_notm}} is a cloud-native, container-intelligence management system that provides operational visibility into the performance and health of applications, services, and platforms. It offers administrators, DevOps teams, and developers full-stack telemetry with advanced features for monitoring, troubleshooting, alerting, and custom dashboard creation. | To monitor infrastructure, networks, and applications, deploy Monitoring agents on supported hosts. The agent type depends on the host platform and determines which metrics are automatically collected. When a Monitoring agent is configured, default metrics are collected automatically, including metadata for labeling, segmentation, and filtering.
No additional instrumentation is required to gain insights from these automatically collected metrics.
For more information, see [Getting started with {{site.data.keyword.monitoringfull_notm}}](/docs/monitoring?topic=monitoring-getting-started), [Monitoring a Windows environment](/docs/monitoring?topic=monitoring-windows) and [Monitoring an Ubuntu Linux VPC server instance](/docs/monitoring?topic=monitoring-ubuntu). |
| {{site.data.keyword.logs_full_notm}} | {{site.data.keyword.logs_full_notm}} is an observability service that is designed to help organizations monitor, troubleshoot, analyze, and alert on application and infrastructure performance in real time and over extended periods. By collecting and analyzing logs from cloud-native applications, servers, databases, and IT systems, {{site.data.keyword.logs_full_notm}} provides actionable insights into system behavior. | {{site.data.keyword.logs_full_notm}} supports log collection from: \n - IBM Cloud services and resources \n - On-premises infrastructure \n - Third-party cloud providers \n - Security and audit logs generated in IBM Cloud \n \n To collect logs, deploy the {{site.data.keyword.logs_full_notm}} agent on supported hosts. The agent type depends on the host platform and determines which logs are automatically collected. When an {{site.data.keyword.logs_full_notm}} agent is configured, logs are collected automatically and sent to your {{site.data.keyword.logs_full_notm}} instance for analysis and alerting. \n \n For more information, see [Getting started with {{site.data.keyword.logs_full_notm}}](/docs/cloud-logs?topic=cloud-logs-getting-started) and [Configuring the {{site.data.keyword.logs_full_notm}} agent](/docs/cloud-logs?topic=cloud-logs-agent-about). |
{: caption="{{site.data.keyword.monitoringfull_notm}} and {{site.data.keyword.logs_full_notm}} details" caption-side="bottom"}

For IBM Cloud Linux virtual server instances and IBM Cloud Windows virtual server instances, the agent that is used with {{site.data.keyword.logs_full_notm}} supports both the Service ID API key and Trusted Profiles authentication methods.

### Combined observability benefits
{: #virt-sol-vpc-observability-design-combined-benefits}

{{site.data.keyword.cloud_notm}} uses a single unified agent that collects both security data (for workload protection) and metrics data (for {{site.data.keyword.monitoringfull_notm}}). Keep the following key points in mind:

- Do not deploy multiple instances of the agent on the same host. However, by connecting the instances, a single agent can collect both security and metrics data.
- You can connect only one monitoring instance to one workload protection instance, and both must be in the same region.

The following table lists the unified agent components.

| Component | Description |
| -------------- | -------------- |
| For monitoring (metrics) | - Agent: Collects metrics from containers, pods, nodes, and Kubernetes resources  \n - Prometheus integration: Custom metrics collection  \n  - Cluster metadata: Automatic tagging with cluster name and context |
| For workload protection (security) | - Node Analyzer: Includes host scanner and Kubernetes Security Posture Management (KSPM) analyzer  \n Host Scanner: Detects vulnerabilities and identifies resolution priority based on available fixed versions and severity  \n - KSPM Analyzer: Kubernetes Security Posture Management for compliance and configuration analysis  \n - Cluster Shield: Security runtime component |
{: caption="Components of unified agent" caption-side="bottom"}

The following table describes the observability capabilities available when you deploy both the unified agent (for {{site.data.keyword.monitoringfull_notm}} and workload protection) and the {{site.data.keyword.logs_full_notm}} agent on VPC virtual server instances.

| Observability | Description |
| -------------- | -------------- |
| Full-stack visibility | Monitor from infrastructure through application layers |
| Correlated insights | Correlate metrics and logs for faster root cause analysis |
| Unified dashboards | View metrics and logs in the integrated {{site.data.keyword.cloud_notm}} console |
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

Now that you understand the observability design for VPC virtual server instances, explore these related topics:

- Security: Review [security design considerations](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-security-design-overview) that includes compliance monitoring.
- Resiliency: Learn about [backup and disaster recovery](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vpc-resiliency-design) strategies.
- Networking: Explore [networking design patterns](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-design) for VPC connectivity.
- Reference architecture: Review the complete [VPC virtual server instance reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vsi-architecture).
