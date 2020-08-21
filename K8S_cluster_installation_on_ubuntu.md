### Kubernetes Single node Cluster installation on Ubuntu 16.04



#### Install Docker

Please follow the procedure to install the Docker : https://github.com/shakeer-linux/docker


**Disable Swap memory:**

Note: If swap is not disabled, kubelet service will not start on the masters and nodes.

```
root@ubuntu:~# swapoff -a
```

```
root@ubuntu:~# cat /etc/fstab  | grep swap
/dev/mapper/ubuntu--vg-swap_1 none            swap    sw              0       0
root@ubuntu:~# swapoff -a
```

```
root@ubuntu:~# sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

```
root@ubuntu:~# cat /etc/fstab  | grep swap
#/dev/mapper/ubuntu--vg-swap_1 none            swap    sw              0       0
root@ubuntu:~#
```



#### Note: 
Set ***/proc/sys/net/bridge/bridge-nf-call-iptables to  1*** by running below command to pass bridged IPv4 traffic to iptables chains. This is a requirement for some CNI plugins to work


```
root@ubuntu:~# modprobe br_netfilter
root@ubuntu:~# echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
root@ubuntu:~#
```

## Kubernet Installation

**Download the package list for upgrade**
```
root@ubuntu:~# sudo apt-get update
```

**Upgrade the installed packages to new version**
```
root@ubuntu:~# sudo apt-get upgrade
```
**Add the kubernetes package key for installation**
```
root@ubuntu:~# sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
OK
root@ubuntu:~#
```
**Now add the Kubernetes repository**
```
root@ubuntu:~# touch /etc/apt/sources.list.d/kubernetes.list
root@ubuntu:~# vim /etc/apt/sources.list.d/kubernetes.list
root@ubuntu:~# cat /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
root@ubuntu:~#
```
```
root@ubuntu:~# apt-get update
root@ubuntu:~#
```

**Now update the package list and install**
```
root@ubuntu:~# sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
Reading package lists… Done
- - - 
********** Some Output has skipped here **********
- - -
root@ubuntu:~#

````


**Configure the hosts file**
```
root@ubuntu:~# cat /etc/hosts
127.0.0.1	localhost
192.168.11.40	ubuntu
127.0.1.1	ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
root@ubuntu:~#
```
#### Note:
I may use **flannet network** to configure so. remember the pod-network here. For flannel to work correctly, you must pass **--pod-network-cidr=10.244.0.0/16** to kubeadm init

#### Initialize the node with below command.
```
root@ubuntu:~# kubeadm init  --apiserver-advertise-address=192.168.11.40 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU
- - - 
********** Some Output has skipped here **********
- - -
our Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config



You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.11.40:6443 --token phg7op.vgqquaaq3keg5x8a \
    --discovery-token-ca-cert-hash sha256:d78c6b0ba891d68cddc7a2030f16f524ec4e3f1289e8cae24b3acb2489568f6e
root@ubuntu:~# 

```
#### Note:
Remember the below commands fromt the **kubeadm init** output and execute it and also remember the **kubeadm join** command for future referenced to add or join the worker nodes to cluster. 
```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
```
root@ubuntu:~# mkdir -p $HOME/.kube
root@ubuntu:~# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@ubuntu:~# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Apply flannel to the cluster**
```
root@ubuntu:~# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
or
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
Ref : https://github.com/coreos/flannel/

**Note : Please wait for few seconds **


**Commands:**

```
root@ubuntu:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.2", GitCommit:"59603c6e503c87169aea6106f57b9f242f64df89", GitTreeState:"clean", BuildDate:"2020-01-18T23:30:10Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.2", GitCommit:"59603c6e503c87169aea6106f57b9f242f64df89", GitTreeState:"clean", BuildDate:"2020-01-18T23:22:30Z", GoVersion:"go1.13.5", Compiler:"gc", Platform:"linux/amd64"}
root@ubuntu:~#


root@ubuntu:~# kubectl get nodes
NAME     STATUS   ROLES    AGE    VERSION
ubuntu   Ready    master   104m   v1.17.2
root@ubuntu:~#

root@ubuntu:~# kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   104m
root@ubuntu:~#


root@ubuntu:~# kubectl cluster-info
Kubernetes master is running at https://192.168.11.40:6443
KubeDNS is running at https://192.168.11.40:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
root@ubuntu:~#
root@ubuntu:~#


root@ubuntu:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-88m6f         1/1     Running   0          5m44s
kube-system   coredns-6955765f44-dvj6h         1/1     Running   0          5m44s
kube-system   etcd-ubuntu                      1/1     Running   0          5m51s
kube-system   kube-apiserver-ubuntu            1/1     Running   0          5m51s
kube-system   kube-controller-manager-ubuntu   1/1     Running   3          5m51s
kube-system   kube-flannel-ds-amd64-hh5m5      1/1     Running   0          5m14s
kube-system   kube-proxy-wcgpl                 1/1     Running   0          5m44s
kube-system   kube-scheduler-ubuntu            1/1     Running   2          5m51s
root@ubuntu:~#
````
