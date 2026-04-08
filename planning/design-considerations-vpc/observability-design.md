---

copyright:
  years: 2025, 2026
lastupdated: "2026-04-08"

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

Generally, for IBM VPC VSI with IBM provided Linux OS image, follow the guide from [Protecting Linux hosts](/docs/workload-protection?topic=workload-protection-protecting-linux-hosts) would help to install and deploy a Workload Protection agent on the VSI and connect to the Workload Protection instance. For VPC VSI with IBM provided Windows OS image, follow the guide from [Managing the Workload Protection agent on Windows Servers](/docs/workload-protection?topic=workload-protection-agent-deploy-windows) will help achieve the goal also. But there are gaps and limitations with several OS images currently provided by IBM VPC VSI, and below best practices or workaround shall be taken to facilitate the agent deployment (for images not specially mentioned below, general guide itself works perfectly).

> NOTE:
> For customer-provide custom image uploaded to IBM VPC VSI platform, users should self check with default guide steps first and handle any possible gaps individually. If necessary, may raise support ticket towards the instance of IBM Cloud Security and Compliance Center Workload Protection.

#### ibm-centos-stream-9-amd64-13
{: #virt-sol-vpc-observability-design-scc-vsi-centos9}

The default kernel version of the CentOS OS with this image does not have a corresponding kernel-devel package available, thus install with `sudo yum -y install kernel-devel-$(uname -r)` advised by the guide will fail. An efficient workaround is:

- Run command `sudo dnf list --showduplicates kernel-devel` to list and get the latest available kernel patch version for current CentOS version, and run command `sudo yum install kernel-<LATEST_KERNEL_PATCH_VERSION>` to upgrade the kernel version(e.g., by February 2026, it should be 5.14.0-665.el9.x86_64, and the command to execute is `sudo yum install kernel-5.14.0-665.el9`), and then reboot the VSI.
- Run `uname -r` to make sure the kernel version has been successfully upgraded. Follow the general guide to install kernel-devel package and then download and install agent to the VSI, which should pass without issues.

#### ibm-centos-stream-10-amd64-5
{: #virt-sol-vpc-observability-design-scc-vsi-centos10}

The same issue with above CentOS 9 applies to CentOS 10 image. Besides taking above workaround suggestions, there is one extra issue with CentOS 10 OS agent installation: the required dkms package has an incompatible GPG key mandated by CentOS 10, and there are no newer version of dkms package delivered with compatible GPG key by now. So before following the general guide to download and install the agent, run `sudo dnf install --nogpgcheck dkms` to install dkms without GPG check first.

#### ibm-debian-13-2-minimal-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-debian13}

General guide will work mostly, just for debian 13 OS image, need to run command `sudo apt-get update` to update the installed packages to latest to avoid failures to fetch some archives issues like below:

```
E: Failed to fetch http://mirrors.adn.networklayer.com/debian-security/pool/updates/main/c/curl/libcurl4_7.74.0-1.3%2bdeb11u15_amd64.deb  404  Not Found [IP: 161.26.0.6 80]
E: Failed to fetch http://mirrors.adn.networklayer.com/debian-security/pool/updates/main/c/curl/curl_7.74.0-1.3%2bdeb11u15_amd64.deb  404  Not Found [IP: 161.26.0.6 80]
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

#### ibm-debian-12-12-minimal-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-debian12}

General guide will work mostly, just for debian 12 OS image, need to run command `sudo apt install curl` to install curl package which is required for downloading of agent installer.

#### ibm-debian-11-11-minimal-amd64-5
{: #virt-sol-vpc-observability-design-scc-vsi-debian11}

Both the issues with debian 13 and 12 apply to debian 11 OS image. So before following the general guide, run command `sudo apt-get update` and `sudo apt install curl` to bridge the gaps.

#### ibm-fedora-coreos-43-testing-1 and ibm-fedora-coreos-43-stable-1
{: #virt-sol-vpc-observability-design-scc-vsi-coreos43}

The IBM VPC VSI offered Fedora CoreOS images are with a read-only Fedora Bootc (Immutable) system, and according to the FAQ documentation https://docs.fedoraproject.org/en-US/fedora-coreos/faq/#_how_do_i_run_custom_applications_on_fedora_coreos, Fedora CoreOS is primarily targeted for containers and discourage additional installation on the base OS. So SCC integration on VSIs rendered by these images are not technically supported.

#### ibm-redhat-ai-nvidia-1-5-2-amd64-1 and ibm-redhat-ai-intel-1-5-1-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-redhatai}

Both the 2 Redhat AI OS image are not registered with an entitled repository server because the images released from Redhat currently have no internal infrastructure. They are offered as CoreOS and no custom application installation is not supported like Fedora CoreOS above.

It is unsure if this is purposely the repository is not attached to Redhat AI to avoid kernel modification or not, but with current supported way of agent installation, this is required. So for these 2 OS images, Cloud Security and Compliance Center agent injection is not supported until such a repository is registered.

#### ibm-rocky-linux-9-6-minimal-amd64-3
{: #virt-sol-vpc-observability-design-scc-vsi-rocky9}

The similar issues with CentOS 9 apply to Rocky Linux 9. The same workaround may be adopted by running `sudo dnf list --showduplicates kernel-devel` to obtain the latest patch version then running `sudo dnf install kernel-<LATEST_KERNEL_PATCH_VERSION>` to upgrade the kernel (e.g., by February 2026, the latest patch version is `5.14.0-611.27.1.el9_7`, and the command to issue is `sudo dnf install kernel-5.14.0-611.27.1.el9_7`). After kernel upgrade, a reboot is required.

#### ibm-rocky-linux-10-minimal-amd64-3
{: #virt-sol-vpc-observability-design-scc-vsi-rocky10}

The similar issues with CentOS 10 apply to Rocky Linux 10. Besides upgrade the kernel patch version to latest then kick off a reboot (e.g., by February 2026, the latest patch version is `6.12.0-124.31.1.el10_1`, and the command to issue is `sudo dnf install kernel-6.12.0-124.31.1.el10_1`), need to install dkms package by running command `sudo dnf install --nogpgcheck dkms` before downloading and deploying the agent.

#### ibm-sles-16-amd64-1
{: #virt-sol-vpc-observability-design-scc-vsi-sles16}

The similar issues with CentOS 9 apply to SUSE Linux Enterprise Server 16 image. Follow below steps to achieve a workaround:

- Run command `sudo zypper search -s kernel-*-devel` to list and obtain the latest kernel patch version (e.g., by February 2026, the patch version is `6.12.0-160000.9.1`)
- Run command `sudo zypper install kernel-<LATEST_KERNEL_PATCH_VERSION>` to upgrade the kernel (e.g., by February 2026, the command is `sudo zypper install kernel-default-6.12.0-160000.9.1`)
- Reboot the VSI. Run `uname -r` to make sure the kernel has been successfully upgraded. Run command `sudo zypper install kernel-devel` to install the kernel-devel package as a replacement for the first step of general guide. Then follow the left steps.

#### ibm-sles-15-7-amd64-3
{: #virt-sol-vpc-observability-design-scc-vsi-sles157}

The similar issues with SUSE Linux Enterprise Server 16 above apply to the SUSE Linux Enterprise Server 15.7 image. Besides what are stated above should be taken (by February 2026, the kernel patch version to be upgraded to is `6.4.0-150700.53.31.1`), there one extra problem to resolve - the current SLES 15 image does not contain enough local repositories for a required kernel package kernel-syms, which will lead agent installation failure. Below step should be performed before the agent downloading and installation step of the general guide:

- Run below 2 commands to configure 2 local repositories containing package kernel-syms:

```
sudo SUSEConnect -p sle-module-desktop-applications/15.7/x86_64
sudo SUSEConnect -p sle-module-development-tools/15.7/x86_64
```

#### ibm-sles-15-6-amd64-6
{: #virt-sol-vpc-observability-design-scc-vsi-sles156}

The same set of issues with SUSE Linux Enterprise Server 15.7 apply to SLES 15.6 image. The same steps should be followed to resolve the gaps. Note below differences:

- The kernel patch version to be upgraded to is for 15.6 patch stream, e,g, by February 2026, it should be `6.4.0-150600.23.81.3`.
- By February 2026 kernel-devel package version to be upgraded to is `6.4.0-150600.23.81.2`, and the same patch version with kernel does not exist for now. It may become consistent again with the version up.
- The 2 commands to execute for installation of kenel-syms package are:

```
sudo SUSEConnect -p sle-module-desktop-applications/15.6/x86_64
sudo SUSEConnect -p sle-module-development-tools/15.6/x86_64
```

#### ibm-sles-15-7-amd64-sap-hana-3 and ibm-sles-15-7-amd64-sap-applications-3
{: #virt-sol-vpc-observability-design-scc-vsi-sles157-sap}

The SUSE Linux Enterprise Server 15.7 for SAP Hana and SUSE Linux Enterprise Server 15.7 for SAP Applications share the exact same workaround with SLES 15.7 above.

#### ibm-sles-15-6-amd64-sap-hana-6 and ibm-sles-15-6-amd64-sap-applications-6
{: #virt-sol-vpc-observability-design-scc-vsi-sles156-sap}

The SUSE Linux Enterprise Server 15.6 for SAP Hana and SUSE Linux Enterprise Server 15.6 for SAP Applications share similar workaround with SLES 15.6 above, but with below key differences:

- These 2 SO images contain extra local repositories for SAP pack, thus the latest available kernel patch version is somehow different from above general SLES 15.6 image. e.g., by February 2026, the patch version to be upgraded to is `6.4.0-150600.23.87.1`.
- The extra local repositories covered kenel-syms package installation, so no extra commands are needed to install more local repositories.

#### ibm-sles-15-5-amd64-sap-hana-11 and ibm-sles-15-5-amd64-sap-applications-11
{: #virt-sol-vpc-observability-design-scc-vsi-sles155-sap}

The images for SUSE Linux Enterprise Server 15.5 for SAP Hana and SUSE Linux Enterprise Server 15.5 for SAP Applications share similar workaround with SLES 15.6 above with miner differences:

- The images currently have provided the latest version for 15.5 patch stream((e,g, by February 2026, it is `5.14.21-150500.55.136.1`), and there is no need to upgrade the kernel version. kernel-devel package could be installed directly before running commands to install and deploy the SCC agent.

#### ibm-sles-15-4-amd64-sap-hana-16 and ibm-sles-15-4-amd64-sap-applications-16
{: #virt-sol-vpc-observability-design-scc-vsi-sles154-sap}

The images for SUSE Linux Enterprise Server 15.4 for SAP Hana and SUSE Linux Enterprise Server 15.4 for SAP Applications share similar workaround with SLES 15.6 above with miner differences:

- The images currently have provided the latest version for 15.4 patch stream(e,g, by February 2026, it is `5.14.21-150400.24.194.1`), and there is no need to upgrade the kernel version. kernel-devel package could be installed directly before running commands to install and deploy the SCC agent.

#### ibm-sles-15-3-amd64-sap-hana-14 and ibm-sles-15-3-amd64-sap-applications-15
{: #virt-sol-vpc-observability-design-scc-vsi-sles153-sap}

The SUSE Linux Enterprise Server 15.3 for SAP Hana and SUSE Linux Enterprise Server 15.3 for SAP Applications share the similar workaround with SLES 15.6 for SAP with below miner differences:

- The kernel patch version should be the latest from 15.3 version stream, r.g., by February 2026, it is `5.3.18-150300.59.229.3`.

```
The description for Event ID <ID> from source SysdigHostShield cannot be found. Either the component that raises this event is not installed on your local computer or the installation is corrupted. You can install or repair the component on the local computer.

If the event originated on another computer, the display information had to be saved with the event.

The following information was included with the event: 

unable to load the configuration: sysdig_endpoint.api_url: '<SCC_API_HOST>' is not valid 'uri'

The message resource is present but the message was not found in the message table

```

## IBM Cloud Monitoring and Logs
{: #virt-sol-vpc-observability-design-mon-and-logs}

IBM Cloud Monitoring and IBM Cloud Logs provide cloud-native observability for applications and infrastructure running on IBM Cloud, including virtual machines on VPC and OpenShift Virtualization.

| Service | Description | Agent deployment metrics collection |
| -------------- | -------------- | -------------- |
| IBM Cloud Monitoring | IBM Cloud Monitoring is a cloud-native, container-intelligence management system that provides operational visibility into the performance and health of applications, services, and platforms. It offers administrators, DevOps teams, and developers full-stack telemetry with advanced features for monitoring, troubleshooting, alerting, and custom dashboard creation. | To monitor infrastructure, networks, and applications, deploy Monitoring agents on supported hosts. The agent type depends on the host platform and determines which metrics are automatically collected. When a Monitoring agent is configured, default metrics are collected automatically, including metadata for labeling, segmentation, and filtering.  
No additional instrumentation is required to gain insights from automatically collected metrics.  
For more information, see [Getting started with IBM Cloud Monitoring](/docs/monitoring?topic=monitoring-getting-started), [Monitoring a Windows environment](/docs/monitoring?topic=monitoring-windows) and [Monitoring an Ubuntu Linux VPC server instance](/docs/monitoring?topic=monitoring-ubuntu). |
| IBM Cloud Logs | IBM Cloud Logs is an observability service designed to help organizations monitor, troubleshoot, analyze, and alert on application and infrastructure performance in real time and over extended periods. By collecting and analyzing logs from cloud-native applications, servers, databases, and IT systems, IBM Cloud Logs provides actionable insights into system behavior. | IBM Cloud Logs supports log collection from:  
- IBM Cloud services and resources  
- On-premises infrastructure  
- Third-party cloud providers  
- Security and audit logs generated in IBM Cloud  

To monitor infrastructure, networks, and applications, deploy Monitoring agents on supported hosts. The agent type depends on the host platform and determines which metrics are automatically collected. When a Monitoring agent is configured, default metrics are collected automatically, including metadata for labeling, segmentation, and filtering. No additional instrumentation is required to gain insights from automatically collected metrics.  

For more information, see [Getting started with IBM Cloud Monitoring](/docs/monitoring?topic=monitoring-getting-started), [Monitoring a Windows environment](/docs/monitoring?topic=monitoring-windows) and [Monitoring an Ubuntu Linux VPC server instance](/docs/monitoring?topic=monitoring-ubuntu). |
{: caption="IBM Cloud Monitoring and IBM Cloud Logs details" caption-side="bottom"}

Be aware that for IBM Cloud Linux VSI and IBM Cloud Windows VSI both Service ID API key and Trusted Profiles authentication methods are supported by the agent with the IBM Cloud Logs service.

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
