
#### To keep a container/pod running on Kubernetes cluster, 
it should to be perform certain task, otherwise Kubernetes will find it unnecessary, therefore it stops.

Ref: https://stackoverflow.com/questions/31870222/how-can-i-keep-a-container-running-on-kubernetes

```
root@ubuntu16Desktop:~/ubuntu18# cat ubuntu18.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ubuntu18
  name: ubuntu18
  namespace : monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ubuntu18
  template:
    metadata:
      labels:
        app: ubuntu18
    spec:
      containers:
      - image: ubuntu:18.04
        name: ubuntu18
        command: [ "/bin/bash", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]

root@ubuntu16Desktop:~/ubuntu18# kubectl create -f ubuntu18.yaml
deployment.apps/ubuntu18 created
root@ubuntu16Desktop:~/ubuntu18#

root@ubuntu16Desktop:~/ubuntu18# kubectl get pods -n monitoring
NAME                                     READY   STATUS    RESTARTS   AGE
ubuntu18-569c9b5c-pwb96                  1/1     Running   0          5m13s
root@ubuntu16Desktop:~/ubuntu18#

```

Get a shell to the running container:

```
root@ubuntu16Desktop:~/ubuntu18# kubectl exec --stdin --tty ubuntu18-569c9b5c-pwb96 -n monitoring  -- /bin/bash
root@ubuntu18-569c9b5c-pwb96:/#
```


