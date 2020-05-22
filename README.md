# smarter-cni - A Kubernetes CNI for IoT with Edge Compute

## README

This repo contains information required to build and install the smarter-cni.


## About smarter-cniKubernetes

### Networking
The networking configuration for a node (Edge Gateway) using smarter-cni can be viewed in two ways:

* External view: the physical network interfaces (ethernet, wifi, cellular, etc.) on the node are managed by the network that each interface is connected to. The system makes no assumptions about the IP addresses provided or DNS names for the node. It is expected that at least one interface provides access to the Internet so that the node can connect to the cloud-based Kubernetes master. We assume that the external interfaces of the node will be externally configured by DHCP, BOOTP, etc.

* Internal view: smarter-cni uses a Linux bridge network to which all the Kubernetes pods are connected via virtual interfaces (only pods that use host networking do not have a virtual interface). Each deployed pod has an interface allocated from this network, receiving an allocated address from within the range of the network.

### DNS

A separate repo: <https://gitlab.com/arm-research/smarter/smarterdns> provides a DNS server that enables Kubernetes pods to discover the IP address of pods running on the same node via their "hostname" (as defined in the Pod YAML description).
Process runnning natively on the node can query this DNS server also.


## Building the container image

checkout the repo: 

    git clone git@gitlab.com:arm-research/smarter/smarter-cni.git
    git checkout cadeniyi


The easiest way to do this is by using the multi-arch building functionality in docker (an experimental feature)

    docker buildx create --use --name mybuild
    cd build
    docker buildx build --platform linux/arm64/v8,linux/arm/v7,linux/amd64 -t registry.gitlab.com/arm-research/smarter/smarter-cni:v0.2 --push .

The file `build/bridge.conf` contains the configuration for the bridge network created for smarter-cni. The "subnet" parameter must match the `--cluster-cidr' value used when starting the Kubernetes master. The "gateway" parameter must match the subnet appropriately.


## Deployment

### On the node
Deploying this container onto a Kubernetes node will provide the CNI binaries used by the Container Runtime Interface (CRI) to deploy Kubernetes pods on that node.

The container runtime (usually containerd which we assume is installed) must be configured to find the CNI plugin binaries. This is often done by editing the file:  `/etc/containrd/config.toml`

If this file does not exist then it can be genrerated running:

    sudo containerd config default > /etc/containerd/config.toml

Then edit the section for the CRI CNI plugins to:

    [plugins.cri.cni]
          bin_dir = "/host/opt/cni/bin"
          conf_dir = "/host/etc/cni/net.d"
          conf_template = ""


### On the master

Deploy the smartercni DaemonSet using the smartercni_ds.yaml. A smarter-cni Pod should be created on every node in the cluster.

