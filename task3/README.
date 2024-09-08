# [ DaemonSets, Services (ClusterIP, NodePort), StaticPods ]

1- How many DaemonSets are created in the cluster in all namespaces?
2- what DaemonSets exist on the kube-system namespace?
```bash
$ kubectl get daemonsets --all-namespaces
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kindnet      2         2         2       2            2           <none>                   22d
kube-system   kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   22d
```
3- What is the image used by the POD deployed by the kube-proxy DaemonSet.
```bash
$ kubectl describe daemonset kube-proxy --namespace=kube-system | grep -i image
    Image:      registry.k8s.io/kube-proxy:v1.30.0
```
4- Deploy a DaemonSet for FluentD Logging. Use the given specifications:
Name: elasticsearch
Namespace: kube-system
Image: k8s.gcr.io/fluentd-elasticsearch:1.20
```bash
$ kubectl apply -f daemonset.yml
daemonset.apps/elasticsearch created
$ kubectl get daemonsets -n kube-system
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
elasticsearch   2         2         0       2            0           <none>                   42s
kindnet         2         2         2       2            2           <none>                   22d
kube-proxy      2         2         2       2            2           kubernetes.io/os=linux   22d

```
5- Deploy a pod named nginx-pod using the nginx:alpine image with the labels set to tier=backend.
```bash
$ kubectl apply -f pod-nginx.yml
pod/nginx-pod unchanged
```
6- Deploy a test pod using the nginx:alpine image.
```bash
$ kubectl apply -f pod-test.yml
pod/test-pod created
```
7- Create a service backend-service to expose the backend application within the cluster on port 80.
```bash
$ kubectl apply -f service-pod-nginx.yml
service/backend-service created
```
8- try to curl the backend-service from the test pod. What is the response?
- Check the status of the Pods and Service in default namespace
```bash
$ kubectl get pods
NAME        READY   STATUS    RESTARTS        AGE
nginx-pod   1/1     Running   1 (4m49s ago)   19h
test-pod    1/1     Running   0               3m23s
$ kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
backend-service   ClusterIP   10.111.227.203   <none>        80/TCP    4m15s
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP   23d
```
- Get into the Test-Pod and Curl the Backend Service from the Test Pod, from below response it means that the `test-pod` was able to successfully  
   reach the `backend-service`, which is routing traffic to the `nginx-pod`.
```bash
$ kubectl exec -it test-pod -- /bin/sh
curl http://backend-service:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
9- Create a deployment named web-app using the image nginx with 2 replicas
```bash
$ kubectl apply -f deploy-nginx.yml
deployment.apps/web-app created
```
10- Expose the web-app as service web-app-service application on port 80 and nodeport 30082 on the nodes on the cluster
```bash
$ kubectl apply -f service-deploy-nginx.yml
service/web-app-service created
```
11- access the web app from the node
- Get the Node IP Address by running below command
```bash
$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                 CONTAINER-RUNTIME
minikube       Ready    control-plane   23d   v1.30.0   192.168.49.2   <none>        Ubuntu 22.04.4 LTS   5.14.0-427.28.1.el9_4.x86_64   docker://26.1.1
minikube-m02   Ready    <none>          23d   v1.30.0   192.168.49.3   <none>        Ubuntu 22.04.4 LTS   5.14.0-427.28.1.el9_4.x86_64   docker://26.1.1
```
- Access the Web App using `crul http://<NODE_IP>:30082`
```bash
$ curl http://192.168.49.2:30082
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
12- How many static pods exist in this cluster in all namespaces?
- First Get nodes in the cluster by running `docker ps` as minikube based on docker driver. 
```bash
$ docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED       STATUS          PORTS                                                                                                                                  NAMES
923e01ef0ab8   kicbase/stable:v0.0.44   "/usr/local/bin/entr…"   3 weeks ago   Up 36 minutes   127.0.0.1:32773->22/tcp, 127.0.0.1:32774->2376/tcp, 127.0.0.1:32775->5000/tcp, 127.0.0.1:32776->8443/tcp, 127.0.0.1:32777->32443/tcp   minikube-m02
12280cac6eed   kicbase/stable:v0.0.44   "/usr/local/bin/entr…"   3 weeks ago   Up 37 minutes   127.0.0.1:32768->22/tcp, 127.0.0.1:32769->2376/tcp, 127.0.0.1:32770->5000/tcp, 127.0.0.1:32771->8443/tcp, 127.0.0.1:32772->32443/tcp   minikube
```
- Second, Get into `minikube` node by running `docker exec -it minikube bash` and navigate to `/var/lib/kubelet` to check staticPodPath in config file,  
  as shown below there are 4 static pods in control-plane node.
```bash
$ docker exec -it minikube bash
root@minikube:/# cd /var/lib/kubelet/
root@minikube:/var/lib/kubelet# cat config.yaml | grep -i staticPodPath
staticPodPath: /etc/kubernetes/manifests
root@minikube:/var/lib/kubelet# cd /etc/kubernetes/manifests
root@minikube:/etc/kubernetes/manifests# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```
- Third, Get into `minikube-m02` node by running `docker exec -it minikube-m02 bash` and navigate to `/var/lib/kubelet` to check staticPodPath in config file,  
 as shown below there are NO static pods in this worker node.
```bash
$ docker exec -it minikube-m02 bash
root@minikube-m02:/# cd /var/lib/kubelet/
root@minikube-m02:/var/lib/kubelet# cat config.yaml | grep -i staticPodPath
staticPodPath: /etc/kubernetes/manifests
root@minikube-m02:/var/lib/kubelet# cd /etc/kubernetes/manifests
root@minikube-m02:/etc/kubernetes/manifests# ls
```
13- On which nodes are the static pods created currently?
- There are 4 static pods in control-plane node `minikube`.
