# KUBERNETES Installation and With Wordpress and MySQL
---
**Installing Kubernetes Cluster with 2 minions on CentOS to manage pods and services.**

**Installation:**

Download and Install Minimal CentOS 7 Operating System in Virtual Boxes.

*Links to Download:*  
http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso  
Download minimal ISO:   
http://centos.excellmedia.net/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso  

**Refferred links to install kubernetes:**  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/  
https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/  
https://www.linuxtechi.com/install-kubernetes-1-7-centos7-rhel7/  
https://www.vultr.com/docs/getting-started-with-kubernetes-on-centos-7  
https://foxutech.com/how-to-run-wordpress-on-kubernetes/  
https://www.youtube.com/watch?v=F-DQeymA0Oc  
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network  


Take 3 nodes:
```sh
IP address	HostName	
—————————————————————
192.168.11.20	k8s-master.   # Master Node
192.168.11.21	k8s-node1.     # Minion Node1
192.168.11.22	k8s-node2.     # Minion Node2
```

**Versions:**
OperatingSystem:
```sh
	# cat /etc/redhat-release
	CentOS Linux release 7.4.1708 (Core)
	# uname -r
	3.10.0-693.21.1.el7.x86_64
```
Docker:
```sh
	# docker -v
	Docker version 1.13.1, build 774336d/1.13.1
```
Kubernetes:
```sh
	[root@k8s-master files]# kubectl version
	Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", 		BuildDate:"2018-03-26T16:55:54Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
	Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", 		BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```


**Steps to installation:**
1. Install ISO on all VM’s.
2. Install basic packages:  
```sh
setup-tools, vim, telnet etc
```
3. Configure second Interface(Host Only Adapter) enp0s8 on all nodes.
```sh
	[root@k8s-master ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
	TYPE="Ethernet"
	BROWSER_ONLY="no"
	BOOTPROTO="static"
	IPADDR=192.168.11.20				## 192.168.11.21 for minion node1
	NETMASK=255.255.255.0			## 192.168.11.22 for minion node2
	NAME="enp0s8"
	DEVICE="enp0s8"
	ONBOOT="yes"
	ZONE=public
      
      Add Host entries on all nodes.
	[root@k8s-master ~]# cat /etc/hosts
	192.168.11.20	k8s-master
	192.168.11.21	k8s-node1
	192.168.11.22	k8s-node2
```
4. Disable selinux on all machines.
```sh
	setenforce 0
	vim /etc/sysconfig/selinux
	SELINUX=disabled
	getenforce 						## To check the selinux status.
```
5. Disable firewalld on all nodes:
```sh
	systemctl firewalld stop
	systemctl firewalld disable
	systemctl firewalld status
	
```
6. Configure kubernetes repo on all nodes.
```sh
[root@k8s-master files]# cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

```
7. Do yum update to get the latest version of kubeadm on all nodes. yum update all


**On Master Node:**  
1. Install Kubeadm and Docker on master node.
```sh
yum install kubeadm docker -y
```
##If you are not disabled firewalld service then enable the below ports on master.
```
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=2379-2380/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10251/tcp
firewall-cmd --permanent --add-port=10252/tcp
firewall-cmd --permanent --add-port=10255/tcp
firewall-cmd --reload
```
2. Load module br_netfilter  
```sh
	modprobe br_netfilter
	echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```
