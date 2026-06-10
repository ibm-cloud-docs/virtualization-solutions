---

copyright:
  years: 2026
lastupdated: "2026-06-10"

keywords: Red Hat OpenShift Virtualization, VPC, Localnet, CUDN, UDN, VLAN, VNI, OVN

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization
account-plan: paid
completion-time: 60m

---

{{site.data.keyword.attribute-definition-list}}

# Red Hat OpenShift on IBM Cloud OVN UDN/CUDN Network prerequisites
{: #udn-prerequisites}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="60m"}

This tutorial shows how to set up the prerequisites for Cluster User-Defined Network (CUDN) Localnets, and Layer 2 Primaries on on {{site.data.keyword.redhat_openshift_full}} Kubernetes Service on IBM Cloud. All tasks described in this tutorial are performed once per cluster. 
{: shortdesc}

## Localnet overview
{: #localnet-prerequisites}

Red Hat OpenShift Kubernetes Service Localnet networks require additional setup that customers do not perform in ROVS. This tutorial explains the required steps. 

### Step 0: Prepare the Red Hat OpenShift on IBM Cloud cluster
{: #localnet-step-0-prepare-roks}

The Red Hat OpenShift on IBM Cloud cluster requires the NMState operator. NMState provides persistent network configuration for bridges and physical networks. This step installs the operator.

Keep the following considerations in mind:

- The installation takes 5 to 10 minutes to complete after you apply the manifest.
- Create only one bridge per interface.
- Do not attempt to change the bridge on `eth0`.

Copy the following code block to a file and apply it with `oc apply -f step0a.yml` to install the NMState operator.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-nmstate
spec:
  finalizers:
  - kubernetes
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-nmstate
  namespace: openshift-nmstate
spec:
  targetNamespaces:
  - openshift-nmstate
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: kubernetes-nmstate-operator
  namespace: openshift-nmstate
spec:
  channel: stable
  installPlanApproval: Automatic
  name: kubernetes-nmstate-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```
{: codeblock}

Wait for the operator to register the `NMState` CRD before you create the `NMState` custom resource. If you apply the `NMState` resource before the CRD is established, you see an error such as `no matches for kind "NMState" in version "nmstate.io/v1"`.

Run the following commands to poll until the CRD appears, and then wait for it to reach the `Established` condition.

```sh
until oc get crd nmstates.nmstate.io >/dev/null 2>&1; do echo "Waiting for NMState CRD..."; sleep 10; done
oc wait --for=condition=established crd/nmstates.nmstate.io --timeout=300s
```
{: pre}

Verify that all NMState CRDs are installed.

```sh
oc get crd | grep nmstate
```
{: pre}

Example output:

```text
nmstates.nmstate.io                                               ...
nodenetworkconfigurationenactments.nmstate.io                     ...
nodenetworkconfigurationpolicies.nmstate.io                       ...
nodenetworkstates.nmstate.io                                      ...
```
{: screen}

After the CRDs are ready, copy the following code block to a file and apply it with `oc apply -f step0b.yml` to create the `NMState` custom resource.

```yaml
---
apiVersion: nmstate.io/v1
kind: NMState
metadata:
  name: nmstate
spec:
  probeConfiguration:
    dns:
      host: root-servers.net
```
{: codeblock}

## Step 1: Apply the bridge configuration
{: #localnet-step-1-bridge}

Apply the basic bridge configuration to use the second PCI-VNI NIC (`eth1`).

Copy the following code block to a file and apply it with `oc apply -f step1.yml`.

```yaml
---
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: "br-eth1"
spec:
  desiredState:
    interfaces:
    - name: "br-eth1"
      description: A dedicated OVS bridge with a NIC as a port
      type: ovs-bridge
      state: up
      bridge:
        allow-extra-patch-ports: true
        options:
          stp: false
        port:
        - name: "eth1"
```
{: codeblock}

Wait for this config to applied to all nodes before moving on to the next step. 

```sh
% oc get NodeNetworkConfigurationPolicy br-eth1
NAME      STATUS      REASON
br-eth1   Available   SuccessfullyConfigured
```
{: codeblock}

## Step2: Apply the bridge mapping configuration
{: #localnet-step-2-bridge}

Apply the basic bridge mapping configuration .

Copy the following code block to a file and apply it with `oc apply -f step2.yml`.

```yaml
---
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: "vpc-vlans"
spec:
  desiredState:
    ovn:
      bridge-mappings:
      - localnet: "vpc-vlans" # <- use your desired network name
        bridge: "br-eth1"
        state: present
```
{: codeblock}

***

## Creating a custom secondary prefix and subnets for your VPC
{: #optional-subnets}

Localnets can use secondary VPC subnets for each network. In this example, three new subnets from a new VPC prefix are used.
You can substitute any prefix or network, or use the VPC defaults.

### Step1: Create a new VPC Prefix
{: #subnet-step-1}

1. Create a new prefix for each zone. This example uses the supernet `198.18.0.0/16` and creates three `/18` prefix (subnet), one per zone.

   ```sh
   ic is vpc-addrc my-vpc-zone-1-vms my-vpc us-south-1 198.18.0.0/18 --default true
   ic is vpc-addrc my-vpc-zone-2-vms my-vpc us-south-2 198.18.64.0/18
   ic is vpc-addrc my-vpc-zone-3-vms my-vpc us-south-3 198.18.128.0/18
   ```
   {: pre}

### Step2: Create new VPC Subnets
{: #subnet-step-2}

2. Create three vpc subnets `www`, `db`, and `app` for zone 1.

   ```sh
   ic is subnetc zone1-vm-www my-vpc us-south-1 --ipv4-cidr-block 198.18.0.0/24
   ic is subnetc zone1-vm-db my-vpc us-south-1 --ipv4-cidr-block 198.18.1.0/24
   ic is subnetc zone1-vm-app my-vpc us-south-1 --ipv4-cidr-block 198.18.2.0/24
   ```
   {: pre}

### Step3: Attach a public gateway as needed
{: #subnet-step-3}

3. Ensure that you have a public gateway attached to each zone. Otherwise, the subnet is private only.

   Get the ID for each subnet. In this example, the `zone1-vm-www` subnet is selected.

   ```sh
   ic is subnets | awk ' $8 ~ /yourVPC-NAME_GOES-HERE/'
   ```
   {: pre}

   Example output:

   ```text
   0716-16da748a-5159-480c-b809-5f9e4750028b   zone1-vm-app    available   198.18.2.0/24   249/256   stone-likeness-bagging-headcount   pgw-0ec02020-2f74-11f1-b0c5-bfe21078e55b   potato-farm   us-south-1   Default
   0716-74a2d06b-de5c-40c2-8311-38dbff74ce62   zone1-vm-db     available   198.18.1.0/24   249/256   stone-likeness-bagging-headcount   -                                          potato-farm   us-south-1   Default
   0716-efd14367-eed3-429d-b390-ea17f0e8cfb1   zone1-vm-www   available   198.18.0.0/24   249/256   stone-likeness-bagging-headcount   pgw-0ec02020-2f74-11f1-b0c5-bfe21078e55b   potato-farm   us-south-1   Default
   ```
   {: screen}

### Step4: Attach a public gateway as needed to the subnet
{: #subnet-step-4}

4. Attach a public gateway to the `zone1-vm-www` subnet by using the subnet ID `0716-efd14367-eed3-429d-b390-ea17f0e8cfb1`.

   ```sh
   ic is subnet-public-gateway-attach zone1-vm-www --pgw r134-053cb8b1-d966-4b02-9f20-5fcbeda76048
   ```
   {: pre}

You must repeat this step for each new subnet and each zone. Public gateways are per VPC and per zone.
{: note}

***


## Layer 2 Primary plumbing
{: #optional-layer2-plumbing }

Layer 2 Primary networks require some initial plumbing to be done before a end user can make use of them . 


### Step1: Install the FRR OVN Router pod
{: #layer2-pri-step-1}

The Red Hat OpenShift on IBM Cloud cluster must be patched to expose the FRR option in the OVN pod. Apply the following patch to enable FRR as an additional routing provider and to enable route advertisements in the OVN-Kubernetes configuration.

```sh
oc patch Network.operator.openshift.io cluster --type=merge -p='{"spec":{"additionalRoutingCapabilities":{"providers":["FRR"]},"defaultNetwork":{"ovnKubernetesConfig":{"routeAdvertisements":"Enabled"}}}}'
```
{: pre}

### Step2: Enable Inter-Namespace networking. 
{: #layer2-pri-step-2}

Next, apply the base network configuration that permits inter-namespace networking for CUDNs that are advertised through BGP. This configuration relaxes the default UDN isolation behavior. Save the following manifest to a file and apply it with `oc apply -f step0.yml`.

```yaml
# Permit intra-CUDN-namespace traffic
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: openshift-network-operator
  name: ovn-kubernetes-config-overrides
data:
  advertised-udn-isolation-mode: "loose"
```
{: codeblock}

### Step3: Enable Route advertisements selector. 
{: #layer2-pri-step-3}

Next, add route advertisements to tell OVN to advertise pod networks through BGP with the OVN FRR pod. Save to a file and apply it with `oc apply -f step1.yml`.
_Note:_ Networks need to have the label "advertise: true"


```yaml
# Add the RouteAdvertisements
---
kind: RouteAdvertisements
apiVersion: k8s.ovn.org/v1
metadata:
  name: default
spec:
  nodeSelector: {}
  frrConfigurationSelector: {}
  networkSelectors:
    - networkSelectionType: ClusterUserDefinedNetworks
      clusterUserDefinedNetworkSelector:
        networkSelector:
          matchLabels:
            advertise: "true"
  advertisements:
    - "PodNetwork"
---
```
{: codeblock}


