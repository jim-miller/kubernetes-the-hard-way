# Provisioning Nodes

Kubernetes requires a set of machines to host the Kubernetes control plane and the worker nodes where containers are
ultimately run. In this lab you will provision the Raspberry Pi nodes required for running a secure and highly available
Kubernetes cluster within an isolated lab network.

## Networking

The Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#kubernetes-model) assumes a flat network in which containers and nodes can communicate with each other. In cases where this is not desired [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) can limit how groups of containers are allowed to communicate with each other and external network endpoints.

> Setting up network policies is out of scope for this tutorial.

### Local Private Network

The network sits behind the OpenWRT router and is in effect a VPC.

In this section a dedicated [Virtual Private Cloud](https://cloud.google.com/compute/docs/networks-and-firewalls#networks) (VPC) network will be setup to host the Kubernetes cluster.

Create the `kubernetes-the-hard-way` custom VPC network:

```
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
```

A [subnet](https://cloud.google.com/compute/docs/vpc/#vpc_networks_and_subnets) must be provisioned with an IP address range large enough to assign a private IP address to each node in the Kubernetes cluster.

Subnet range for the lan is 192.168.1.0/24

> The `192.168.1.0/24` IP address range can host up to 254 compute instances.

### Firewall Rules

By default, OpenWRT allows internal communication across all protocols.

TODO: create a firewall rule that allows external SSH, ICMP, and HTTPS. I.e. mimic the following:

```
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp                Fals
```

### Kubernetes Public IP Address

Allocate a static IP address that will be attached to the external load balancer fronting the Kubernetes API Servers:

Verify the IP address of OpenWRT's wan interface.

## Compute Instances

The compute instances in this lab will be provisioned using [Ubuntu Server](https://www.ubuntu.com/server) 20.04, which has good support for the [containerd container runtime](https://github.com/containerd/containerd). Each compute instance will be provisioned with a fixed private IP address to simplify the Kubernetes bootstrapping process.


### Set up SSH Keyset to Access Nodes

The nodes will allow passwordless access to nodes that can be pre-configured before flashing the nodes' SD cards. First
let's create an SSH keyset to be used for this access.

```
ssh-keygen -t ed25519 -f ~/.ssh/k8s_cluster_access
```

### Flashing the Kubernetes Controllers and Workers

Two controllers: `{controller-{0,1}` and three workers: `worker-{0,1,2}`

You are able to set the hostname and SSH access with the Raspberry PI Imager while flashing the new image. Once you
select an image, press CTRL-SHIFT-X to bring up advanced options.

For each controller and worker:
  * Select `Set hostname:` worker-0
  * UNSELECT `Configure wifi`
  * `Set authorized_keys for pi` <paste the output of ~/.ssh/k8s_cluster_access.pub here>

### Boot up the Nodes

Insert SD cards, power on.

### Verification

From the OpenWRT Status dashboard ensure `Active DHCP Leases` contain the followin:

```
gcloud compute instances list --filter="tags.items=kubernetes-the-hard-way"
```

> output

```
Hostname        IPv4 address     MAC address          Lease time remaining
controller-0    192.168.1.XXX    DC:A6:32:D6:XX:XX    10h 51m 34s
controller-1    192.168.1.XXX    DC:A6:32:D6:XX:XX    10h 51m 34s
worker-0        192.168.1.XXX    DC:A6:32:D6:XX:XX    10h 51m 34s
worker-1        192.168.1.XXX    DC:A6:32:D6:XX:XX    10h 51m 34s
worker-2        192.168.1.XXX    DC:A6:32:D6:XX:XX    10h 51m 34s
```

## Configuring SSH Access

SSH will be used to configure the controller and worker instances.

TODO: Detail the following
  * Create a local SSH key pair.
  * Add a ssh config file to use that key with controller-X and worker-X
  * SCP public key to controller and node instances
  * Add public key to `~/.ssh/authorized_keys`

Test SSH access to the `controller-0` node:

```
ssh controller-0
```

```
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-1042-gcp x86_64)
...
```

Type `exit` at the prompt to exit the `controller-0` compute instance:

```
$USER@controller-0:~$ exit
```
> output

```
logout
Connection to XX.XX.XX.XXX closed
```

Next: [Provisioning a CA and Generating TLS Certificates](04-certificate-authority.md)
