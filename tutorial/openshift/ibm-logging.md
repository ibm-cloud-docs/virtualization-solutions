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

# Logging for {{site.data.keyword.redhat_openshift_notm}} Virtualization
{: #logging-rove-design}
{: #vsphere-openshift-logging}
{: #tutorial-observability-logging}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}
{: toc-use-case="ApplicationModernization"}
{: toc-industry="Software and platform applications"}
{: toc-compliance="HIPPA"}

The following guide describes how to configure logging for your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment. {{site.data.keyword.cloud}} Logs is a fully managed logging service that provides a centralized logging solution for your cloud resources. It provides a unified view of your cloud resources, including infrastructure, applications, and services.
{: shortdesc}

## Overview of {{site.data.keyword.logs_full_notm}} with {{site.data.keyword.redhat_openshift_notm}} Virtualization
{: #roks-virt-ibm-logs-overview}

You can use {{site.data.keyword.cloud_notm}} Logs to monitor your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment, which also includes:

- Virtual servers
- Networks
- Storage
- Applications that run on {{site.data.keyword.redhat_openshift_notm}} Virtualization, including the performance, availability, and security of your applications.

{{site.data.keyword.monitoringfull}} monitors your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment in the following ways:

- **Centralized logging**: A centralized logging solution for your cloud resources.
- **Log analysis**: A log analysis solution for your cloud resources.
- **Log management**: A log management solution for your cloud resources.

## Prerequisites
{: #roks-virt-logging-prerequisites}
{: step}

Before you configure logging for your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment, make sure that the following requirements are met:

1. Create an IBM Cloud Logs instance.
   - Provision an IBM Cloud Logs instance to start collecting and managing log data. For more information, see [Provision Instance](/docs/cloud-logs?topic=cloud-logs-instance-provision).
   - Configure a Cloud Logs bucket to enable long‑term log retention and advanced search capabilities. For more information, see [Configure Bucket](/docs/cloud-logs?topic=cloud-logs-configure-data-bucket). You need this configuration if you want long-term data retention or search. The standard Cloud Logs instance provides a minimum retention period of 7 days and a maximum retention period of 90 days for priority logs.
   - Create a Cloud Object Storage bucket that stores your long‑term log data. For more information, see [Creating a Cloud Object Storage bucket](/docs/cloud-object-storage?topic=cloud-object-storage-getting-started-cloud-object-storage).

2. Enable logging with a ROKS cluster
   - You enable integrated logging for your {{site.data.keyword.redhat_openshift_notm}} on IBM Cloud (ROKS) cluster by connecting it to the IBM Cloud Logs. For more information, see [Connect ICL with ROKS](https://cloud.ibm.com/docs/openshift?topic=openshift-logging).

## {{site.data.keyword.monitoringfull_notm}}
{: #roks-virt-ibm-monitoring}

{{site.data.keyword.monitoringfull_notm}} provides a comprehensive monitoring solution for your cloud resources. It offers a unified view of your cloud resources, including infrastructure, applications, and services. {{site.data.keyword.mon_full}} monitors your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment, including virtual machines, networks, and storage. Also, it monitors your applications that are running on {{site.data.keyword.redhat_openshift_notm}} Virtualization, including the performance, availability, and security of your applications.

This integration requires that you install a monitoring agent on your virtual servers. See [Monitoring an Ubuntu Linux VPC server instance](/docs/monitoring?topic=monitoring-ubuntu#ubuntu_step3) for an example installation of a monitoring agent on an Ubuntu virtual server.

For more information, see [Installing an agent-based virtual server](#agent-based-vsivm-installation-instructions).

{{site.data.keyword.monitoringfull_notm}} also supports monitoring overall VPC resource consumption and other VPC services. For more information, see [Getting started with IBM Cloud Monitoring](https://cloud.ibm.com/docs/monitoring?topic=monitoring-getting-started). 

### IBM Cloud Logs
{: #vpc-observability-logs}

{{site.data.keyword.vpc_short}} supports integration with IBM Cloud Logs. IBM Cloud VPC generates platform events and routes them to an IBM Cloud Logs instance by using {{site.data.keyword.logs_routing_full}}. For more information, see [Logging for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-logging).

IBM Cloud Activity Tracker Events Routing routes the events to an IBM Cloud Logs instance. For more information about the type of activity tracker events generated, see [Activity tracking events for IBM Cloud VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-at_events).

To set up log forwarding to IBM Cloud Logs, follow the steps for [Linux](https://cloud.ibm.com/docs/cloud-logs?topic=cloud-logs-agent-linux) and [Windows](https://cloud.ibm.com/docs/cloud-logs?topic=cloud-logs-agent-windows).

#### IBM Cloud Monitoring
{: #observability-design-mon}

{{site.data.keyword.monitoringfull_notm}} is a cloud-native and container-intelligence management system that you can include as part of your IBM Cloud architecture.

##### Installing IBM Cloud Monitoring 
{: #observability-design-install}
{: step}

To install and configure IBM Cloud Monitoring for your {{site.data.keyword.redhat_openshift_notm}} cluster, use the following steps:

1. Manually install Sysdig on your {{site.data.keyword.redhat_openshift_notm}} cluster. For more information, see the [Manual installation process for Sysdig on {{site.data.keyword.redhat_openshift_notm}}](https://docs.sysdig.com/en/administration/onprem-manual-installation-openshift/){: external}.
2. Add your {{site.data.keyword.redhat_openshift_notm}} cluster to {{site.data.keyword.monitoringfull_notm}}. For more information, see [Adding a {{site.data.keyword.redhat_openshift_notm}} cluster to IBM Cloud Monitoring](/docs/monitoring?topic=monitoring-openshift_cluster).

##### Installing an agent-based virtual server
{: #observability-design-install-agent}

To install IBM Logging Agents on a Linux VPC virtual server, go to the **Monitoring Sources** in your **IBM Cloud Monitoring instance** details in IBM Cloud Console. Then, click the **Linux** tab to deploy the monitoring agent on your Linux VPC virtual server. For more information, see
[Linux Agent Installation](/docs/monitoring?topic=monitoring-ubuntu).

To install IBM Logging Agents on a Windows VPC virtual server, see [Windows Agent Installation](/docs/monitoring?topic=monitoring-windows).

When you install the Windows agent, configure the Prometheus remote write ingestion endpoint and provide the required API token. For more information, see [Prometheus Remote Write ingestion endpoints](/docs/monitoring?topic=monitoring-endpoints#prometheus_remote_write_endpoints).

To get the IBM Cloud Log monitoring API token, use the following steps

1. Log in to your IBM Cloud Monitoring instance.
2. Open the **Dashboard**
3. Click the **User** icon that displays your initials at the end of the page
4. Under **Secrets management**, select **SysDig API tokens**.
5. Scroll down to locate the **Token** and click **Copy**

## Next steps
{: #observability-next-steps}

After you implement observability for your {{site.data.keyword.redhat_openshift_notm}} Virtualization environment, consider the following information.

- **Backup and recovery**: Implement backup solutions for your virtual servers. For more information, see [Backup solution for {{site.data.keyword.redhat_openshift_notm}} Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-backup).
- **Migration**: Learn about migrating workloads to {{site.data.keyword.redhat_openshift_notm}} Virtualization. For more information, see [Migration Toolkit for Virtualization](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-migration-design-mtv).
- **Design considerations**: Review the design guidance for production deployments.
   - Review guidance about sizing, resource allocation, and compute infrastructure planning for {{site.data.keyword.redhat_openshift_notm}} Virtualization workloads. For more information, see [Compute design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-compute-design)
   - Review guidance about designing network architectures, connectivity models, and traffic flows for {{site.data.keyword.redhat_openshift_notm}} Virtualization clusters. For more information, see [Networking design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-network-design)
   - Explore storage architecture recommendations for persistent volumes, performance needs, and data availability in virtualized workloads. For more information, see [Storage design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-storage-design-overview)
   - Learn best practices to secure {{site.data.keyword.redhat_openshift_notm}} Virtualization environments, which include access controls, encryption, and compliance considerations. For more information, see [Security design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-security-design-overview)
   - Understand strategies to build resilient {{site.data.keyword.redhat_openshift_notm}} Virtualization deployments that help ensure high availability and rapid workload recovery. For more information, see [Resiliency design](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-openshift-resiliency-design)
- **Reference Architecture**: Explore the complete reference architecture. For more information, see [{{site.data.keyword.redhat_openshift_notm}} Virtualization reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-rove-architecture).
