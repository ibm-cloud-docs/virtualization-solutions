## Network options for IBM ROKS

IBM ROKS offers two styles of networking in IBM Cloud VPC.

-   Networking options:

    Callico: Oriented around Pod / Container Masquerade networks. Not well suited for virtual machines. Callico has been supported for a long time and has many built in features. A pod or VM can only have one network attachment in Callico. 

    OVN: Allows for both Pod / Container Masquerade networks and Cluster user defined networks. Masquerade mode works like Callico and its suited for pods. In Layer 2 mode a/k/a (L2); L2 networks are like VMWare NSX-T Overlay Segments and well suited for Virtual machines. L2 nets allow for arbitrary IP Subnets to be used, BYOP etc. L2 nets can have DHCP or Static IP assignments. Each Namespace / Group can have a single primary L2 and many secondary L2 networks. Primary networks can be advertised via BGP to an external router. Secondays are isolated to the namespace.
    
-   Network controller choice must be selected at time of cluster creation. VSI based setups will allow for eiter option default is Callico. Bare metal will defalut to OVN.

-   OVN is not as full featured as NSX-T and, will depend on Routers , Firewalls and Load balancers running in VPC or ROKS to support firewalling, custom routes, GRE tunnels etc. 

-   ROKS hosted Firewalls can connect Primary and Secondary L2 Nets and Masquerade, however VPC hosted devices will only see L2 Primary networks.

-   L2 Networks allow for network migrations from NSX-T and V overlay segments. However, L2 network are not a replacement for Vlan segments. Vlan segments need Localnet or VirtualMachine networks coming in a future release.

-   Future VPC options, including DRS will improve L2 network connectivity. Allowing Customers to have BGP as a service in their VPC.

-   Stretch networking spanning two availability zones will be supported in a future release and will depend on DRS or a VPC hosted firewall / router that support cross zone high availability.



