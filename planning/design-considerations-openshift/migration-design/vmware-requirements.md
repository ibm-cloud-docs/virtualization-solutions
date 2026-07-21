---

copyright:
  years: 2025, 2026
lastupdated: "2026-07-21"

keywords: VMware to OpenShift migration requirements, VMware vSphere 6.5 migration, ESXi host migration prerequisites, VMware Tools requirements, VM naming conventions OpenShift, vCenter migration checklist, OpenShift virtualization prerequisites, NFC service memory ESXi, virtual machine migration requirements

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# VMware requirements and prerequisites for migrating to Red Hat OpenShift Virtualization
{: #virt-sol-openshift-migration-design-migration-vmware}

Check VMware vSphere, ESXi, and VM naming requirements before migrating to Red Hat OpenShift Virtualization on IBM Cloud.
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
- Virtual server names can't duplicate the name of a virtual server in the Red Hat OpenShift Virtualization environment.
- A certified and supported operating system.
