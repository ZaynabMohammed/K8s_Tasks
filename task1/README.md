# Task 1 [Pods, ReplicaSet, Deployment]
1- Create a pod with the name redis and with the image redis.
```bash
$ kubectl apply -f pod_redis.yml
pod/nginx created
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          43s
```
2- Create a pod with the name nginx and with the image “nginx123” Use a pod-definition YAML file.

---> ImagePullBackOff, because there is no image with name “nginx123”
```bash
$ kubectl apply -f pod_nginx123.yml
pod/nginx created
$ kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
nginx   0/1     ImagePullBackOff   0          12s
redis   1/1     Running            0          10m
```
3- Change the nginx pod image to “nginx” check the status again

---> Now, Nginx Pod is running
```bash
$ kubectl apply -f pod_nginx.yml
pod/nginx created
$ kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
nginx   1/1     Running            0          8s
redis   1/1     Running            0          10m
```
4-  Create a ReplicaSet with name= replica-set-1, image= busybox, replicas= 3
- BusyBox is a minimalistic Linux environment, and by default, when you run a BusyBox container without specifying a command that keeps it running,  
   it will execute its default command and then immediately exit, causing the container to stop.
- So, I use `command: ["sleep", "3600"] ` , to keep the container running for 3600 seconds (1 hour).
```bash
$ kubectl apply -f replica_busybox.yml
replicaset.apps/replica-set-1 created
$ kubectl get replicaset
NAME            DESIRED   CURRENT   READY   AGE
replica-set-1   3         3         3       17s
$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
replica-set-1-26f5p   1/1     Running   0          11s
replica-set-1-hphpk   1/1     Running   0          11s
replica-set-1-sw9sm   1/1     Running   0          11s
```
5- Scale the ReplicaSet replica-set-1 to 5 PODs.
```bash
$ kubectl scale replicaset replica-set-1 --replicas 5
replicaset.apps/replica-set-1 scaled
$ kubectl get replicaset
NAME            DESIRED   CURRENT   READY   AGE
replica-set-1   5         5         5       6m10s
kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
replica-set-1-26f5p   1/1     Running   0          6m4s
replica-set-1-b82z9   1/1     Running   0          12s
replica-set-1-bmkcb   1/1     Running   0          12s
replica-set-1-hphpk   1/1     Running   0          6m4s
replica-set-1-sw9sm   1/1     Running   0          6m4s
```
6- Delete any one of the 5 PODs then check How many PODs exist now? 
   Why are there still 5 PODs, even after you deleted one?
   
   ---> one pod is terminating, but replicaset start a new pod to guarantee that 5 pods are running
   ```bash
$ kubectl delete pod replica-set-1-26f5p
pod "replica-set-1-26f5p" deleted
$ kubectl get pods
NAME                  READY   STATUS        RESTARTS   AGE
replica-set-1-26f5p   1/1     Terminating   0          9m22s
replica-set-1-2fbpr   1/1     Running       0          21s
replica-set-1-b82z9   1/1     Running       0          3m30s
replica-set-1-bmkcb   1/1     Running       0          3m30s
replica-set-1-hphpk   1/1     Running       0          9m22s
replica-set-1-sw9sm   1/1     Running       0          9m22s
  ```
7- Create a Deployment with name= deployment-1, image= busybox, replicas= 3
```bash
$ kubectl apply -f deploy_busybox.yml
deployment.apps/deployment-1 created
$ kubectl get replicaset
NAME                     DESIRED   CURRENT   READY   AGE
deployment-1-fdfd77b47   3         3         3       36s
$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
deployment-1-fdfd77b47-6lsdd   1/1     Running   0          20s
deployment-1-fdfd77b47-7mvk2   1/1     Running   0          20s
deployment-1-fdfd77b47-ql72k   1/1     Running   0          20s
$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1   3/3     3            3           2m4s
```
8- Update deployment-1 image to nginx then check the ready pods again

