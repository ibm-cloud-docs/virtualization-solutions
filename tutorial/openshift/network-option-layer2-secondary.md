---

copyright:
  years: 2026
lastupdated: "2026-07-23"

keywords: Layer 2 secondary network OpenShift, CUDN configuration, UDN setup, OVN Layer 2 secondary, isolated networks OpenShift, OpenShift Virtualization networking

subcollection: virtualization-solutions

content-type: tutorial
services: OpenShift Virtualization
account-plan: paid
completion-time: 45m

---

{{site.data.keyword.attribute-definition-list}}

# Layer 2 secondary UDN examples for Red Hat OpenShift Virtualization
{: #layer-2-secondary-udn-examples-for-red-hat-openshift-virtualization}
{: toc-content-type="tutorial"}
{: toc-services="OpenShift Virtualization"}
{: toc-completion-time="45m"}


This tutorial shows how to create an example three-tier application by using Cluster-User-Defined Network (CUDN) Layer 2 secondary networks and namespaces on {{site.data.keyword.redhat_openshift_full}} Kubernetes&reg; Service on {{site.data.keyword.cloud_notm}}. The example demonstrates how to attach virtual servers that run on {{site.data.keyword.redhat_openshift_notm}} Virtualization to Layer 2 secondary networks that provide isolated network segments without external access. For more information on network types, see [OVN networking in {{site.data.keyword.redhat_openshift_notm}} for vSphere administrators](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-options-overview).
{: shortdesc}

This tutorial shows how to create an example three-tier application by using Cluster User-Defined Network (CUDN) Layer 2 secondary networks and namespaces on {{site.data.keyword.redhat_openshift_full}} Kubernetes Service on IBM Cloud. The example demonstrates how to attach virtual servers running on {{site.data.keyword.redhat_openshift_notm}} Virtualization to Layer 2 secondary networks that provide isolated network segments without external access. For more information about network types, see [Open Virtual Network (OVN) networking in OpenShift for vSphere administrators](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-network-options-overview).

## Overview
{: #layer2-secondary-overview}

The following guide demonstrates how to create an example three-tier application by using CUDN Layer 2 secondary networks and namespaces. The overall setup creates isolated network segments with no external access. Layer 2 secondary networks provide network isolation between different application tiers and allow controlled communication within and across namespaces.

### Network, namespace, and virtual server details
{: #layer2-secondary-network-details}

The following table summarizes the example network layout.

| CUDN                   | Namespace        | Role      | Subnet          | Virtual servers                |
| ---------------------- | ---------------- | --------- | --------------- | ------------------------------ |
| `white-isolated`       | `white`          | Secondary | 10.20.0.64/26   | `jail-web00`, `jail-db00`      |
| `black-white-isolated` | `white`, `black` | Secondary | 10.100.0.64/26  | `jail-db00`, `jail-app00`      |
{: caption="Example Layer 2 secondary CUDN, namespace, and virtual server layout" caption-side="bottom"}

### Diagram of the final setup
{: #layer2-secondary-diagram}

![Layer 2 secondary isolated network setup](../../images/openshift/openshift-virtualization-layer2-secondary-basic.png "Architecture diagram showing three virtual servers (jail-web00, jail-db00, and jail-app00) connected across White and black namespaces by using two Layer 2 secondary networks: White-isolated (10.20.0.0/24) and black-White-isolated (192.168.30.0/24)"){: caption="Layer 2 secondary isolated network setup" caption-side="bottom"}

You can copy each code block to a file and apply it with `oc apply -f <file-name>.yml`. Some code blocks might cross page boundaries in PDF and Word formats, and some long lines might wrap in those formats.
{: tip}

## Create namespace and CUDN Layer 2 secondary network
{: #layer2-secondary-white-namespace}
{: step}

Layer 2 secondary networks provide isolated network segments that can be attached to virtual servers as extra network interfaces. Unlike Layer 2 primary networks, secondary networks do not require special namespace labels and can coexist with the default pod network.

Create the `white` namespace and the `white-isolated` network. Save the following manifest to a file and apply it with `oc apply -f step0.yml`.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: white
---
apiVersion: k8s.ovn.org/v1
kind: ClusterUserDefinedNetwork
metadata:
  name: white-isolated
spec:
  namespaceSelector:
    matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: In
        values:
          - white
  network:
    topology: Layer2
    layer2:
      role: Secondary
      subnets:
        - 10.20.0.64/26
      ipam:
        lifecycle: Persistent
```
{: codeblock}

## Create another namespace and CUDN Layer 2 secondary network
{: #layer2-secondary-black-namespace}
{: step}

Create the `black` namespace and the `black-white-isolated` network that spans both the `white` and `black` namespaces. This configuration allows virtual servers in both namespaces to communicate on the same Layer 2 network segment. Save the following manifest to a file and apply it with `oc apply -f step1.yml`.

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: black
---
apiVersion: k8s.ovn.org/v1
kind: ClusterUserDefinedNetwork
metadata:
  name: black-white-isolated
spec:
  namespaceSelector:
    matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: In
        values:
          - white
          - black
  network:
    topology: Layer2
    layer2:
      role: Secondary
      subnets:
        - 10.100.0.64/26
      ipam:
        lifecycle: Persistent
```
{: codeblock}

## Create virtual servers in the namespaces
{: #layer2-secondary-create-vms}
{: step}

The following manifests create the three virtual servers from the overview table:

- `jail-web00` is in the `white` namespace and is attached to the `white-isolated` network.
- `jail-db00` is in the `white` namespace and is attached to both the `white-isolated` network and the `black-white-isolated` network. It bridges the two Layer 2 segments at the application level.
- `jail-app00` is in the `black` namespace and is attached to the `black-white-isolated` network.

Each virtual server attaches only to Layer 2 secondary networks. These virtual servers have no external access and must be accessed through the {{site.data.keyword.redhat_openshift_notm}} **web console**.

Update each manifest with a new password and `ssh_authorized_keys` value before you apply it.
{: important}

Apply each manifest with `oc apply -f <file-name>.yml`. Use these links to go to each manifest.

- [Virtual server `jail-web00` in the `white` namespace](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-secondary-udn-examples-for-red-hat-openshift-virtualization#layer2-secondary-vm-jail-web00)
- [Virtual server `jail-db00` in the `white` namespace](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-secondary-udn-examples-for-red-hat-openshift-virtualization#layer2-secondary-vm-jail-db00)
- [Virtual server `jail-app00` in the `black` namespace](/docs/virtualization-solutions?topic=virtualization-solutions-layer-2-secondary-udn-examples-for-red-hat-openshift-virtualization#layer2-secondary-vm-jail-app00)

### Virtual server `jail-web00` in the `white` namespace
{: #layer2-secondary-vm-jail-web00}

```yaml
# Bare YAML to stand up a {{site.data.keyword.redhat_openshift_notm}} virtual server
---
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: "jail-web00"
  namespace: "white"
  annotations:
    description: "example vm jail-web00"
  labels:
    app: "jail-web00-white-server"
    kubevirt.io/dynamic-credentials-support: 'true'
    vm.kubevirt.io/template: "centos-stream9-server-small"
    vm.kubevirt.io/template.namespace: openshift
    vm.kubevirt.io/template.revision: '1'
    vm.kubevirt.io/template.version: v0.34.0
spec:
  dataVolumeTemplates:
    - apiVersion: cdi.kubevirt.io/v1beta1
      kind: DataVolume
      metadata:
        creationTimestamp: null
        name: "jail-web00-white-server"
      spec:
        sourceRef:
          kind: DataSource
          name: "centos-stream9"
          namespace: openshift-virtualization-os-images
        storage:
          resources:
            requests:
              storage: 30Gi
  runStrategy: RerunOnFailure
  template:
    metadata:
      annotations:
        vm.kubevirt.io/flavor: "small"
        vm.kubevirt.io/os: "centos-stream9"
        vm.kubevirt.io/workload: "server"
      labels:
        kubevirt.io/domain: example
        kubevirt.io/size: "small"
    spec:
      domain:
        cpu:
          cores: 1
          sockets: 1
          threads: 1
        devices:
          disks:
            - disk:
                bus: virtio
              name: rootdisk
            - disk:
                bus: virtio
              name: cloudinitdisk
          interfaces:
            - name: white-net
              bridge: {}
              model: virtio
          networkInterfaceMultiqueue: true
          rng: {}
        memory:
          guest: "2Gi"
      hostname: "jail-web00"
      networks:
        - name: white-net
          multus:
            networkName: "white-isolated"
      terminationGracePeriodSeconds: 180
      volumes:
        - dataVolume:
            name: "jail-web00-white-server"
          name: rootdisk
        - cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: admin
              password: "aRandomPassword9292"
              ssh_authorized_keys:
                 - "ssh-ed25519 AAAA....BBBB....CCCCC.....DDDD put-your-key-here@example.local"
              chpasswd: { expire: False }
              runcmd:
                - [ dnf, install, -y, epel-release ]
                - [ dnf, install, -y, iperf3, nc, darkhttpd ]
          name: cloudinitdisk
```
{: codeblock}

### Virtual server `jail-db00` in the `white` namespace
{: #layer2-secondary-vm-jail-db00}

The `jail-db00` virtual server attaches to both Layer 2 secondary networks. You can reach it from `jail-web00` on the `white-isolated` network and from `jail-app00` on the `black-white-isolated` network.

```yaml
---
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: "jail-db00"
  namespace: "white"
  annotations:
    description: "example vm jail-db00"
  labels:
    app: "jail-db00-white-server"
    kubevirt.io/dynamic-credentials-support: 'true'
    vm.kubevirt.io/template: "centos-stream9-server-small"
    vm.kubevirt.io/template.namespace: openshift
    vm.kubevirt.io/template.revision: '1'
    vm.kubevirt.io/template.version: v0.34.0
spec:
  dataVolumeTemplates:
    - apiVersion: cdi.kubevirt.io/v1beta1
      kind: DataVolume
      metadata:
        creationTimestamp: null
        name: "jail-db00-white-server"
      spec:
        sourceRef:
          kind: DataSource
          name: "centos-stream9"
          namespace: openshift-virtualization-os-images
        storage:
          resources:
            requests:
              storage: 30Gi
  runStrategy: RerunOnFailure
  template:
    metadata:
      annotations:
        vm.kubevirt.io/flavor: "small"
        vm.kubevirt.io/os: "centos-stream9"
        vm.kubevirt.io/workload: "server"
      labels:
        kubevirt.io/domain: example
        kubevirt.io/size: "small"
    spec:
      domain:
        cpu:
          cores: 1
          sockets: 1
          threads: 1
        devices:
          disks:
            - disk:
                bus: virtio
              name: rootdisk
            - disk:
                bus: virtio
              name: cloudinitdisk
          interfaces:
            - name: white-net
              bridge: {}
              model: virtio
            - name: black-white-net
              bridge: {}
              model: virtio
          networkInterfaceMultiqueue: true
          rng: {}
        memory:
          guest: "2Gi"
      hostname: "jail-db00"
      networks:
        - name: white-net
          multus:
            networkName: "white-isolated"
        - name: black-white-net
          multus:
            networkName: "black-white-isolated"
      terminationGracePeriodSeconds: 180
      volumes:
        - dataVolume:
            name: "jail-db00-white-server"
          name: rootdisk
        - cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: admin
              password: "aRandomPassword9292"
              ssh_authorized_keys:
                 - "ssh-ed25519 AAAA....BBBB....CCCCC.....DDDD put-your-key-here@example.local"
              chpasswd: { expire: False }
              runcmd:
                - [ dnf, install, -y, epel-release ]
                - [ dnf, install, -y, iperf3, nc, darkhttpd ]
          name: cloudinitdisk
```
{: codeblock}

### Virtual server `jail-app00` in the `black` namespace
{: #layer2-secondary-vm-jail-app00}

```yaml
---
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: "jail-app00"
  namespace: "black"
  annotations:
    description: "example vm jail-app00"
  labels:
    app: "jail-app00-black-server"
    kubevirt.io/dynamic-credentials-support: 'true'
    vm.kubevirt.io/template: "centos-stream9-server-small"
    vm.kubevirt.io/template.namespace: openshift
    vm.kubevirt.io/template.revision: '1'
    vm.kubevirt.io/template.version: v0.34.0
spec:
  dataVolumeTemplates:
    - apiVersion: cdi.kubevirt.io/v1beta1
      kind: DataVolume
      metadata:
        creationTimestamp: null
        name: "jail-app00-black-server"
      spec:
        sourceRef:
          kind: DataSource
          name: "centos-stream9"
          namespace: openshift-virtualization-os-images
        storage:
          resources:
            requests:
              storage: 30Gi
  runStrategy: RerunOnFailure
  template:
    metadata:
      annotations:
        vm.kubevirt.io/flavor: "small"
        vm.kubevirt.io/os: "centos-stream9"
        vm.kubevirt.io/workload: "server"
      labels:
        kubevirt.io/domain: example
        kubevirt.io/size: "small"
    spec:
      domain:
        cpu:
          cores: 1
          sockets: 1
          threads: 1
        devices:
          disks:
            - disk:
                bus: virtio
              name: rootdisk
            - disk:
                bus: virtio
              name: cloudinitdisk
          interfaces:
            - name: black-white-net
              bridge: {}
              model: virtio
          networkInterfaceMultiqueue: true
          rng: {}
        memory:
          guest: "2Gi"
      hostname: "jail-app00"
      networks:
        - name: black-white-net
          multus:
            networkName: "black-white-isolated"
      terminationGracePeriodSeconds: 180
      volumes:
        - dataVolume:
            name: "jail-app00-black-server"
          name: rootdisk
        - cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: admin
              password: "aRandomPassword9292"
              ssh_authorized_keys:
                 - "ssh-ed25519 AAAA....BBBB....CCCCC.....DDDD put-your-key-here@example.local"
              chpasswd: { expire: False }
              runcmd:
                - [ dnf, install, -y, epel-release ]
                - [ dnf, install, -y, iperf3, nc, darkhttpd ]
          name: cloudinitdisk
```
{: codeblock}

## Test the network setup
{: #layer2-secondary-test}
{: step}

From the {{site.data.keyword.redhat_openshift_notm}} **web console**, log in to the virtual servers to test network connectivity.

### Test scenarios
{: #layer2-secondary-tests}

Run the following tests to verify the network isolation and connectivity:

1. Test connectivity within the White-isolated network

   Ping from `jail-web00` to `jail-db00`. This test succeeds because both virtual servers are on the same `white-isolated` network.

2. Test connectivity within the black-White-isolated network

   Ping from `jail-db00` to `jail-app00`. This test succeeds because both virtual servers are on the same `black-white-isolated` network that spans the `white` and `black` namespaces.

3. Test network isolation between different Layer 2 networks

   Ping from `jail-web00` to `jail-app00`. This test fails because these virtual servers are on different Layer 2 secondary networks (`white-isolated` and `black-white-isolated`) with no routing between them.

These tests demonstrate the network isolation that Layer 2 secondary networks provide. Virtual servers can communicate within the same Layer 2 network segment, but traffic cannot flow between different Layer 2 networks without additional routing configuration.
{: note}

## Next steps
{: #layer2-secondary-next-steps}

After you complete this tutorial, you can:

- Add more virtual servers to the existing networks
- Create more Layer 2 secondary networks for different application tiers
- Implement network policies to further control traffic within the networks
