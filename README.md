# Kubernetes NOOB Guide on Arch Linux

This is my personal guide / notes on installing Kubernetes Cluster on Arch Linux.

## Installation
1. Install the package group `kubernetes-control-plane`. This will install all of the needed packages for the master node.

```bash
$ sudo pacman -Syu kubernetes-control-plane
```

Install `worker-node` package group for the worker nodes.

```bash
$ sudo pacman -Syu worker-node
```

2. Turn off swap. Depending on your setup you may need to permanently disable swap in `/etc/fstab`. Do this also for all of the nodes.

```bash
$ sudo swapoff -a
```

3.  Install a container package. For this guide, I have used `containerd`. Do this for all of the nodes.

```bash
$ sudo pacman -Syu containerd
```

4. Run `containerd` pre-requisites as documented [here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd). Do this for all of the nodes.

```bash
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ sudo sysctl net.ipv4.ip_forward=1
$ sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

5. Initialize `kubeadm` on the master node. This is based on flannel network [guide](https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md). Do this only for the master node.

```bash
$ kubeadm init --pod-network-cidr='10.244.0.0/16'
```

Copy the log produced from this command (i.e. kubeadm join ...) you will need it later.

6. During the time of writing, the previous command had instructions to create a configuration by entering the following commands. Do this only for the master node.

```bash
$ mkdir -p $HOME/.kube 
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

7. Update the `kubeadm` configuration to use the correct cni binary directory. For some reason Arch Linux has a different cni bin dir structure.

```bash
$ vim /var/lib/kubelet/kubeadm-flags.env
```

8. Restart the `kubelet` service. Do this for all of the nodes.

```bash
$ sudo systemctl restart kubelet.
```

9. Reboot the machine. Do this for all of the nodes.

```bash
$ sudo reboot
```

10. Create the pod network. This guide uses the flannel pod network. Do this only for the master node.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

11. Join the worker nodes. The join command is found from the logs produced in the `kubeadm init` command from the master node.

```bash
$ kubeadm join --token xxx --discover-token-ca-cert-hash xxx
```

## MetalLB Installation
This is a software based load balancer. Documentation can be found [here](https://metallb.universe.tf/installation/)

1. Create the namespace

```bash
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
```

2. Install Metal LB

```bash
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
```

3. Create member list
```bash
$ kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

## NGINX Ingress controller
1. Install NGINX ingress controller. The cloud version is applied here instead of baremetal because Metal LB is used.
```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
```

2. For some reason creating ingresses gets hung up because of a webhook. It's an open bug in the controller. Details are found [here](https://github.com/kubernetes/ingress-nginx/issues/5401#issuecomment-662424306). Below is the suggested workaround:
```bash
$ kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

```

Links:
* https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/
