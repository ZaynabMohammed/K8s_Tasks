# [ ConfigMaps, Secrets, Multi-container, Init-container, PV, PVC ]

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

- No `default-token` secrets in this environment as, The `default-token` secret in Kubernetes is automatically created
  for each service account in a namespace, and it typically contains the following:
  1. **Token**: This is the service account token, which is used for authenticating to the Kubernetes API server.
  2. **CA Certificate**: The certificate authority (CA) bundle used to verify the API server’s certificate.
  3. **Namespace**: The namespace in which the pod is running.
     
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
- Check the Pod and Secret Status.
```bash
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
db-pod   1/1     Running   0          36s
$ kubectl get secrets
NAME        TYPE     DATA   AGE
db-secret   Opaque   4      108s
```
## A Quick comparsion between multi-container and init-container
| multi-container   | init-container  | 
|------------|------------|
| Containers run together and often collaborate as part of the same application or service, sharing the Pod's resources and network.| Temporary container and execute setup tasks before the main application starts, ensuring a controlled startup sequence.|   
| Use Cases: Running sidecar containers alongside the main application. | Use Cases: Waiting for external dependencies to be ready, Initializing configurations or environment settings.|

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
  3. Use command to `echo ["$(GREETING) $(COMPANY) $(GROUP)"] ` message.
  4. You can check the output using `kubctl logs [ pod-name ]` command.
```bash
$ kubectl apply -f print-envars-greeting.yml
pod/print-env-greeting created
$ kubectl get pods
NAME                 READY   STATUS              RESTARTS   AGE
print-env-greeting   0/1     ContainerCreating   0          5s
$ kubectl get pods
NAME                 READY   STATUS      RESTARTS   AGE
print-env-greeting   0/1     Completed   0          10s
$ kubectl logs print-env-greeting
Welcome to DevOps Industries
```
13- Where is the default kubeconfig file located in the current environment?  
- It located at path `$HOME/.kube/config` .
14- How many clusters are defined in the default kubeconfig file?
- One cluster `minikube-cluster`
15- What is the user configured in the current context?
- user: minikube && current-context: minikube
```bash
$ cat config | grep current-context
current-context: minikube
```
16- Create a Persistent Volume with the given specification.  
Volume Name: pv-log  
Storage: 100Mi  
Access Modes: ReadWriteMany  
Host Path: /pv/log  
```bash
$ kubectl apply -f pv.yml
persistentvolume/pv-log created
$ kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1                  <unset>                          24s
```
17- Create a Persistent Volume Claim with the given specification.  
Volume Name: claim-log-1  
Storage Request: 50Mi  
Access Modes: ReadWriteMany  
```bash
$ kubectl apply -f pvc.yml
persistentvolumeclaim/claim-log-1 created
$ kubectl get pvc
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Bound    pv-log   100Mi      RWX                           <unset>                 23s
```
18- Create a webapp pod to use the persistent volume claim as its storage.  
Name: webapp  
Image Name: nginx  
Volume: PersistentVolumeClaim=claim-log-1  
Volume Mount: /var/log/nginx  
```bash
$ kubectl apply -f pv-nginx.yml
pod/webapp created
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          17s
```