3. Add following line “--runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice” in 10-kubeadm.conf file to avoid errors.
```sh
[root@k8s-master ~]# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
	ExecStart=/usr/bin/kubelet --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice $KUBELET_KUBECONFIG_ARGS
	 $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS
	 $KUBELET_CGROUP_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS
```
4. Enable and start docker and kubelet services
```sh
service enable docker
service start docker
service enable kubelet
service start kubelet
```
5. Initializing kubernetes cluster and Initializing master
```sh
[root@k8s-master ~]# kubeadm init --apiserver-advertise-address=192.168.11.20 --pod-network-cidr=10.244.0.0/16
	[init] Using Kubernetes version: v1.10.0
	[init] Using Authorization modes: [Node RBAC]
	[preflight] Running pre-flight checks.
		[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
		[WARNING FileExisting-crictl]: crictl not found in system path
	Suggestion: go get github.com/kubernetes-incubator/cri-tools/cmd/crictl
	[certificates] Generated ca certificate and key.
	[certificates] Generated apiserver certificate and key.
	[certificates] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.15]
	[certificates] Generated apiserver-kubelet-client certificate and key.
	[certificates] Generated etcd/ca certificate and key.
	[certificates] Generated etcd/server certificate and key.
	[certificates] etcd/server serving cert is signed for DNS names [localhost] and IPs [127.0.0.1]
	[certificates] Generated etcd/peer certificate and key.
	[certificates] etcd/peer serving cert is signed for DNS names [k8s-master] and IPs [10.0.2.15]
	[certificates] Generated etcd/healthcheck-client certificate and key.
	[certificates] Generated apiserver-etcd-client certificate and key.
	[certificates] Generated sa key and public key.
	[certificates] Generated front-proxy-ca certificate and key.
	[certificates] Generated front-proxy-client certificate and key.
	[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
	[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
	[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
	[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
	[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
	[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
	[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
	[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
	[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
	[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
	[init] This might take a minute or longer if the control plane images have to be pulled.
	[apiclient] All control plane components are healthy after 94.514270 seconds
	[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
	[markmaster] Will mark node k8s-master as master by adding a label and a taint
	[markmaster] Master k8s-master tainted and labelled with key/value: node-role.kubernetes.io/master=""
	[bootstraptoken] Using token: 3u77u7.rnapcinvjw7fj5ic
	[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
	[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
	[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
	[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
	[addons] Applied essential addon: kube-dns
	[addons] Applied essential addon: kube-proxy

	Your Kubernetes master has initialized successfully!

	To start using your cluster, you need to run the following as a regular user:

	mkdir -p $HOME/.kube
  	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  	sudo chown $(id -u):$(id -g) $HOME/.kube/config

	You should now deploy a pod network to the cluster.
	Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  	https://kubernetes.io/docs/concepts/cluster-administration/addons/

	You can now join any number of machines by running the following on each node
	as root:

	 kubeadm join 192.168.11.20:6443 --token 3u77u7.rnapcinvjw7fj5ic --discovery-token-ca-cert-hash
	 sha256:85dea3c3aad54b0bf27c4a5446f89b79b8d6b74e500cc100290ee679f93744cd
```
6. To Make kubectl work run below commands:
```sh
	## For non-root user:
		mkdir -p $HOME/.kube
		sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
		sudo chown $(id -u):$(id -g) $HOME/.kube/config
	## For root user:
		export KUBECONFIG=/etc/kubernetes/admin.conf
```

7. Make a record of the kubeadm join command that kubeadm init outputs. You will need this in a moment.

8. **Installation of POD network.**  

We must install a pod network add-on so that our pods can communicate with each other. The network must be deployed before any applications. Also, kube-dns, an internal helper service, will not start up before a network is installed. 

kubeadm only supports Container Network Interface (CNI) based networks (and does not support kubenet). 
Ref:    https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network
```sh	
	kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml		
```

**On Minion/Work Nodes:**
1. Do yum update to get the latest version of kubeadm on all nodes. yum update all

