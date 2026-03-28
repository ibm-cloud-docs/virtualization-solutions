---

copyright:
  years: 2026, 2026
lastupdated: "2026-03-28"

keywords: virtualization, faq, migration, vpc, openshift, vmware, storage, networking, backup, disaster recovery

subcollection: virtualization-solutions

content-type: faq

---

{{site.data.keyword.attribute-definition-list}}

# FAQs for Virtualization Solutions
{: #virtualization-solutions-faq}

## General Questions
{: #general-questions}

### What are the two main virtualization options available on IBM Cloud?
{: #virtualization-options}
{: faq}

IBM Cloud offers two primary virtualization solutions:

- **IBM Cloud Virtual Servers for VPC** - Traditional VM-based virtualization with flexible compute profiles and simple deployment
- **Red Hat OpenShift Virtualization** - Container-native virtualization that runs VMs alongside containers on a unified Kubernetes platform

### Which virtualization solution should I choose for my workload?
{: #choose-solution}
{: faq}

Choose **VPC Virtual Servers** if you:
- Need traditional VM infrastructure with flexible compute profiles
- Want simple deployment without container orchestration
- Prefer a straightforward IaaS model

Choose **Red Hat OpenShift Virtualization** if you:
- Want to modernize gradually by running VMs alongside containers
- Need Kubernetes-native management
- Plan to migrate applications from VMs to containers over time

### What are the key differences between VPC Virtual Servers and OpenShift Virtualization?
{: #key-differences}
{: faq}

**VPC Virtual Servers:**
- Traditional IaaS model
- Simpler to deploy and manage
- Pay-per-use pricing
- Managed hypervisor

**OpenShift Virtualization:**
- Kubernetes-native platform
- Unified management for VMs and containers
- Requires OpenShift cluster
- More complex but enables modernization path

## Migration Questions
{: #migration-questions}

### What migration methods are available for moving from VMware to IBM Cloud?
{: #migration-methods}
{: faq}

**For VPC Virtual Servers:**
- Image import
- Direct volume copy
- Live network transfer
- VDDK extraction
- RackWare RMM (automated)

**For OpenShift Virtualization:**
- Migration Toolkit for Virtualization (MTV)

**Both platforms:**
- Service providers like WanClouds or PrimaryIO

### How long does a typical migration take?
{: #migration-duration}
{: faq}

Migration duration varies by method:
- **Manual methods**: 2-3 hours per VM
- **RackWare RMM**: 30-60 minutes per VM with automation
- **MTV**: Varies based on VM size and network bandwidth, supports warm migration to minimize downtime

### Can I migrate VMs with multiple disks?
{: #multi-disk-migration}
{: faq}

Yes, both platforms support multi-disk VMs:
- **VPC**: Use direct volume copy method or RackWare RMM
- **OpenShift**: MTV handles multi-disk VMs automatically

### Do I need to shut down my VMs during migration?
{: #migration-downtime}
{: faq}

It depends on the migration method:
- **Cold migration**: Yes, VMs must be powered off
- **Warm migration**: No, MTV and RackWare support warm migration with minimal downtime (minutes vs hours)

### Migration partners
{: #migration-partners}
{: faq}

IBM Cloud works with several migration partners. Customers can do migration with self-service and extensive documentation is available to support customers on their efforts.

IBM Consulting, Red Hat Consulting, and IBM business partners can help if customers require either resources or skills during their migration efforts.

## Storage Questions
{: #storage-questions}

### What storage options are available for VPC Virtual Servers?
{: #vpc-storage-options}
{: faq}

VPC Virtual Servers support:
- **Block Storage for VPC** - Boot and data volumes with customizable IOPS
- **File Storage for VPC** - NFS-based shared storage
- **Cloud Object Storage** - Backup and archival storage

### What storage is required for OpenShift Virtualization production workloads?
{: #openshift-storage-requirements}
{: faq}

For production workloads:
- **Red Hat OpenShift Data Foundation (ODF)** is required
- Bare metal servers with local NVMe drives are needed for ODF
- ODF provides block, file, and object storage from the same infrastructure

### Can I use existing VPC storage with OpenShift Virtualization?
{: #vpc-storage-with-openshift}
{: faq}

Yes, you can use:
- File Storage for VPC
- Block Storage for VPC (virtual server workers only)

However, ODF is recommended for production and required for disaster recovery features.

## Networking Questions
{: #networking-questions}

### How does networking work in OpenShift Virtualization?
{: #openshift-networking}
{: faq}

OpenShift Virtualization networking includes:
- VMs run in pods connected to the default pod network
- **OVN-Kubernetes** provides software-defined networking
- **Multus** enables multiple network interfaces per VM
- **Localnet** provides direct VPC subnet access via VLAN attachments

### Can migrated VMs keep their IP addresses?
{: #ip-address-retention}
{: faq}

IP address retention depends on the platform:
- **VPC**: No, VMs receive new IPs from VPC subnets
- **OpenShift with Localnet**: Yes, VMs can use existing VPC subnet IPs
- **RackWare**: Supports bridge servers for IP retention during migration

### How do I expose VMs to external traffic?
{: #external-traffic}
{: faq}

**VPC Virtual Servers:**
- Floating IPs
- Load balancers
- Public gateways

**OpenShift Virtualization:**
- Routes
- LoadBalancer services
- NodePort services

## Compute Questions
{: #compute-questions}

### What compute options are available for OpenShift Virtualization?
{: #openshift-compute-options}
{: faq}

OpenShift Virtualization supports:
- **Bare metal servers** - Required for production VMs per Red Hat support policy
- **Virtual servers** - Supported for containers and non-production VMs
- **Bare metal with local NVMe drives** - Required for ODF storage

### Can I run both VMs and containers on the same OpenShift cluster?
{: #vms-and-containers}
{: faq}

Yes, this is a key benefit of OpenShift Virtualization:
- VMs and containers share the same infrastructure and management platform
- Enables gradual modernization from VMs to containers
- Unified operations and security policies

## Backup & Disaster Recovery Questions
{: #backup-dr-questions}

### What backup options are available for VPC Virtual Servers?
{: #vpc-backup-options}
{: faq}

VPC Virtual Servers support multiple backup solutions:
- VPC Block Storage Snapshots (manual or automated)
- IBM Cloud Backup for VPC (policy-driven)
- IBM Cloud Backup and Recovery (agent-based)
- Veeam Backup & Replication
- Third-party solutions (Commvault, Rubrik, Veritas)

### What backup options are available for OpenShift Virtualization?
{: #openshift-backup-options}
{: faq}

OpenShift Virtualization supports:
- Veeam Kasten K10 (Kubernetes-native)
- OADP (OpenShift API for Data Protection)
- IBM Cloud Backup and Recovery
- Third-party solutions

### How do I implement disaster recovery for OpenShift Virtualization?
{: #openshift-disaster-recovery}
{: faq}

Disaster recovery options include:
- **RHACM + ODF Regional DR** for cross-region replication
- **OADP** for backup to object storage
- **Veeam Kasten** for application mobility
- Requires ODF on both primary and DR clusters

## Security & Compliance Questions
{: #security-compliance-questions}

### How is data encrypted in IBM Cloud virtualization solutions?
{: #data-encryption}
{: faq}

**VPC Virtual Servers:**
- Provider-managed or customer-managed encryption (Key Protect/HPCS)
- Boot and data volume encryption

**OpenShift Virtualization:**
- etcd and worker disk encryption
- Persistent volume encryption

**Both platforms:**
- Encryption in transit via TLS/VPN

### What compliance certifications does IBM Cloud support?
{: #compliance-certifications}
{: faq}

IBM Cloud supports multiple compliance frameworks:
- ISO 27001, 27017, 27018
- SOC 1, SOC 2, SOC 3
- PCI DSS, HIPAA, FedRAMP
- GDPR compliance support

## Cost & Licensing Questions
{: #cost-licensing-questions}

### What licensing is required for OpenShift Virtualization?
{: #openshift-licensing}
{: faq}

Licensing requirements:
- Red Hat OpenShift subscription includes OpenShift Virtualization
- Bare metal servers require appropriate sizing for workload
- Guest OS licenses (Windows, RHEL) are customer responsibility

### Can I bring my own licenses (BYOL)?
{: #byol}
{: faq}

Yes, BYOL is supported:
- **VPC**: Custom images support BYOL
- **OpenShift**: BYOL for guest operating systems
- **RackWare RMM**: BYOL model available in IBM Cloud catalog

## Performance & Sizing Questions
{: #performance-sizing-questions}

### What are the performance considerations for OpenShift Virtualization?
{: #openshift-performance}
{: faq}

Key performance factors:
- Bare metal servers provide better performance than virtual servers
- Local NVMe storage (ODF) provides high IOPS
- Multi-zone clusters not recommended due to storage latency
- Network performance depends on worker node profiles

### How do I size my environment for migration?
{: #environment-sizing}
{: faq}

Sizing considerations:
- Assess current VM inventory (CPU, memory, storage, IOPS)
- Consider network bandwidth for data transfer
- Plan for 20-30% overhead for Kubernetes/OpenShift
- Use IBM Cloud sizing tools or consult with IBM

## Support Questions
{: #support-questions}

### Where can I get help with migration?
{: #migration-help}
{: faq}

Multiple support options are available:
- IBM Consulting and Expert Labs
- Red Hat Consulting
- IBM Business Partners
- Migration service providers (WanClouds, PrimaryIO, RackWare)
- Self-service with extensive documentation
