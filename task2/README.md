# Task2 [NameSpaces, Resources Requests & Limits]
1- How many Namespaces exist on the system?
```bash
$ kubectl get ns
NAME              STATUS   AGE
default           Active   3m49s
kube-node-lease   Active   3m49s
kube-public       Active   3m49s
kube-system       Active   3m49s
```
2- How many pods exist in the kube-system namespace?
```bash
$ kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS        AGE
coredns-7db6d8ff4d-tt8ff           1/1     Running   0               4m12s
etcd-minikube                      1/1     Running   0               4m28s
kube-apiserver-minikube            1/1     Running   0               4m26s
kube-controller-manager-minikube   1/1     Running   0               4m26s
kube-proxy-8bl2k                   1/1     Running   0               4m12s
kube-scheduler-minikube            1/1     Running   0               4m26s
storage-provisioner                1/1     Running   1 (3m40s ago)   4m23s
```
3- Create a deployment with 
- Name: beta
- Image: redis
- Replicas: 2
- Namespace: finance
- Resources Requests:  
  CPU: .5 vcpu  
  Mem: 1G  
- Resources Limits:  
  CPU: 1 vcpu  
  Mem: 2G  
```bash
$ kubectl create ns finance
namespace/finance created
$ kubectl apply -f depolyment.yml
deployment.apps/beta created
```
```bash
$ kubectl get pods -n finance
NAME                    READY   STATUS    RESTARTS   AGE
beta-7b6855d758-jc2st   1/1     Running   0          5m49s
beta-7b6855d758-wj6dw   1/1     Running   0          5m49s
```
```bash
$ kubectl get all -n finance
NAME                        READY   STATUS    RESTARTS   AGE
pod/beta-7b6855d758-jc2st   1/1     Running   0          21m
pod/beta-7b6855d758-wj6dw   1/1     Running   0          21m

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/beta   2/2     2            2           21m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/beta-7b6855d758   2         2         2       21m
```
4- How many Nodes exist on the system?
```bash
$ kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   3m26s   v1.30.0
```