2. Install Kubeadm and Docker on master node.
```sh
	yum install kubeadm docker -y
```
3. Add line “--runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice” in 10-kubeadm.conf file.
```sh	
	[root@k8s-master ~]# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
		ExecStart=/usr/bin/kubelet --runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice $KUBELET_KUBECONFIG_ARGS
		 $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS
		 $KUBELET_CGROUP_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS 
```
4. Install kubeadm and docker package on both nodes and Start and enable services
```sh
	systemctl enable docker
	systemctl start docker
	systemctl enable kubelet
	systemctl start kubelet
	systemctl daemon-reload
	systemctl restart kubelet.service
```
5. Now Join worker nodes to master node. To join worker nodes to Master node, a token is required. 
Whenever kubernetes master initialized , then in the output we get command and token. Copy that command and run on both nodes.
```sh
	kubeadm join 192.168.11.20:6443 --token mkm92g.jajnw07mz3du6jdy --discovery-token-ca-cert-hash sha256:5b8be32a74b88c69bcb851a28cbb4cd6dcac5c685312180303495fc367ec0db4
```
**Deployment on Master Node:**
	*Check below commands:*
```sh	
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS    ROLES     AGE       VERSION
k8s-master   Ready     master    6h        v1.10.0
k8s-node1    Ready     <none>    6h        v1.10.0
	
[root@k8s-master ~]# kubectl get pods --all-namespaces -n kube-system
NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE
default       wordpress-78bd55fc98-5jb6v           1/1       Running   0          3h
default       wordpress-mysql-bcc89f687-ps7mr      1/1       Running   0          5h
kube-system   etcd-k8s-master                      1/1       Running   0          6h
kube-system   kube-apiserver-k8s-master            1/1       Running   0          6h
kube-system   kube-controller-manager-k8s-master   1/1       Running   0          6h
kube-system   kube-dns-86f4d74b45-xwlq7            3/3       Running   0          6h
kube-system   kube-flannel-ds-45ztb                1/1       Running   0          6h
kube-system   kube-flannel-ds-7vk4f                1/1       Running   1          6h
kube-system   kube-proxy-4wk89                     1/1       Running   0          6h
kube-system   kube-proxy-kwh8b                     1/1       Running   0          6h
kube-system   kube-scheduler-k8s-master            1/1       Running   0          6h
[root@k8s-master ~]#
```


1. Create a persistent volume for mysql container.
```
kubectl create -f mysql-volume.yaml
```
```sh
[root@k8s-master files]# cat mysql-volume.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
2. Create a secret for MySQL Password. Secret is an object that stores a piece of sensitive data like a password or key. 
```sh	
[root@k8s-master files]# kubectl create secret generic mysql-pass --from-literal=password=<YOUR_PASSWORD>
```
```
[root@k8s-master files]# kubectl get secrets
NAME                  TYPE                                  DATA      AGE
mysql-pass            Opaque                                1         5h
```

3. Deploy MySQL
```
kubectl create -f mysql.yaml
```
The following manifest describes a single-instance MySQL Deployment. The MySQL container mounts the PersistentVolume at /var/lib/mysql. The MYSQL_ROOT_PASSWORD environment variable sets the database password from the Secret.

```sh
[root@k8s-master files]# cat mysql.yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```
4. create a persistent volume for Wordpress container/pod.
```
kubectl create -f vol_wp.yaml
```

```sh
[root@k8s-master files]# cat vol_wp.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: task-pv-vol-wp
  labels:
    type: local
spec:
  storageClassName: wp
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data3"
```
5. Deploy WordPress

The following manifest describes a single-instance WordPress Deployment and Service. It uses many of the same features like a PVC for persistent storage and a Secret for the password. But it also uses a different setting: type: NodePort. This setting exposes WordPress to traffic from outside of the cluster.
```	
kubectl create -f wordpress-deployment.yaml
```
```sh
[root@k8s-master files]# cat wordpress-deployment.yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
#  type: LoadBalancer
  type: NodePort
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    type: local
    app: wordpress
spec:
  storageClassName: wp
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.9.4-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
[root@k8s-master files]#
```
6. Accessing the Wordpress Application from Browser:

```sh
	[root@k8s-master files]# kubectl get services
	NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
	kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        7h
	wordpress         NodePort    10.101.252.179   <none>        80:31623/TCP   3h
	wordpress-mysql   ClusterIP   None             <none>        3306/TCP       6h
	[root@k8s-master files]#
```

