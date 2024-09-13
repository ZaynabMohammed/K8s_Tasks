# [ ]
**1- Create a namespace haproxy-controller-devops.**
```bash
$ kubectl create namespace haproxy-controller-devops
namespace/haproxy-controller-devops created
$ kubectl get ns | grep -i ^haproxy
haproxy-controller-devops   Active   4m36s
```
**2- Create a ServiceAccount haproxy-service-account-devops under the same namespace**
```bash
$ kubectl create serviceaccount haproxy-service-account-devops --namespace haproxy-controller-devops
serviceaccount/haproxy-service-account-devops created
$ kubectl get serviceaccount -n haproxy-controller-devops
NAME                             SECRETS   AGE
default                          0         77s
haproxy-service-account-devops   0         41s
```
**3- Create a `ClusterRole` which should be named as `haproxy-cluster-role-devops`, to grant**  
 - permissions: `"get", "list", "watch", "create", "patch", "update"`  
 - to: `"Configmaps",”secrets”,"endpoints","nodes","pods","services","namespaces","events","serviceaccounts"`. 
```bash
$ kubectl apply -f haproxy-cluster-role.yml
clusterrole.rbac.authorization.k8s.io/haproxy-cluster-role-devops created
$ kubectl get clusterroles | grep -i ^haproxy
haproxy-cluster-role-devops                                            2024-09-13T21:09:34Z
```
```bash
$ kubectl describe clusterrole haproxy-cluster-role-devops
Name:         haproxy-cluster-role-devops
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources        Non-Resource URLs  Resource Names  Verbs
  ---------        -----------------  --------------  -----
  configmaps       []                 []              [get list watch create patch update]
  endpoints        []                 []              [get list watch create patch update]
  events           []                 []              [get list watch create patch update]
  namespaces       []                 []              [get list watch create patch update]
  nodes            []                 []              [get list watch create patch update]
  pods             []                 []              [get list watch create patch update]
  secrets          []                 []              [get list watch create patch update]
  serviceaccounts  []                 []              [get list watch create patch update]
  services         []                 []              [get list watch create patch update]
```
**4- Create a ClusterRoleBinding which should be named as `haproxy-cluster-role-binding-devops` under the same namespace.**  

**roleRef:** 
1. apiGroup should be `rbac.authorization.k8s.io`.
2. kind should be ClusterRole, name should be `haproxy-cluster-role-devops`
   
**subjects:** 
1. kind should be `ServiceAccount`, name should be `haproxy-service-account-devops`
2. namespace should be `haproxy-controller-devops`.
3. apiGroup should be `""`
```bash
$ kubectl apply -f haproxy-cluster-role-binding.yml
clusterrolebinding.rbac.authorization.k8s.io/haproxy-cluster-role-binding-devops created
$ kubectl get clusterrolebindings | grep -i ^haproxy
haproxy-cluster-role-binding-devops                             ClusterRole/haproxy-cluster-role-devops  
```
```bash
$ kubectl describe clusterrolebindings haproxy-cluster-role-binding-devops
Name:         haproxy-cluster-role-binding-devops
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  haproxy-cluster-role-devops
Subjects:
  Kind            Name                            Namespace
  ----            ----                            ---------
  ServiceAccount  haproxy-service-account-devops  haproxy-controller-devops
```
**5- Create a backend deployment which should be named as `backend-deployment-devops` under the same namespace, labels `run` should be `ingress-default-backend`**  

**SPEC**  
1. `replica` should be `1`.
2. selector's matchLabels `run` should be `ingress-default-backend`.
3. Template's labels run under metadata should be `ingress-default-backend`.
4. The container should named as `backend-container-devops`.
5. use image `gcr.io/google_containers/defaultbackend:1.0`.
6. containerPort should be `8080`.
```bash
$ kubectl apply -f backend-deployment-devops.yml
deployment.apps/backend-deployment-devops created
$ kubectl get deployments -n haproxy-controller-devops
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
backend-deployment-devops   1/1     1            1           18s
```
6- **Create a `service` for backend which should be named as `service-backend-devops` under the same namespace, labels `run` should be `ingress-default-backend`.**  
Configure spec as selector's `run` should be `ingress-default-backend`.  

**PORTS:**  
1. name as `port-backend`, protocol should be `TCP`, port should be `8080` and targetPort should be `8080`.
```bash
$ kubectl apply -f service-backend-devops.yml
service/service-backend-devops created
$ kubectl get service -n haproxy-controller-devops
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service-backend-devops   ClusterIP   10.108.225.109   <none>        8080/TCP   14s
```
7- **Create a deployment for `frontend` which should be named `haproxy-ingress-devops` under the same namespace.** 
Configure spec as `replica` should be `1`, selector's matchLabels should be `haproxy-ingress`, template's labels `run` should be `haproxy-ingress` under metadata.  

**CONTAINER:**    
1. name should be `ingress-container-devops` under the same service account `haproxy-service-account-devops`
2. use image `haproxytech/kubernetes-ingress`
3. give `args` as `--default-backend-service=haproxy-controller-devops/service-backend-devops`
4. `resources requests` for `cpu` should be `500m` and for `memory` should be `50Mi`,
5. `livenessProbe` httpGet path should be `/healthz` its port should be `1024`.
   
**PORTS:**  
1. The `first port` name should be `http` and its containerPort should be `80`.    
2. Thde `second port` name should be `https` and its containerPort should be `443`.    
3. The `third port` name should be stat its `containerPort` should be `1024`.
   
**ENV:**  
1. The `first env` name should be `TZ` its value should be `Etc/UTC`,
2. The `second env` name should be `POD_NAME` and its  
valueFrom:  
  fieldRef:  
    fieldPath: should be metadata.name   
3. The `third env` name should be `POD_NAMESPACE` and its  
valueFrom:  
  fieldRef:  
    fieldPath: should be metadata.namespace
```bash
   $ kubectl apply -f haproxy-ingress-devops.yaml
deployment.apps/haproxy-ingress-devops created
$ kubectl get deployments -n haproxy-controller-devops
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
backend-deployment-devops   1/1     1            1           44m
haproxy-ingress-devops      1/1     1            1           72s
```
8- **Create a `service` for frontend which should be named as `ingress-service-devops` under bsame namespace,**  
labels `run` should be `haproxy-ingress`. Configure spec as selectors `run` should be `haproxy-ingress`, `type` should be `NodePort`.   

**PORTS:**
1. The first port name should be `http`,its port should be `80`, protocol should be `TCP`, targetPort should be `80` and nodePort `32456`.
2. The second port name should be `https`, its port should be `443`, protocol should be `TCP`, targetPort should be`443` and nodePort `32567`.
3. The third port name should be `stat`, its port should be `1024`, protocol should be `TCP`, targetPort should be `1024` and nodePort `32678`.
```bash
$ kubectl apply -f service-ingress-devops.yml
service/ingress-service-devops created
$ kubectl get service -n haproxy-controller-devops
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                     AGE
ingress-service-devops   NodePort    10.96.82.160     <none>        80:32456/TCP,443:32567/TCP,1024:32678/TCP   6s
service-backend-devops   ClusterIP   10.108.225.109   <none>        8080/TCP                                    43m
```
   
