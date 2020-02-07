
## Kubernetes Dashboard Installation


**Download the kubernetes dashobard yaml file.**

```
shakeer:~ shakeerp$ curl -LO http://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

**Add the line “type: NodePort” in below two services. This line has not added in recommended.yaml**

```
###########################
 30 ---
 31
 32 kind: Service
 33 apiVersion: v1
 34 metadata:
 35   labels:
 36     k8s-app: kubernetes-dashboard
 37   name: kubernetes-dashboard
 38   namespace: kubernetes-dashboard
 39 spec:
 40   ports:
 41     - port: 443
 42       targetPort: 8443
 43   selector:
 44     k8s-app: kubernetes-dashboard
 45   type: NodePort
########################### 
235 ---
236
237 kind: Service
238 apiVersion: v1
239 metadata:
240   labels:
241     k8s-app: dashboard-metrics-scraper
242   name: dashboard-metrics-scraper
243   namespace: kubernetes-dashboard
244 spec:
245   ports:
246     - port: 8000
247       targetPort: 8000
248   selector:
249     k8s-app: dashboard-metrics-scraper
250   type: NodePort
###########################
```
Note: Check the difference between edited and original yaml files.

```
shakeer:dashobard shakeerp$ diff recommended.yaml recommended.yaml_bkp
44a45
>   type: NodePort
248a250
>   type: NodePort
shakeer:dashobard shakeerp$
```

**Now, run/excute the dashboard deployment yaml file.**

```
shakeer:~ shakeerp$ kubectl apply -f recommended.yaml
```
Get the information of ports to access the dashboard.

```
shakeer:~ shakeerp$ kubectl get svc -n kubernetes-dashboard
NAME                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
dashboard-metrics-scraper   NodePort   10.96.73.64     <none>        8000:31062/TCP   28m
kubernetes-dashboard        NodePort   10.96.221.104   <none>        443:31429/TCP    28m
shakeer:~ shakeerp$
```

```
shakeer:~ shakeerp$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
shakeer:~ shakeerp$
```

**Open the link in Browser ::  https://192.168.99.100:31429**

Here, it will ask for the token. So, by running below process you will generate a token. copy the token on browser.

```
shakeer:~ shakeerp$ kubectl create serviceaccount dashboard -n default
serviceaccount/dashboard created

shakeer:~ shakeerp$ kubectl create clusterrolebinding dashboard-admin -n default --clusterrole=cluster-admin --serviceaccount=default:dashboard
clusterrolebinding.rbac.authorization.k8s.io/dashboard-admin created

shakeer:~ shakeerp$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode

eyJhbGciOiJSUzI1NiIsImtpZCI6InU0RmptcHk0TzFiRk5SdUJ0UzhKcHB4SEg3Q3RzYXNOeC1OUVFHS2xtcVkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC10b2tlbi1idnB0dyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4YTBiMDRlNS1jMTE3LTQ1ZDItYWNkMC05MmI3MjAxMWIyZDgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQifQ.FuWYZ4O5heoD3E75NbAOZz-yP3VUHXkjkdwp5-syvoQmjFH6YCwW9laciAfSrgwCXZYPQoWkmSlGnFe4K1NZVvyWeMbhE8Bins-EpeJ5kOpRkOPQ2dwhLty3J9esTwO5qAcidJhtRD5VWxKebEyA5VvL3g-E7dkjB1vt1fn3tIiOKe-5m1ntjBOWVnTljFrIKGRnS7DcLswscIwAEAuM6aFVUobMCB6pcwwNEaQRV8lhAVQztEhQZe5Z7xrppLu6JmMoAGvXLSOTPaJjHIWgyu9-eGwsMtWslOVdTC77bQ3y6oJ5oYjBc54hmyh-4JnRk-1sC234QMLh4EdCl3R67w

shakeer:~ shakeerp$
```
**Copy the Above token into the browser. Then you can access the Kubernetes dashboard.**
