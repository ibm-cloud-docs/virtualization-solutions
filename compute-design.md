## Compute options for IBM ROKS

Virtual Machine / Compute for ROKS. Currently only supported in VPC BareMetal Gen2 hosts.

-   Supported on Gen 2 BareMetal profiles only Gen3 Support will be in a future release. Profile arability varies by location. See [Profile Availability](https://cloud.ibm.com/docs/openshift?topic=openshift-vpc-flavors#us-south-physical-table) for details.

-   Minimum of 3 similar hosts required per availability zone.
-   Hosts can operate at any VPC network speed 10Gb - 200Gb. For optimal results 50G or higher is recommended.
-   Host will use NVME drives for workload storage using Openshift ODF which is based on Ceph. Hosts need 4 or more NVME drives for ODF. Hosts with NFS storage for primary workloads are not supported currently. VPC block storage is not supported on baremetal hosts currently either.
-   Hosts will also only support a single VNIC / Server Network attachment. This will prevent the use of Public Address ranges, Vlan VNI and other newer VPC network features. VNI attachments will be supported in a future release. VNI support will allow for better segregation of workload and storage traffic, as well support for direct connection into a VPC subnet.
-   Clusters need to be dedicated to VM or Pod deployments. Mixed usage is not recommended by Redhat or IBM.
-   ~~IBM also recommends using IBM Fusion for cluster backups to S3, NFS or 3rd party systems. See [IBM Fusion](https://www.ibm.com/docs/en/storage-fusion/storage/2.6.0?topic=fusion-installing-storage-cloud) features for more information .~~
-   ROKS / Openshift Compute Supports many of the same features as VMWare vSphere as well as some new options. One of the new features for VM deployment is templates with predefined “T-Shirt” sizes (Small, Medium, Large, etc.). The sizes can be customized to fit user’s needs. IE Small + 200G of extra storage. VM sizes can also be arbitrary in size (6 cores, 10G ram, 5 disks, more than one network)

-   ROKS networks can also contain both POD and VM deployments as well as the reverse POD and VMS can attach to both L2 and Masquerade networks. When using OVN networking VMs can attach to more than one network.

-   VMs can have more than one drive attachment as well as access to shared storage. Drives can be resized on the fly, provided the VM’s Operating system supports it.

-   ROKS fully support Redhat’s Forlift migration tool allowing for migrations from vSphere to ROKS. However, at this time Forklift does not support migrations from VMWare’s vCloud Director. However, for single tenant setups IBM can provide customers direct access to their vSphere setup. For multi-tenant users automated support will be available in a future release, MT users will need assistance from IBM for access.



