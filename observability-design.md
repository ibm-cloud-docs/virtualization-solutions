---

copyright:
  years: 2025
lastupdated: "2025-11-26"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Observability Design Components
{: #virt-sol-observability-design-overview}

Observability in IBM Cloud provides the visibility and insights needed to monitor, troubleshoot, and optimize applications and infrastructure across hybrid and multicloud environments. It goes beyond traditional monitoring by offering end-to-end visibility into metrics, logs, and traces, enabling proactive detection of issues and faster root-cause analysis.
IBM Cloud Observability solutions include IBM Cloud Monitoring with Sysdig, IBM Cloud Logging, which together deliver a comprehensive view of system health and performance. 

These services help organizations ensure application reliability, security compliance, and operational efficiency by providing real-time dashboards, alerting, and deep analytics.

The key compute architecture elements are shown in the following diagram.

![Red Hat Virtualization on IBM Cloud Observability](images/openshift-virtualization-high-level-observability.svg "Red Hat Virtualization on IBM Cloud Observability"){: caption="Red Hat Virtualization on IBM Cloud Observability" caption-side="bottom"}


## RedHat Advanced Cluster Management (RHACM)
{: #virt-sol-observability-design-rhacm}

[OpenShift Virtualization]{: tag-red}

Red Hat Advanced Cluster Management (RHACM) is a centralized tool that simplifies the management of virtual machines (VMs) deployed to Red Hat OpenShift clusters as well as the clusters themselves.Advanced Cluster Management addresses these requirements by providing a centralized platform for provisioning, monitoring, and decommissioning VMs across multiple OpenShift clusters. It is also a critical element when building DR solutions with OpenShift clusters, but it is optional - mostly useful if you have a hybrid and multicloud OpenShift environment.

Red Hat Advanced Cluster Management provides end-to-end management visibility and control to manage your OpenShift environment. Take control of your application modernization program with management capabilities for cluster creation, application lifecycle, and provide security and compliance for all of them across hybrid cloud environments. Clusters and applications are all visible and managed from a single console, with built-in security policies. Run your operations from anywhere that Red Hat OpenShift runs.

The **hub cluster** is the common term that is used to define the central controller that runs in a Red Hat Advanced Cluster Management for Kubernetes cluster. From the hub cluster, you can access the console and product components, as well as the Red Hat Advanced Cluster Management APIs. You can also use the console to search resources across clusters and view your topology.

The **managed cluster** is the term that is used to define additional clusters that are managed by the hub cluster. The connection between the two is completed by using the klusterlet, which is the agent that is installed on the managed cluster. The managed cluster receives and applies requests from the hub cluster and enables it to service cluster lifecycle, application lifecycle, governance, and observability on the managed cluster.

Red Hat Advanced Cluster Management **cluster lifecycle** defines the process of creating, importing, managing, and destroying Kubernetes clusters across various infrastructure cloud providers, private clouds, and on-premises data centers. The cluster lifecycle function is provided by the multicluster engine for Kubernetes operator, which is installed automatically with Red Hat Advanced Cluster Management.

Red Hat Advanced Cluster Management **application lifecycle** defines the processes that are used to manage application resources on your managed clusters. 

**Governance** enables you to define policies that either enforce security compliance, or inform you of changes that violate the configured compliance requirements for your environment. Using dynamic policy templates, you can manage the policies and compliance requirements across all of your management clusters from a central interface.

### Observability
{: #virt-sol-observability-design-rhacm-observe}

[OpenShift Virtualization]{: tag-red}

The **Observability** component collects and reports the status and health of the OpenShift Container Platform, managed clusters to the hub cluster, which are visible from the Grafana dashboard. You can create custom alerts to inform you of problems with your managed clusters. 


## Red Hat OpenShift Observability
{: #virt-sol-observability-design-red-hat-observability}

[OpenShift Virtualization]{: tag-red}

Red Hat OpenShift Observability provides real-time visibility, monitoring, and analysis of various system metrics, logs, traces, and events to help users quickly diagnose and troubleshoot issues before they impact systems or applications. To help ensure the reliability, performance, and security of your applications and infrastructure, OpenShift Container Platform offers the following Observability components:

* Monitoring
* Logging
* Distributed tracing
* Red Hat build of OpenTelemetry
* Network Observability
* Power monitoring

Red Hat OpenShift Observability connects open-source observability tools and technologies to create a unified Observability solution. The components of Red Hat OpenShift Observability work together to help you collect, store, deliver, analyze, and visualize data.

**Monitoring** stack components are deployed by default in every OpenShift Container Platform installation and are managed by the Cluster Monitoring Operator (CMO). These components include Prometheus, Alertmanager, Thanos Querier, and others. The CMO also deploys the Telemeter Client, which sends a subset of data from platform Prometheus instances to Red Hat to facilitate Remote Health Monitoring for clusters.

With **logging** you can collect, visualize, forward, and store log data to troubleshoot issues, identify performance bottlenecks, and detect security threats. Users can configure the LokiStack deployment to produce customized alerts and recorded metrics.

With **distributed tracing**, you can store and visualize large volumes of requests passing through distributed systems, across the whole stack of microservices, and under heavy loads. Use it for monitoring distributed transactions, gathering insights into your instrumented services, network profiling, performance and latency optimization, root cause analysis, and troubleshooting the interaction between components in modern cloud-native microservices-based applications.


With Red Hat build of **OpenTelemetry** you can instrument, generate, collect, and export telemetry traces, metrics, and logs to analyze and understand your software’s performance and behavior. Use open-source back ends like Tempo or Prometheus, or use commercial offerings. Learn a single set of APIs and conventions, and own the data that you generate.

**Network Observability** helps you to observe the network traffic for OpenShift Container Platform clusters and create network flows with the Network Observability Operator. View and analyze the stored network flows information in the OpenShift Container Platform console for further insight and troubleshooting.


**Power monitoring** allows you to monitor the power usage of workloads and identify the most power-consuming namespaces running in a cluster with key power consumption metrics, such as CPU or DRAM measured at the container level. Visualize energy-related system statistics with the Power Monitoring Operator.

## IBM Cloud Security and Compliance Center Workload Protection
{: #virt-sol-observability-design-scc-wpp}


[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

IBM Cloud Security and Compliance Center Workload Protection agent can be used to find and prioritize software vulnerabilities, detect and respond to threats, and manage configurations, permissions, and compliance of the hosted VMs.

To add monitoring features with IBM Cloud Security and Compliance Center Workload Protection in the IBM Cloud, you must provision an instance of the IBM Cloud Security and Compliance Center Workload Protection service. After you provision an instance of the IBM Cloud Security and Compliance Center Workload Protection service in the IBM Cloud, you can deploy the agent on your cluster. The agent collects data that you can use for intrusion detection, posture management, vulnerability scanning, and incident response capabilities.

In OpenShift Virtualization, your can deploy the Workload Protection Agent in:

* OpenShift Cluster
* Virtual machine operating systems running on OpenShift Virtualization

With this solution, customers can accelerate their hybrid cloud adoption by addressing security and regulatory compliance. Easily identify vulnerabilities, validate compliance and permissions, block runtime threats and respond to incidents faster across any platform: Cloud or on-prem, hosts or VMs and containers or OpenShift/Kubernetes.

## IBM Cloud Monitoring and Logs
{: #virt-sol-observability-design-mon-and-logs}

[VPC VSI]{: tag-blue} [OpenShift Virtualization]{: tag-red}

**IBM Cloud Monitoring and Log** agents can be used to enhance logging and monitoring hosted VM’s operating systems.

**IBM Cloud Monitoring** is a cloud-native, and container-intelligence management system that you can include as part of your IBM Cloud architecture. Use it to gain operational visibility into the performance and health of your applications, services, and platforms. It offers administrators, DevOps teams and developers full-stack telemetry with advanced features to monitor and troubleshoot, define alerts, and design custom dashboards.

**IBM Cloud Logs** is an observability service in IBM Cloud designed to help organizations monitor, troubleshoot, analyze, and alert on the performance of their applications and infrastructure in real time and long term. By collecting and analyzing logs from cloud-native applications, servers, databases, and other IT systems, IBM Cloud Logs provides actionable insights into system behavior, helping SRE and Dev teams quickly identify and resolve issues. With IBM Cloud Logs, you can monitor operational data that is generated in IBM Cloud, on-premises, and by other cloud providers. You can also monitor security data that is generated in IBM Cloud.

To monitor your infrastructure, network, and applications with the IBM Cloud Monitoring service, you can deploy Monitoring agents on supported hosts. The host determines the agent type that you can deploy. The agent type determines the metrics that are collected automatically for that host. When you configure a Monitoring agent, data for default metrics is automatically collected. These metrics include metadata that you can use to label, segment, and display metrics when you monitor them. You do not need additional instrumentation or configuration in your hosts to obtain metrics that are collected automatically by the agent to gain insight into what is happening in them.

You can configure the Logging agent to collect and send infrastructure and application logs to an IBM Cloud Logs instance directly. The Logging agent is based on the Fluent Bit open-source agent which is used to collect and process log data. You can deploy the Logging agent in supported environments and manage data from various sources and formats.

In the context of OpenShift Virtualization, you can deploy the Monitoring and Logging agent in your VMs to get better insights what is happening inside the operating systems. These will further enhance the observability of the OpenShift Virtualization cluster and its workloads.
