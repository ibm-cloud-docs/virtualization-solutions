---

copyright:
  years: 2025
lastupdated: "2025-11-26"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Compute Design Components
{: #virt-sol-compute-design-overview}

IBM Cloud provides a comprehensive portfolio of compute options designed to support diverse workloads, from traditional applications to modern cloud-native solutions. These offerings deliver flexibility, scalability, and enterprise-grade security, enabling organizations to deploy workloads across virtualized, containerized, and bare metal environments.

Key IBM Cloud Compute Options:

1. Virtual Servers for VPC
2. Bare Metal Servers

IBM Cloud Compute solutions empower businesses to choose the right infrastructure for their needs—whether optimizing cost, achieving high performance, or enabling hybrid and multicloud strategies.

The key compute architecture elements are shown in the following diagram.

![Red Hat Virtualization on IBM Cloud Compute](images/openshift-virtualization-high-level-compute.svg "Red Hat Virtualization on IBM Cloud Compute"){: caption="Red Hat Virtualization on IBM Cloud Compute" caption-side="bottom"}


## Red Hat OpenShift Worker Nodes
{: #virt-sol-compute-design-rove-workers}

[OpenShift Virtualization]{: tag-red}

{{site.data.keyword.openshiftshort}} (ROKS) is a managed service in the IBM Cloud. 

It allows you to create your own cluster of compute hosts where you can deploy and manage containerized apps on IBM Cloud. Combined with an intuitive user experience, built-in security and isolation, and advanced tools to secure, manage, and monitor your cluster workloads, you can rapidly deliver highly available and secure containerized apps in the public cloud

A cluster consists of a master and one or more compute hosts that are called worker nodes. Worker nodes are organized into worker pools of the same flavor, or profile of CPU, memory, operating system, attached disks, and other properties. The worker nodes correspond to the Kubernetes Node resource, and are managed by a Kubernetes master that centrally controls and monitors all Kubernetes resources in the cluster. So when you deploy the resources for a containerized app, the Kubernetes master decides which worker node to deploy those resources on, accounting for the deployment requirements and available capacity in the cluster. Kubernetes resources include services, deployments, and pods.

When you create a cluster, the worker nodes are provisioned in a worker pool. After cluster creation, you can add more worker nodes to a pool by resizing it or by adding more worker pools. By default, the worker pool exists in one zone. Clusters that have a worker pool in only one zone are called single zone clusters. 

Bare Metal servers are required in the worker pool to install Red Hat OpenShift Virtualization. The nodes also should be provision with the local NVMe drives to create the OpenShift Data Foundation instance, providing the software defined storage cluster.

For more information, refer to [IBM Cloud Docs](https://cloud.ibm.com/docs/openshift?topic=openshift-overview).



## IBM Cloud Virtual Servers for VPC
{: #virt-sol-compute-design-virtual-servers}

[VPC VSI]{: tag-blue} [{{site.data.keyword.openshiftshort}}]{: tag-roks}

Secure, isolated virtual machines deployed within a Virtual Private Cloud (VPC). Ideal for traditional workloads, development environments, and scalable applications. Offers customizable profiles, dedicated or shared instances, and integration with advanced networking and storage services.

For more information on VSI profiles, see [IBM Cloud Docs](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-flavors).

## IBM Cloud Bare Metal Servers for VPC
{: #virt-sol-compute-design-bare-metal-servers}

[OpenShift Virtualization]{: tag-red}

Provide single-tenant, physical servers physical servers dedicated to your workloads, delivering maximum performance, security, and control.

For more information on Bare Metal profiles, see [IBM Cloud Docs](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-flavors).

OpenShift Virtualization supported the following profiles [Profile Availability](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-flavors#us-south-physical-table).
