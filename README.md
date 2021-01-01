# Kubernetes on Arch Linux

This is my personal guide / notes on installing Kubernetes Cluster on Arch Linux

## Installation
1. Install the packate group `kubernetes-control-plane`. This will install all of the needed packages for the master node.

```bash
$ sudo pacman -Syu kubernetes-control-plane
```

2. Turn off swap. Depending on your setup you may need to permanently disable swap in `/etc/fstab`

```bash
$ swapoff -a
```

3.  Install a container package. For this guide, I have used `containerd`

```bash
$ sudo pacman -Syu containerd
```

4. Run `containerd` pre-requisites as documented [here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

```bash
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ sudo sysctl net.ipv4.ip_forward=1
$ sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

5. Initialize `kubeadm` on the master node. This is based on flannel network [guide](https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md)

```bash
$ kubeadm init --pod-network-cidr='10.244.0.0/16'
```

Copy the log produced from this command (i.e. kubeadm join ...) you will need it later.

6. During the time of writing, the previous command had instructions to create a configuration by entering the following commands:

```bash
$ mkdir -p $HOME/.kube 
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

7. Update the `kubeadm` configuration to use the correct cni binary directory. For some reason Arch Linux has a different cni bin dir structure.

```bash
$ vim /var/lib/kubelet/kubeadm-flags.env
```

8. Restart the `kubelet` service

```bash
$ sudo systemctl restart kubelet
```

9. Reboot the machine

```bash
$ sudo reboot
```

10. Create the pod network. This guide uses the flannel pod network.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```


## NGINX Ingress controller
Links:
* https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/
