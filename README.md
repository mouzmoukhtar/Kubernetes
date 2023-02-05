# kubernetes Practical Labs
set of practical labs for kubernetes i wish useful for u  ♡ 

## agenda 

*  [Create Pods]()
*  [ReplicaSets]()
*  [Deployments]()

## Create Pods
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
* Create a pod with the name nginx and with the image “nginx-iti”
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

## ReplicaSets
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
