**Introduction**
 
Kubernetes is a system designed to manage containerized applications built within Docker containers in a clustered environments. It provides basic mechanisms for deployment, maintenance and scaling of applications on public, private or hybrid setups, means, it handles the entire life cycle of a containerized application. It also have intelligence of self-healing features where containers can be auto provisioned, restarted or even replicated.

**Goals** 

* Understand the utility of Kubernetes
* Learn how to Setup Kubernetes Cluster
Prerequisites
* Understanding of Kubernetes Componetes.
* The nodes must installed docker and bridge-utils to manipulate linux bridge.
* All machines should communicate with each other and Master node needs to be connected to the Internet as it download the necessary files

**Kubernetes Components** 

Kubernetes works in server-client concept, where, it has a master that provide centralized control for a all minions(agent). We will be deploying one Kubernetes master with one minion and we will also have a workspace machine from where we will run all installation scripts.

* **kubeadm:** the command to bootstrap the cluster.
* **kubelet:** the component that runs on all of the machines in your cluster and does things like starting pods and containers.
* **kubectl:** the command line util to talk to your cluster.

Kubernetes has several components:

**Master Components:**

* ***etcd*** – A highly available key-value store for shared configuration and service discovery.
* ***flannel*** – An etcd backed network fabric for containers.***
* ***kube-apiserver*** – Provides the API for Kubernetes orchestration.
* ***kube-controller-manager*** – Enforces Kubernetes services.
* ***kube-scheduler*** – Schedules containers on hosts.

**Minion Components:**

* ***Docker*** - 	a daemon that runs application containers defined in pods.
* ***kubelet*** – Processes a container manifest so the containers are launched according to how they are described.
* ***kube-proxy*** – Provides network proxy services.