From the Above output 31623 is Port and node1 IP address is 192.168.11.21.

**Open http://192.168.11.21:31623 in you browser**

7. To Access containers or pods we can you below command.
```sh
	[root@k8s-master files]# kubectl get pods
	NAME                              READY     STATUS    RESTARTS   AGE
	wordpress-78bd55fc98-5jb6v        1/1       Running   0          3h
	wordpress-mysql-bcc89f687-ps7mr   1/1       Running   0          6h
```
**Login to Kubernetes container:**
```sh
		kubectl exec -it wordpress-mysql-bcc89f687-ps7mr -- /bin/bash
```

*Some commands and their outputs:*
```sh
[root@k8s-master files]# kubectl get nodes
NAME         STATUS    ROLES     AGE       VERSION
k8s-master   Ready     master    7h        v1.10.0
k8s-node1    Ready     <none>    7h        v1.10.0
[root@k8s-master files]# kubectl get pods
NAME                              READY     STATUS    RESTARTS   AGE
wordpress-78bd55fc98-5jb6v        1/1       Running   0          3h
wordpress-mysql-bcc89f687-ps7mr   1/1       Running   0          6h
[root@k8s-master files]# kubectl get pods --all-namespaces
NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE
default       wordpress-78bd55fc98-5jb6v           1/1       Running   0          3h
default       wordpress-mysql-bcc89f687-ps7mr      1/1       Running   0          6h
kube-system   etcd-k8s-master                      1/1       Running   0          7h
kube-system   kube-apiserver-k8s-master            1/1       Running   0          7h
kube-system   kube-controller-manager-k8s-master   1/1       Running   0          7h
kube-system   kube-dns-86f4d74b45-xwlq7            3/3       Running   0          7h
kube-system   kube-flannel-ds-45ztb                1/1       Running   0          7h
kube-system   kube-flannel-ds-7vk4f                1/1       Running   1          7h
kube-system   kube-proxy-4wk89                     1/1       Running   0          7h
kube-system   kube-proxy-kwh8b                     1/1       Running   0          7h
kube-system   kube-scheduler-k8s-master            1/1       Running   0          7h


[root@k8s-master files]# kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                    STORAGECLASS   REASON    AGE
task-pv-vol-wp   3Gi        RWO            Retain           Bound     default/wp-pv-claim      wp                       3h
task-pv-volume   3Gi        RWO            Retain           Bound     default/mysql-pv-claim   manual                   6h


[root@k8s-master files]# kubectl get pvc
NAME             STATUS    VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound     task-pv-volume   3Gi        RWO            manual         6h
wp-pv-claim      Bound     task-pv-vol-wp   3Gi        RWO            wp             3h


[root@k8s-master files]# kubectl get services
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        7h
wordpress         NodePort    10.101.252.179   <none>        80:31623/TCP   3h
wordpress-mysql   ClusterIP   None             <none>        3306/TCP       6h


[root@k8s-master files]# kubectl get deployments
NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
wordpress         1         1         1            1           3h
wordpress-mysql   1         1         1            1           6h


[root@k8s-master files]# kubectl get all
NAME                              READY     STATUS    RESTARTS   AGE
wordpress-78bd55fc98-5jb6v        1/1       Running   0          3h
wordpress-mysql-bcc89f687-ps7mr   1/1       Running   0          6h

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        7h
wordpress         NodePort    10.101.252.179   <none>        80:31623/TCP   3h
wordpress-mysql   ClusterIP   None             <none>        3306/TCP       6h

NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
wordpress         1         1         1            1           3h
wordpress-mysql   1         1         1            1           6h

NAME                        DESIRED   CURRENT   READY     AGE
wordpress-78bd55fc98        1         1         1         3h
wordpress-mysql-bcc89f687   1         1         1         6h
```

