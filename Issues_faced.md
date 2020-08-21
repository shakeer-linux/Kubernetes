#                  ::Issues::


### Issue No 1: 

Issue: Not able to shedule the POD's on single node kubernetes cluster, got below error.


```
root@ubuntu:~# kubectl describe pod wordpress-mysql-5b697dbbfc-q46r5
Name:           wordpress-mysql-5b697dbbfc-q46r5
Namespace:      default
Priority:       0
Node:           <none>
Labels:         app=wordpress
                pod-template-hash=5b697dbbfc
                tier=mysql
Annotations:    <none>
Status:         Pending
IP:
IPs:            <none>
Controlled By:  ReplicaSet/wordpress-mysql-5b697dbbfc
Containers:
  mysql:
    Image:      mysql:5.6
    Port:       3306/TCP
    Host Port:  0/TCP
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-pass'>  Optional: false
    Mounts:
      /var/lib/mysql from mysql-persistent-storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-qld78 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  mysql-persistent-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mysql-pv-claim
    ReadOnly:   false
  default-token-qld78:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-qld78
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  9s (x7 over 5m14s)  default-scheduler  0/1 nodes are available: 1 node(s) had taints that the pod didn't tolerate.
root@ubuntu:~#
```

***Solution:***

***Control plane node isolation***

```
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#master-isolation
```

By default, your cluster will not schedule Pods on the control-plane node for security reasons. If you want to be able to schedule Pods on the control-plane node, e.g. for a single-machine Kubernetes cluster for development, 

run:
```kubectl taint nodes --all node-role.kubernetes.io/master-
```
With output looking something like:
```
node "test-01" untainted

or 

taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```




```
root@ubuntu:~# kubectl taint nodes --all node-role.kubernetes.io/master-
node/ubuntu untainted
root@ubuntu:~#
```





