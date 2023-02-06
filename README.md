# kubernetes Practical Labs
set of practical labs for kubernetes i wish useful for u  ♡ 

## agenda 

1.  [Create Pods](#1-Create-Pods)
1.  [ReplicaSets](#2-ReplicaSets)
1.  [Deployments](#3-Deployments)
1.  [Resources Limits](#4-Resources-Limits)
1.  [NameSpaces](#5-NameSpaces)
1.  [Services](#6-Services)
1.  [StartUp & Readiness &Liveness](#7-StartUp,Readiness-&-Liveness)
1.  [Static pods](#8-Static-pods)
1.  [DaemonSets](#9-DaemonSets)
1.  [Init containers](#10-Init-containers)
1.  [Multi containers](#11-Multi-containers)


## 1. Create Pods
* Create a pod with the name redis and with the image redis.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
```
output:
```
$ kubectl apply -f redis-pod.yaml
pod/redis created
$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          42s
$ kubectl describe pod redis
Name:             redis
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.58.2
Start Time:       Sun, 05 Feb 2023 14:49:06 +0200
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.11
IPs:
  IP:  172.17.0.11
Containers:
  redis:
    Container ID:   docker://08db3484672d1cf2d5d594e0532d5e906544b9c6b402cba6ba8d803a19d84f60
    Image:          redis
    Image ID:       docker-pullable://redis@sha256:f761e3e82ab08798413f9aa8de8b54b91f84d92fbdc56735f895d9ec80745b1f
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 05 Feb 2023 14:49:16 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-v6q6l (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-v6q6l:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m56s  default-scheduler  Successfully assigned default/redis to minikube
  Normal  Pulling    3m56s  kubelet            Pulling image "redis"
  Normal  Pulled     3m48s  kubelet            Successfully pulled image "redis" in 7.389261214s
  Normal  Created    3m48s  kubelet            Created container redis
  Normal  Started    3m47s  kubelet            Started container redis

```
* Create a pod with the name nginx-iti and with the image “nginx”
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    name: nginx-iti
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80
```
```
kubectl apply -f nginx-pod.yaml
kubectl get pod nginx-pod
kubectl describe pod nginx-pod
```
<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## 2. ReplicaSets
create a ReplicaSet with 
name= replica-set-1
image= busybox
replicas= 3
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replica
  labels:
    app: busyboxiti
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: backend
  template:
    metadata:
      labels:
        tier: backend
    spec:
      containers:
      - name: busyboxiti
        image: busybox
        command:
          - sleep
          - "3600"
```
output
```
$ kubectl apply -f replicaset-busybox.yaml 
replicaset.apps/busyboxiti-replica created
$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/busyboxiti-replica-98785   1/1     Running   0          13s
pod/busyboxiti-replica-cffbj   1/1     Running   0          13s
pod/busyboxiti-replica-llqmt   1/1     Running   0          13s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d5h

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/busyboxiti-replica   3         3         3       13s
```
scale pods to 7 pods
```
$ kubectl scale replicaset.apps/busyboxiti-replica --replicas=7
replicaset.apps/busyboxiti-replica scaled
$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/busyboxiti-replica-98785   1/1     Running   0          83s
pod/busyboxiti-replica-cffbj   1/1     Running   0          83s
pod/busyboxiti-replica-llqmt   1/1     Running   0          83s
pod/busyboxiti-replica-nd49x   1/1     Running   0          8s
pod/busyboxiti-replica-ngxds   1/1     Running   0          8s
pod/busyboxiti-replica-r6dz7   1/1     Running   0          8s
pod/busyboxiti-replica-rqxmk   1/1     Running   0          8s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d5h

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/busyboxiti-replica   7         7         7       83s
```
* impotant not if u delete one of replicas pods it will running agin automatically
<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## 3. Deployments
create a Deployment with
name= deployment-iti
image= nginx
replicas= 3
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment-iti
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-iti
        image: nginx
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080
```
output

```
$ kubectl apply -f deployment-nginx.yaml 
deployment.apps/nginx-deployment-iti created
$ kubectl get all
NAME                                        READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-iti-5db7769d55-5w9tq   1/1     Running   0          11s
pod/nginx-deployment-iti-5db7769d55-n95m9   1/1     Running   0          11s
pod/nginx-deployment-iti-5db7769d55-vmd6s   1/1     Running   0          11s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   3d5h

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment-iti   3/3     3            3           11s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-iti-5db7769d55   3         3         3       11s


```
* uopdate of deployment 
```
$ kubectl set image deployment.apps/nginx-deployment-iti nginx-iti=nginx:1.2.14
deployment.apps/nginx-deployment-iti image updated
$ kubectl rollout status deployment.apps/nginx-deployment-iti
Waiting for deployment "nginx-deployment-iti" rollout to finish: 1 out of 3 new replicas have been updated...
```
* rollout of deployment
```
$ kubectl rollout undo deployment.apps/nginx-deployment-iti
deployment.apps/nginx-deployment-iti rolled back
$ kubectl rollout status deployment.apps/nginx-deployment-iti
deployment "nginx-deployment-iti" successfully rolled out

```
* important not if u not write version of image it takes tha latest automatically 

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## 4. Resources Limits

* important note if tha application need of cpu exceed limits pod of cpu it will be throttle
* if tha application need of memory exceed limits pod of memory it will be terminate
in case cpu
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo
spec:
  containers:
    - name: cpu-demo-ctr
      image: vish/stress
      resources:
        limits:
          cpu: "1"
        requests:
          cpu: "0.5"
      args:
      - -cpus
      - "2"
```
output
```
$ kubectl apply -f cpu-limit.yaml 
pod/cpu-demo created
$ kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
cpu-demo   1/1     Running   0          21s
```
in case memory
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mem-demo
spec:
  containers:
    - name: mem-demo-ctr
      image: polinux/stress 
      resources:
        limits:
          memory: "100"
        requests:
          memory: "50"
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
```
output
```
$ kubectl apply -f mem-limit.yaml 
pod/mem-demo created
$ kubectl get pod
NAME       READY   STATUS              RESTARTS   AGE
mem-demo   0/1     ContainerCreating   0          5m49s

```
* limit range:Provides constraints to limit resource consumption per Containers or Pods in a namespace.
in case cpu
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: # this section defines default limits
      cpu: 500m
    defaultRequest: # this section defines default requests
      cpu: 500m
    max: # max and min define the limit range
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```
```
$kubectl create namespace default-mem-example
$kubectl apply -f memory-defaults.yaml --namespace=default-mem-example
```
in case memory
```yaml
apiVersion: v1

kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container

```
```
$kubectl create namespace default-cpu-example
$kubectl apply -f cpu-defaults.yaml --namespace=default-cpu-example
```
* ResourceQuota:you might decide to limit how many Deployments that can live in a single namespace.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: pods-high
spec:
    hard:
      requests.cpu: "5"
      requests.memory: 5Gi
      limits.cpu: "10"
      limits.memory: 10Gi
      pods: "5"

```
we deploy pods with replica 7 but 5 only will deploy
```
$ kubectl apply -f qouta-resource.yaml 
resourcequota/pods-high created
$ kubectl describe quota
Name:            pods-high
Namespace:       default
Resource         Used   Hard
--------         ----   ----
limits.cpu       2500m  10
limits.memory    640Mi  10Gi
pods             5      5
requests.cpu     2500m  5
requests.memory  640Mi  5Gi

```
<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## 5. Services
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: iti
  labels:
    name: iti
```
```
$ kubectl apply -f namespace.yaml 
namespace/iti created
$ kubectl create namespace pro
namespace/pro created
$ kubectl get namespace
NAME                        STATUS   AGE
default                     Active   36d
haproxy-controller-devops   Active   19d
iti                         Active   2m12s
kube-node-lease             Active   36d
kube-public                 Active   36d
kube-system                 Active   36d
kubernetes-dashboard        Active   3d12h
pro                         Active   15s

```
Create a deployment with
Name: beta
Image: redis
Replicas: 2
Namespace: finance
Resources Requests:
CPU: .5 vcpu
Mem: 1G
Resources Limits:
CPU: 1 vcpu
Mem: 2G

create finance namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: finance
  labels:
    name: finance

```
create beta deploy
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: beta-deploy
  Namespace: finance
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-iti
        image: redis
        resources:
          requests:
            memory: "0.5"
            cpu: 1Gi
          limits:
            memory: 2Gi
            cpu: "1"
            
```
```
$ kubectl apply -f finance-namespace.yaml 
namespace/finance configured
$ kubectl apply -f beta-deploy.yaml 
deployment.apps/beta-deploy created
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>


## 6. Services
* Deploy a pod named nginx-pod using the nginx:alpine image with
the labels set to tier=backend.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      tier: backend
  template:
    metadata:
      labels:
        app: nginx
        tier: backend
    spec:
      containers:
      - name: nginx-backend
        image: nginx:alpine
        ports:
        - containerPort: 80


```
```
$ kubectl apply -f deploy-nginx-backend.yaml 
deployment.apps/nginx-backend created
```
Deploy a test pod using the nginx:alpine image
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      tier: backend
  template:
    metadata:
      labels:
        app: nginx
        tier: backend
    spec:
      containers:
      - name: nginx-test
        image: nginx:alpine
        ports:
        - containerPort: 80

```
```
$ kubectl apply -f deploy-test-pod.yaml 
deployment.apps/nginx-test created

```
Create a service backend-service to expose the backend
application within the cluster on port 80.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service-np
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  selector:
      app: nginx
      tier: backend

---

apiVersion: v1
kind: Service
metadata:
  name: backend-service-ip
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: nginx
    tier: backend
    
```
```
$ kubectl apply -f backend-service.yaml 
service/backend-service-np created
service/backend-service-ip created
```

try to curl the backend-service from the test pod. What is the response?
```
$ kubectl get po -o wide
NAME                             READY   STATUS    RESTARTS      AGE    IP            NODE       NOMINATED NODE   READINESS GATES
nginx-backend-64d54d7f54-btswf   1/1     Running   1 (39m ago)   117m   172.17.0.14   minikube   <none>           <none>
nginx-test-9f8f95f8b-bc2zk       1/1     Running   1 (39m ago)   131m   172.17.0.15   minikube   <none>           <non
$ kubectl exec -it nginx-test-9f8f95f8b-bc2zk -- /bin/sh
/ # curl 172.17.0.14 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
```
Create a deployment named web-app using the image nginx with 2 replicas
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-web-app
        image: nginx
        ports:
        - containerPort: 80
        
```

```
$ kubectl expose deployment nginx-web-app --name=web-app-svc --type=NodePort --port=80 --target-port=80
service/web-app-svc exposed
$ kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP        4m18s
web-app-svc   NodePort    10.108.129.154   <none>        80:32355/TCP   12s
 minikube service web-app-svc
|-----------|-------------|-------------|---------------------------|
| NAMESPACE |    NAME     | TARGET PORT |            URL            |
|-----------|-------------|-------------|---------------------------|
| default   | web-app-svc |          80 | http://192.168.58.2:32355 |
|-----------|-------------|-------------|---------------------------|
🎉  Opening service default/web-app-svc in default browser...
$ kubectl port-forward service/web-app-svc 9090:80
Forwarding from 127.0.0.1:9090 -> 80
Forwarding from [::1]:9090 -> 80
Handling connection for 9090
Handling connection for 9090


```
Expose the web-app as service web-app-service application on
port 80 and nodeport 30082 on the nodes on the cluster
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service-np
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30082
  selector:
      app: nginx
    
```
```
$ kubectl apply -f web-app-service.yaml 
service/web-app-service-np created
$ kubectl get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP        20m
web-app-service-np   NodePort    10.104.158.92   <none>        80:30082/TCP   62s
$ minikube service web-app-service-np
|-----------|--------------------|-------------|---------------------------|
| NAMESPACE |        NAME        | TARGET PORT |            URL            |
|-----------|--------------------|-------------|---------------------------|
| default   | web-app-service-np |          80 | http://192.168.58.2:30082 |
|-----------|--------------------|-------------|---------------------------|
🎉  Opening service default/web-app-service-np in default browser...
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>


## 7. StartUp & Readiness &Liveness

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

```

```
$ kubectl apply -f pods/probe/exec-liveness.yaml 
pod/liveness-exec created
$ kubectl describe pod liveness-exec
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  40s   default-scheduler  Successfully assigned default/liveness-exec to minikube
  Normal  Pulling    40s   kubelet            Pulling image "registry.k8s.io/busybox"
  Normal  Pulled     31s   kubelet            Successfully pulled image "registry.k8s.io/busybox" in 8.315409155s
  Normal  Created    31s   kubelet            Created container liveness
  Normal  Started    31s   kubelet            Started container liveness
$ kubectl describe pod liveness-exec
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  4m32s                 default-scheduler  Successfully assigned default/liveness-exec to minikube
  Normal   Pulled     4m23s                 kubelet            Successfully pulled image "registry.k8s.io/busybox" in 8.315409155s
  Normal   Pulled     3m10s                 kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.544904093s
  Normal   Created    112s (x3 over 4m23s)  kubelet            Created container liveness
  Normal   Started    112s (x3 over 4m23s)  kubelet            Started container liveness
  Normal   Pulled     112s                  kubelet            Successfully pulled image "registry.k8s.io/busybox" in 5.194440263s
  Warning  Unhealthy  67s (x9 over 3m52s)   kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    67s (x3 over 3m42s)   kubelet            Container liveness failed liveness probe, will be restarted
  Normal   Pulling    37s (x4 over 4m32s)   kubelet            Pulling image "registry.k8s.io/busybox"

```

[The rest of the examples](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## 8. Static pods
[/etc/kubernetes/manifests] this location can u create static pod when scheduler not working
can u get from this cmd
```
$ kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS        AGE
coredns-565d847f94-d7jkk           1/1     Running   22 (158m ago)   37d
etcd-minikube                      1/1     Running   22 (158m ago)   37d
fluentd-elasticsearch-6sqvc        1/1     Running   17 (158m ago)   27d
kube-apiserver-minikube            1/1     Running   22 (158m ago)   37d
kube-controller-manager-minikube   1/1     Running   22 (158m ago)   37d
kube-proxy-zcd6x                   1/1     Running   22 (158m ago)   37d
kube-scheduler-minikube            1/1     Running   22 (158m ago)   37d
storage-provisioner                1/1     Running   41 (157m ago)   37d

```
<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

