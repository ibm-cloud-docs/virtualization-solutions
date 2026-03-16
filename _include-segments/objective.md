## Objective
{: #virt-sol-vpc-migration-tutorial-overview-objectives}

The following objectives are covered in the following tutorial.

- Create a VPC and the resources that it needs
- Set up {{site.data.keyword.vmwaresolutions_short}} instance with a Microsoft Windows&reg; virtual server and an RHEL virtual server to use as migration examples
- Build a private connection between {{site.data.keyword.vmwaresolutions_short}} and VPC through a Transit Gateway (TGW)
- Move your virtual server disks into VPC Block Storage volumes
- Create virtual servers from the migrated disks

Some VMware configuration steps, such as connecting your environment to VPC through a Transit Gateway, are different.
{: note}

The virtual servers that are part of this tutorial have the following constraints:

- They have a single disk size of less than 250 GB
- They are not Bring Your Own License (BYOL)
- They access the internet to download the necessary packages to perform the migration

The following diagram illustrates what you build as part of this tutorial.

![Virtualization Solutions VPC Migration Tutorial High-Level Architecture](../../images/vpc-vsi/vpc-vsi-migration-tutorial-high-level-diagram.svg){: caption="Virtualization Solutions VPC Migration Tutorial High Level Architecture" caption-side="bottom"}
