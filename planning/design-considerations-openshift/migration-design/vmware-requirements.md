---

copyright:
  years: 2025
lastupdated: "2026-01-13"

keywords:

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}


# VMware Migration requirements
{: #virt-sol-openshift-migration-design-migration-vmware}

If you migrate from a VMware environment to OpenShift, you must meet the following requirements before you migrate.
{: shortdesc}

**VMware environment requirements:**

- VMware vSphere must be version 6.5 or later.
- If you are migrating more than 10 virtual machines from an ESXi host in the same migration plan, you must increase the NFC service memory of the host.

**Virtual machine requirements:**

- VMware Tools is installed.
- ISO/CDROM disks are unmounted.
- Each NIC must contain no more than one IPv4 and/or one IPv6 address.
- Virtual machine name contains only:
    - lowercase letters (a-z), numbers (0-9), or hyphens (-), up to a maximum of 253 characters.
    - The first and last characters must be alphanumeric.
    - The name must not contain uppercase letters, spaces, periods (.), or special characters.
- Virtual machine name can't duplicate the name of a virtual machine in the OpenShift Virtualization environment.
- Operating system is certified and supported.
