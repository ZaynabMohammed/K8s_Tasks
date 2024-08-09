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
4-  create a ReplicaSet with name= replica-set-1, image= busybox, replicas= 3
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
7- create a Deployment with name= deployment-1, image= busybox, replicas= 3
```bash
$ kubectl apply -f deploy_busybox.yml
deployment.apps/deployment-1 created
$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1   3/3     3            3           2m4s
$ kubectl get replicaset
NAME                     DESIRED   CURRENT   READY   AGE
deployment-1-fdfd77b47   3         3         3       36s
$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
deployment-1-fdfd77b47-6lsdd   1/1     Running   0          20s
deployment-1-fdfd77b47-7mvk2   1/1     Running   0          20s
deployment-1-fdfd77b47-ql72k   1/1     Running   0          20s
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