---> Deployment is configured and a new replicaset is now running to create new pods
```bash
$ kubectl apply -f deploy_busybox.yml
deployment.apps/deployment-1 configured
$ kubectl get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
deployment-1-566566bc89   3         2         2       7s
deployment-1-fdfd77b47    1         2         2       10m
$ kubectl get pods
NAME                            READY   STATUS        RESTARTS   AGE
deployment-1-566566bc89-8qbxc   1/1     Running       0          12s
deployment-1-566566bc89-mw5k4   1/1     Running       0          19s
deployment-1-566566bc89-xkbsh   1/1     Running       0          15s
deployment-1-fdfd77b47-6lsdd    1/1     Terminating   0          10m
deployment-1-fdfd77b47-7mvk2    1/1     Terminating   0          10m
deployment-1-fdfd77b47-ql72k    1/1     Terminating   0          10m

```
---> After all termination, New pods are running now with new replicaset
```bash
$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
deployment-1-566566bc89-8qbxc   1/1     Running   0          60s
deployment-1-566566bc89-mw5k4   1/1     Running   0          67s
deployment-1-566566bc89-xkbsh   1/1     Running   0          63s
$ kubectl get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
deployment-1-566566bc89   3         3         3       2m11s
deployment-1-fdfd77b47    0         0         0       12m
```
9- Run kubectl describe deployment deployment-1 and check events
 What is the deployment strategy used to upgrade the deployment-1?
 ```bash
$ kubectl describe deployment deployment-1
Name:                   deployment-1
StrategyType:           RollingUpdate
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  11m   deployment-controller  Scaled up replica set deployment-1-fdfd77b47 to 3
  Normal  ScalingReplicaSet  57s   deployment-controller  Scaled up replica set deployment-1-566566bc89 to 1
  Normal  ScalingReplicaSet  53s   deployment-controller  Scaled down replica set deployment-1-fdfd77b47 to 2 from 3
  Normal  ScalingReplicaSet  53s   deployment-controller  Scaled up replica set deployment-1-566566bc89 to 2 from 1
  Normal  ScalingReplicaSet  50s   deployment-controller  Scaled down replica set deployment-1-fdfd77b47 to 1 from 2
  Normal  ScalingReplicaSet  50s   deployment-controller  Scaled up replica set deployment-1-566566bc89 to 3 from 2
  Normal  ScalingReplicaSet  47s   deployment-controller  Scaled down replica set deployment-1-fdfd77b47 to 0 from 1
```
10- Rollback the deployment-1, What is the used image with the deployment-1?
```bash
$ kubectl rollout undo deployment deployment-1
deployment.apps/deployment-1 rolled back
$ kubectl get pods
NAME                            READY   STATUS        RESTARTS   AGE
deployment-1-566566bc89-8qbxc   1/1     Terminating   0          14m
deployment-1-566566bc89-mw5k4   1/1     Terminating   0          14m
deployment-1-566566bc89-xkbsh   1/1     Terminating   0          14m
deployment-1-fdfd77b47-hsqmh    1/1     Running       0          14s
deployment-1-fdfd77b47-st7vp    1/1     Running       0          20s
deployment-1-fdfd77b47-vppv9    1/1     Running       0          17s
$ kubectl get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
deployment-1-566566bc89   0         0         0       14m
deployment-1-fdfd77b47    3         3         3       25m
```
---> After all termination, New pods are running now with old replicaset
$ kubectl get pods
```bash
NAME                           READY   STATUS    RESTARTS   AGE
deployment-1-fdfd77b47-hsqmh   1/1     Running   0          37s
deployment-1-fdfd77b47-st7vp   1/1     Running   0          43s
deployment-1-fdfd77b47-vppv9   1/1     Running   0          40s
```
---> Run kubectl describe deployment deployment-1,to check used image?!
```bash
$ kubectl describe deployment deployment-1
Pod Template:
  Labels:  app=busybox
  Containers:
   busybox-container:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sleep
      3600
```
11- Create a deployment using nginx:latest image and name it as nginx-deployment. App labels should be
app: nginx-app and type: front-end. The container should be named as nginx-container; also make sure replica counts are 3.
```bash
$ kubectl apply -f deploy_nginx.yml
deployment.apps/nginx-deployment created
$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           22s
$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-694f765b8d   3         3         3       30s
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-694f765b8d-4zj6q   1/1     Running   0          33s
nginx-deployment-694f765b8d-8wf9g   1/1     Running   0          33s
nginx-deployment-694f765b8d-q9xsz   1/1     Running   0          33s
```

