# smarter-cni - A Kubernetes CNI for IoT with Edge Compute

## README

This repo contains information required to install and build the smarter-cni.

## About smarter-cni
### Networking
The networking configuration for a node (Edge Gateway) using smarter-cni can be viewed in two ways:

* External view: the physical network interfaces (ethernet, wifi, cellular, etc.) on the node are managed by the network that each interface is connected to. The system makes no assumptions about the IP addresses provided or DNS names for the node. It is expected that at least one interface provides access to the Internet so that the node can connect to the cloud-based Kubernetes server. We assume that the external interfaces of the node will be externally configured by DHCP, BOOTP, etc.

* Internal view: smarter-cni uses a Linux bridge network to which all the Kubernetes pods are connected via virtual interfaces (only pods that use host networking do not have a virtual interface). Each deployed (non-host networked) pod has an interface allocated from this network, receiving an allocated address from within the range of the network.

### DNS

A separate repo: <https://gitlab.com/smarter-project/smarter-dns> provides a DNS server that enables Kubernetes pods to discover the IP address of pods running on the same node via their "hostname" (as defined in the Pod YAML description).
Processes runnning natively on the node can query this DNS server also.


## Deployment

### On the node
Deploying this container onto a Kubernetes node will provide the CNI binaries and configuration used by the Container Runtime Interface (CRI) to deploy Kubernetes pods on that node. 
For this reason smarter-cni would usually be one of the first deployed pods on a node.

For smarter-cni to be scheduled to run on a node, the node must have the 'smarter.cni=deploy' label
This can be added using:

`k3s kubectl label <NODENAME> smarter.cni=deploy`

### On the server

#### Using helm

A helm chart is provided at chart directory of the repository
```
helm install smarter-cni chart
```

#### Using Yaml file

The YAML file `smartercni_ds.yaml` contains a ConfigMap "smartercniconfig" which contains some default values for Linux bridge created by smarter-cni: 

* `172.39.0.0/16` for the subnet
* `172.39.0.1` for the gateway for that subnet. 

*The subnet value must match the `cluster-cidr` value provided to the Kubernetes server.*

These values are substituted into the configuration file for the bridge network created by snmarter-cni: `00-smarter-bridge.conf`


Deploy the smarter-cni DaemonSet using the smartercni_ds.yaml. A smarter-cni Pod should be created on every node in the cluster with the `smarter.cni=deploy` label. For example using k3s:

	k3s kubectl apply -f smartercni_ds.yaml


## Building the smarter-cni container image

checkout the repo: 

    git clone https://gitlab.com/smarter-project/smarter-cni.git

The easiest way to do this is by using the multi-arch building functionality in docker (an experimental feature)

    docker buildx create --use --name mybuild
    cd build
    docker buildx build --platform linux/arm64/v8,linux/arm/v7,linux/amd64 -t registry.gitlab.com/smarter-project/smarter-cni:vX.X --push .



