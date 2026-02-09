---

copyright:
  years: 2025, 2026
lastupdated: "2026-02-09"

keywords:

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# VMware migration requirements
{: #virt-sol-openshift-migration-design-migration-vmware}

If you migrate from a VMware environment to Red Hat OpenShift, you must meet the following requirements before you migrate.
{: shortdesc}

VMware environment requirements:

- VMware vSphere must be version 6.5 or later.
- If you are migrating more than 10 virtual servers from an ESXi host in the same migration plan, you must increase the NFC service memory of the host.

Virtual server requirements:

- VMware Tools is installed.
- ISO and CD-ROM disks are unmounted.
- Each NIC must contain no more than one IPv4 and or one IPv6 address.
- Virtual server names must contain only the following characters:
   - lowercase letters (a-z), numbers (0-9), or hyphens (-), up to a maximum of 253 characters.
   - The first and last characters must be alphanumeric.
- The name must not contain uppercase letters, spaces, periods (.), or special characters.
- Virtual server names can't duplicate the name of a virtual server in the Red Hat OpenShift Virtualization environment.
- A certified and supported operating system.
