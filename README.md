# k8s-cluster
Kubernetes cluster setup.

This setup comprises of three nodes: the control-plane (master) and two worker nodes. `kubeadm` is used for setting up the cluster.

The instructions below are a high level. This setup assumes basic knowledge of Linux, Ubuntu in particular and how VirtualBox works.


```
This is a scratchpad for learning and testing my scripts. It also serves as a reminder and a handy store for me find and recall things easily, hopefully.

It is not particular useful for anyone and it should not be used for a Production environment.
```

## Required Software
```
VirtualBox: 5.0.20
Ubuntu: 20.04 LTS
Containerd: github.com/containerd/containerd 1.5.9-0ubuntu1~20.04.4
Kubernetes: 1.24.3

Calico network overlay - cniVersion: 0.3.1
Source: https://projectcalico.docs.tigera.io/manifests/calico.yaml
```
## Install Software
### Create the Master Node Virutal Machine
Create a virtual machine in VirtualBox with 2 vCPUs and 2Gb of memory. 
***
The host is a very old (2014/15?) MacBook Pro with a Intel i5 Dual Core with 8Gb of RAM - enough grunt for learning but not enough for anything for it to be useful.
***

Start up the virtual machine.
Install the required software:
```
sudo apt update
sudo apt install -y kubeadm kubelet kubectl containerd

# Prevent future upgrade
sudo apt-mark hold kubeadm kubelet kubectl containerd 
```

Reserve three IP addresses in your network for the custer nodes.

Edit `/etc/hosts` and call this host `k8s-master` and the work nodes as `k8s-node1` and `k8s-node2`.

Change the host name:

`sudo hostnamectl set-hostname k8s-master`

Edit the `/etc/netplan/00-netplan` and apply the change `sudo netplan apply`.

## Prepare Worker Nodes
Shutdown the master node.

Make two clones in VirtualBox, from the master node.

If the host has a lot of CPU cores and RAM, keep the settings as is. Otherwise, be frugal - downsize the vCPU to 1 and RAM to 1Gb for each of the worker node.

Change the hostnames for the two nodes, k8s-node1 and k8s-node2.

Update the netplan on each node.

Restart the nodes and make sure the hostnames, IP addresses are correct.

Ensure HTTP port 443 and SSH port 22 are open and traffic can flow between the nodes and the control plane.

## Create the Cluster

Log on the master node. 

Execute:

`sudo kubeadm init --service-cidr 172.16.0.0/16`

where `--service-cidr` is optional. If not specified, `10.96.0.0/12` is use as the default.

Source: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/

After the cluster has been created successfully, kubeadm display a set of instructions.

## Join the Cluster

Log each worker node and execute the kubeadm join command provided in the previous step.

For example:

`kubeadm join <master-node-ip:6443> --token <token> \
	--discovery-token-ca-cert-hash <sha256:hash>`

The token is valid for 24 hours. If the 24 hours grace time has been passed or the token is lost, generate a new token from the master-node.

## Generate a Kubeadm Join Token

Log on the master node.

List the current tokens:

`kubeadm token list`

Create token:

`kubeadm token create --print-join-command`

Log on each worker node and use the generated kubeadm join command to join the worker node to the cluster.

## List the Nodes

Log on the master node.

List the nodes in the cluster.

```
kubectl get nodes
NAME         STATUS   ROLES           AGE   VERSION
k8s-master   Ready    control-plane   02h   v1.24.3
k8s-node1    Ready                    01h   v1.24.3
k8s-node2    Ready                    01h   v1.24.3
```
Notice the `ROLES` column, the master node is labeled as `control-plane` and the worker nodes have no label on them.

Lets label them.

```
kubectl label node k8s-node1 node-role.kubernetes.io/worker=worker
kubectl label node k8s-node2 node-role.kubernetes.io/worker=worker
```

Now list the nodes again.

```
kubectl get nodes
NAME         STATUS   ROLES           AGE   VERSION
k8s-master   Ready    control-plane   03h   v1.24.3
k8s-node1    Ready    worker          04h   v1.24.3
k8s-node2    Ready    worker          04h   v1.24.3
```
