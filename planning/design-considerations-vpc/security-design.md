---

copyright:
  years: 2025, 2026
lastupdated: "2026-07-21"

keywords: VPC security design, IBM Cloud IAM, security groups VPC, network ACLs, data encryption VPC, Key Protect encryption VPC, Hyper Protect Crypto Services VPC, Security and Compliance Center, VPC Flow Logs, compliance monitoring

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Designing security for IBM Cloud VPC virtual servers
{: #virt-sol-vpc-security-design-overview}

Design security for IBM Cloud Virtual Private Cloud (VPC) virtual servers, covering Identity and Access Management (IAM), data encryption, network security, and compliance governance.
{: shortdesc}

The key security architecture elements are shown in the following diagram.

![IBM Cloud VPC VSI Security](../../images/vpc-vsi/vpc-vsi-high-level-security.svg "IBM Cloud VPC VSI Security"){: caption="IBM Cloud VPC VSI Security" caption-side="bottom"}

For workload migration and deployment, robust security capabilities are essential to maintain confidentiality, integrity, and availability while meeting regulatory and compliance requirements. IBM Cloud's security services integrate with native platform capabilities to provide end-to-end protection for virtualization and container workloads.

## Shared responsibility model
{: #virt-sol-vpc-security-design-shared-responsibility}

IBM Cloud uses a shared responsibility model that defines which security and compliance responsibilities are managed by IBM Cloud and which ones lie with customers. Understanding this model is critical for implementing effective security controls. See [Shared responsibilities for using IBM Cloud products](/docs/overview?topic=overview-shared-responsibilities) and [Infrastructure-as-a-service](/docs/overview?topic=overview-shared-responsibilities#iaas-services-responsibilities).

IBM Cloud provides a secure cloud platform that you can trust. IBM Cloud compliance results from a platform and services that are built on best-in-industry security standards, including General Data Protection Regulation (GDPR), Health Insurance Portability and Accountability Act (HIPAA), ISO 9001, ISO 27001, ISO 27017, ISO 27018, Payment Card Industry (PCI), SOC 2, and others. See [Understanding compliance in IBM Cloud](/docs/overview?topic=overview-compliance).

## Identity and access management
{: #virt-sol-vpc-security-design-iam}

IBM Cloud Identity and Access Management (IAM) provides centralized access control for IBM Cloud resources, enabling organizations to manage users, service IDs, access groups, and policies across the entire IBM Cloud platform.

### IAM components
{: #virt-sol-vpc-security-design-iam-components}

| IAM features | Description |
| -------------- | -------------- |
| Users and services IDs | - IBMid authentication for human users \n - Service IDs for applications and automation \n - API keys for programmatic access \n - Multifactor authentication (MFA) support |
| Access groups | - Logical grouping of users and service IDs  \n - Centralized policy management \n - Dynamic membership based on identity attributes \n - Simplified access governance at scale |
| IAM policies | - Resource-level access control \n - Platform roles for infrastructure management \n - Service roles for workload operations \n - Attribute-based access control (ABAC) |
{: caption="Identity and Access Management features" caption-side="bottom"}

## Data encryption
{: #virt-sol-vpc-security-design-encryption}

IBM Cloud provides comprehensive encryption capabilities to protect data at rest and in transit across VPC environments. The following table details each encryption service and the encryption capabilities available with that service.

| Service | Description |
| -------------- | -------------- |
| VPC block storage encryption | - Provider-managed encryption by default (IBM-managed keys). \n - Customer-managed encryption by using IBM Cloud Key Protect or Hyper Protect Crypto Services \n - AES-256 encryption standard \n - Encryption of virtual server instance boot volumes and data volumes |
| IBM Cloud Key Protect | - Bring-your-own-key (BYOK) model with keys protected by Federal Information Processing Standard (FIPS) 140-2 Level 2 cloud hardware security module (HSM). \n - Centralized key lifecycle management. \n - Key rotation and versioning. \n - Audit logging for key operations. \n - Integration with VPC and Red Hat OpenShift services |
| IBM Hyper Protect Crypto Services | - Keep-your-own-key (KYOK) model utilizing FIPS 140-2 Level 4 cloud HSM (the only cloud vendor to offer this level) \n - Customer-controlled HSM \n - Exclusive customer control over encryption keys \n - Enhanced compliance for regulated industries |
{: caption="Encryption-at-rest encryption capabilities" caption-side="bottom"}
{: summary="This table provides all the encryption-at-rest encryption capabilities."}
{: #vpc-encryption-at-rest}
{: tab-title="Encryption-at-rest"}
{: tab-group="Encryption"}

| Service | Description |
| -------------- | -------------- |
| VPC Network Encryption | - End-to-end encryption is possible when using secure endpoints, such as HTTPS servers on port 443, with floating IPs attached to instances. \n - VPN gateway encryption using Internet Protocol Security (IPsec). \n - Transport Layer Security (TLS)/Secure Sockets Layer (SSL) for application layer security  \n - Direct Link with Media Access Control Security (MACsec) encryption for private connectivity |
{: caption="Encryption-in-transit encryption capabilities" caption-side="bottom"}
{: summary="This table provides all the encryption-in-transit encryption capabilities."}
{: #openshift-encryption-in-transit}
{: tab-title="Encryption-in-transit"}
{: tab-group="Encryption"}

## Network security
{: #virt-sol-vpc-security-design-network}

IBM Cloud VPC provides multiple layers of network security controls to protect workloads and control traffic flow.

| VPC security control | Description | Key features |
| -------------- | -------------- | -------------- |
| VPC Security Groups | Security Groups are stateful firewall controls that protect virtual instances on IBM Cloud VPC, with stateful rules where responses are automatically allowed when a request is permitted. | - Instance-level (network interface) security  \n - Stateful traffic filtering  \n - Attached to virtual server instance network interface controllers (NICs) or load balancers  \n - Ingress (inbound) and egress (outbound) rules  \n - Support for protocol, port, and source/destination specification |
| VPC Access Control Lists (ACLs) | ACLs control traffic to and from subnets, acting as built-in virtual firewalls at the subnet level. | - Subnet-level security  \n - Stateless traffic filtering - if you want to permit traffic both ways on a target you must set up two rules  \n - All resources in a subnet with an associated ACL follow ACL rules  \n - Rules evaluated in numerical order (priority-based)  \n - Allow and deny rules for granular control  \n - Use ACLs for broad subnet-level controls  \n - Combine ACLs with security groups for defense-in-depth  \n - Implement explicit deny rules for known malicious traffic  \n - Order rules efficiently (most specific first)  \n - Document ACL rule purposes and maintenance procedures |
{: caption="VPC network security controls" caption-side="bottom"}

## Compliance and governance
{: #virt-sol-vpc-security-design-compliance}

IBM Cloud provides comprehensive compliance capabilities and certifications to meet regulatory requirements across industries.

### IBM Cloud Security and Compliance Center Workload Protection
{: #virt-sol-vpc-security-design-scc}

| Feature | Description |
| -------------- | -------------- |
| Posture management | - Continuous security posture assessment. \n - Configuration compliance scanning. \n - Drift detection from security baselines. \n - Remediation guidance and automation |
| Compliance monitoring | - Regulatory compliance validation. \n - Custom control framework definition. \n - Evidence collection for audits. \n - Compliance dashboards and reporting |
| Workload protection | - Runtime threat detection. \n - Vulnerability scanning for VMs and containers. \n - File integrity monitoring. \n - Compliance scanning for CIS benchmarks and other frameworks |
{: caption="IBM Cloud Security and Compliance Center Workload Protection" caption-side="bottom"}

### Activity tracking and logging
{: #virt-sol-vpc-security-design-logging}

| Feature | Description |
| -------------- | -------------- |
| IBM Cloud Activity Tracker | - Audit logging for all {{site.data.keyword.cloud_notm}} application programming interface (API) calls \n - User activity tracking and attribution \n - Resource lifecycle event logging |
| VPC Flow Logs | - Network traffic capture and analysis  \n - Troubleshooting connectivity issues \n - Security incident investigation \n - Compliance evidence collection |
{: caption="Activity tracking and logging" caption-side="bottom"}

## Next steps
{: #virt-sol-vpc-security-design-next-steps}

Now that you understand the security design for VPC virtual servers, explore these related topics:

- **Networking**: Review [networking security controls](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-design) including security groups and ACLs
- **Compliance**: Learn about [observability and compliance monitoring](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-observability-design-overview)
- **Storage**: Explore [storage encryption options](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-storage-design-overview) for data protection
- **Reference architecture**: Review the complete [VPC virtual server reference architecture](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-vsi-architecture)
