---

copyright:
  years: 2025, 2026
lastupdated: "2026-10-06"

keywords: VMware prerequisites OpenShift Virtualization IBM Cloud, VMware vSphere 6.5 migration requirements, ESXi host migration prerequisites IBM Cloud, VMware Tools requirements OpenShift, VM naming conventions OpenShift migration, vCenter migration checklist IBM Cloud, NFC service memory ESXi migration, VDDK requirements OpenShift migration, VMware migration prerequisites ROKS, prepare VMware for OpenShift migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# VMware prerequisites for {{site.data.keyword.redhat_openshift_notm}} Virtualization
{: #virt-sol-openshift-migration-design-migration-vmware}

Check VMware vSphere, ESXi host settings, VDDK, and VM naming requirements before migrating VMs to {{site.data.keyword.redhat_openshift_notm}} Virtualization on {{site.data.keyword.cloud_notm}}.
{: shortdesc}

VMware environment requirements:

- VMware vSphere must be version 6.5 or later.
- If you are migrating more than 10 virtual servers from an ESXi host in the same migration plan, you must increase the Network File Copy (NFC) service memory of the host.

Virtual server requirements:

- VMware Tools is installed.
- ISO and CD-ROM disks are unmounted.
- Each network interface card (NIC) must contain no more than one IPv4 and or one IPv6 address.
- Virtual server names must contain only the following characters:
   - lowercase letters (a-z), numbers (0-9), or hyphens (-), up to a maximum of 253 characters.
   - The first and last characters must be alphanumeric.
- The name must not contain uppercase letters, spaces, periods (.), or special characters.
- Virtual server names can't duplicate the name of a virtual server in the {{site.data.keyword.redhat_openshift_notm}} Virtualization environment.
- A certified and supported operating system.
