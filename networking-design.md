---
copyright:
  years: 2025
lastupdated: "2025-11-25"

keywords: ROKS, OpenShift Data Foundation, ODF, File Storage, Block Storage, Encryption

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

## Network options for IBM ROKS

{: \#network-design-roks}

IBM ROKS offers two styles of networking in IBM Cloud VPC.

-   Networking options:

    Callico: Oriented around Pod / Container Masquerade networks. Not well suited for virtual machines. Callico has been supported for a long time and has many built in features. A pod or VM can only have one network attachment in Callico.

    OVN: Allows for both Pod / Container Masquerade networks and Cluster user defined networks. Pod Networking mode works like Callico’s Masquerade mode and it’s suited for pods. User Defined Layer 2 mode a/k/a (L2); L2 networks are like VMWare NSX-T Overlay Segments and well suited for Virtual machines. L2 nets allow for arbitrary IP Subnets to be used, BYOP etc. L2 nets can have DHCP or Static IP assignments. Each Namespace / Group can have a single primary L2 and many secondary L2 networks. Primary networks can be advertised via BGP to an external router. Secondaries are isolated to the namespace and will need a hosted router to allow for egress from the cluster.

-   Network controller choice must be selected at time of cluster creation. VSI based setups will allow for either option default is Callico. Bare metal will default to OVN.

-   OVN is not as full featured as NSX-T and, will depend on Routers, Firewalls and Load balancers running in VPC and or ROKS to support firewalling, custom routes, GRE tunnels etc.

-   ROKS hosted Firewalls can connect Primary and Secondary L2 Nets and Pod networks, however VPC hosted devices will only see L2 Primary networks.

-   L2 Networks can allow for some network migrations from NSX-T and V overlay segments. However, L2 network are not a replacement for VLAN segments. DNAT, and Complex SNAT and VLAN attachments will depend on Localnet and Cross account VPC VLAN VNI, support slated for mid 2026.

-   Future VPC options, including DRS will improve L2 network connectivity. Allowing Customers to have BGP as a service in their VPC.

-   Stretch networking spanning two availability zones will be supported in a future release and will depend on DRS or a VPC hosted firewall / router that support cross zone high availability.

## IBM Cloud VPC

{: \#vpc-networking}

With IBM Cloud VPC, a private networking space can be created on IBM’s public cloud.

### Default private networking with subnets

{: \#vpc-networking-subnets}

VPCs span a region and are divided into subnets spanning individual zones within that region. These subnets use a range of private IP addresses, with the option to bring your own public IP range. Subnets within a VPC are private by default and are able to talk to each other without setting up any routes. As a result, all resources within a VPC are able to communicate to one another. VSIs are attached to one or more subnets. See the architecture diagram in [About networking](https://cloud.ibm.com/docs/vpc?topic=vpc-about-networking-for-vpc) for a visual representation of the VPC networking concepts. For additional information and design considerations, see [Setting IP ranges](https://cloud.ibm.com/docs/vpc?group=ip-ranges).

### External connectivity

{: \#vpc-networking-external-connectivity}

External connectivity can be achieved with public gateways, floating IPs, and VPNs. Public gateways span an entire subnet and all VSIs attached to it and support initiating connections to the internet. Floating IPs span a single VSI and support initiating connections to and receiving connections from the internet. The IBM Cloud VPN for VPC service supports secure connectivity from a VPC to another private network. See [About site-to-site VPN gateways](https://cloud.ibm.com/docs/vpc?topic=vpc-using-vpn) for additional details on working with VPNs.

### Security

{: \#vpc-networking-security}

Networking security can be controlled with security groups and access control lists. Access control lists manage traffic at the subnet level, whereas security groups manage traffic at the VSI level. Access control lists therefore can gate external connectivity established via a public gateway on a subnet, while security groups can be used to gate external connectivity established via a floating IP on a VSI. See [Security in your VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-security-in-your-vpc) for additional details.

### Interconnectivity

{: \#vpc-networking-interconnectivity}

Interconnecting a VPC with an on-premises network can be done with IBM Cloud Direct Link, while interconnecting VPCs to each other and various other resources can be done with IBM Cloud Transit Gateway. Connecting a VPC with IBM Cloud classic infrastructure can also be done with IBM Cloud Transit Gateway. To determine the appropriate IBM Cloud Direct Link offering for your workload and learn more about interconnectivity with IBM Cloud VPC, see [Interconnecting your VPC using IBM Cloud offerings](https://cloud.ibm.com/docs/vpc?topic=vpc-interconnectivity).
