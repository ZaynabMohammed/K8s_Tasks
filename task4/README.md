# [ ConfigMaps, Secrets, Multi-container-pod, PV, PVC ]

1- How many ConfigMaps exist in the environment?
```bash
$ kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      24d
```
2- Create a new ConfigMap Use the spec given below.  
ConfigName Name: webapp-config-map  
Data: APP_COLOR=darkblue  
```bash
$ kubectl apply -f configmap.yml
configmap/webapp-config-map created
```
3- Create a webapp-color POD with nginx image and use the created ConfigMap
```bash
kubectl apply -f pod-nginx.yml
pod/webapp-color created
```
- Verify the Creation
```bash
$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
webapp-color   1/1     Running   0          57s
$ kubectl get configmaps
NAME                DATA   AGE
kube-root-ca.crt    1      24d
webapp-config-map   1      76s
```
- Verify the Environment Variable  
To confirm that the environment variable `APP_COLOR` is set the Pod, you can exec into the running Pod.
```bash
$ kubectl exec -it webapp-color -- /bin/bash
root@webapp-color:/# echo $APP_COLOR
darkblue
```
4- How many Secrets exist on the system?
```bash
$ kubectl get secrets --all-namespaces
NAMESPACE     NAME                     TYPE                            DATA   AGE
kube-system   bootstrap-token-or0gwv   bootstrap.kubernetes.io/token   5      24d
```
5- How many secrets are defined in the default-token secret?
```bash
- There are no default-token secrets as, The default-token secret in Kubernetes is automatically created
for each service account in a namespace, and it typically contains the following:    
1. `Token`: This is the service account token, which is used for authenticating to the Kubernetes API server.
2. `CA Certificate`: The certificate authority (CA) bundle used to verify the API server’s certificate.
3. `Namespace`: The namespace in which the pod is running.
```  
6- Create a POD called db-pod with the image mysql:5.7 then check the POD status
- Container `db-pod` in pod "db-pod" is waiting to start.
```bash
$ kubectl run db-pod --image=mysql:5.7
pod/db-pod created
$ kubectl get pods
NAME     READY   STATUS              RESTARTS   AGE
db-pod   0/1     ContainerCreating   0          6s
$ kubectl logs db-pod
Error from server (BadRequest): container "db-pod" in pod "db-pod" is waiting to start: ContainerCreating
```
7- why the db-pod status not ready
- `CrashLoopBackoff`: It fails to start due to environment variables not being set correctly.
```bash
$ kubectl get pods
NAME     READY   STATUS             RESTARTS      AGE
db-pod   0/1     CrashLoopBackOff   3 (38s ago)   3m48s
$ kubectl get pods
NAME     READY   STATUS   RESTARTS      AGE
db-pod   0/1     Error    5 (99s ago)   5m40s
```
8- Create a new secret named db-secret with the data given below.
Secret Name: db-secret
Secret 1: MYSQL_DATABASE=sql01
Secret 2: MYSQL_USER=user1
Secret3: MYSQL_PASSWORD=password
Secret 4: MYSQL_ROOT_PASSWORD=password123  

- You can encode values based on `base64` encoding. using the following command in a Linux-based system:
```bash
$ echo -n 'sql01' | base64
c3FsMDE=
$ echo -n 'user1' | base64
dXNlcjE=
$ echo -n 'password' | base64
cGFzc3dvcmQ=
$ echo -n 'password123' | base64
cGFzc3dvcmQxMjM=
```
- Create the Secret
```bash
$ kubectl apply -f db-secret.yml
secret/db-secret created
```
9- Configure db-pod to load environment variables from the newly created secret.
- Create the db-pod
```bash
$ kubectl apply -f db-pod.yml
pod/db-pod created
```
- Check the Pod Status.
```bash
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
db-pod   1/1     Running   0          36s
$ kubectl get secrets
NAME        TYPE     DATA   AGE
db-secret   Opaque   4      108s
```
10- Create a multi-container pod with 2 containers.  
Name: yellow  
Container 1 Name: lemon  
Container 1 Image: busybox  
Container 2 Name: gold  
Container 2 Image: redis  
```bash
$ kubectl apply -f yellow-pod.yml
pod/yellow created
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
yellow   2/2     Running   0          7s
```
11- Create a pod red with redis image and use an initContainer that uses the busybox image and sleeps for 20 seconds.
```bash
$ kubectl get pods
NAME   READY   STATUS     RESTARTS   AGE
red    0/1     Init:0/1   0          36s
$ kubectl get pods
NAME   READY   STATUS            RESTARTS   AGE
red    0/1     PodInitializing   0          43s
$ kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
red    1/1     Running   0          52s
```
12- Create a pod named print-envars-greeting.
   1. Configure spec as, the container name should be print-env-container and use bash image.
   2. Create three environment variables:
       a. GREETING and its value should be “Welcome to”
       b. COMPANY and its value should be “DevOps”
       c. GROUP and its value should be “Industries”
  3. Use command to echo ["$(GREETING) $(COMPANY) $(GROUP)"] message.
  4. You can check the output using <kubctl logs -f [ pod-name ]> command.
