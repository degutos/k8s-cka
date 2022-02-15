# Scheduler

### Scheduler are available on kube-system


```
$  kubectl get pods --namespace=kube-system
NAME READY STATUS RESTARTS AGE
coredns-78fcdf6894-bk4ml 1/1 Running 0 1h
coredns-78fcdf6894-ppr6m 1/1 Running 0 1h
etcd-master 1/1 Running 0 1h
kube-apiserver-master 1/1 Running 0 1h
kube-controller-manager-master 1/1 Running 0 1h
kube-proxy-dgbgv 1/1 Running 0 1h
kube-proxy-fptbr 1/1 Running 0 1h
kube-scheduler-master 1/1 Running 0 1h
my-custom-scheduler 1/1 Running 0 9s
weave-net-4tfpt 2/2 Running 1 1h
weave-net-6j6zs 2/2 Running 1 1h
```

PS without  scheduler service we can not provision a  pod on  a Node and the pod  will have its state as Pending


```
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
nginx 0/1 Pending 0 3s
```


When we see pod in `Pending` state check if the `Kube-scheduler-master` is running

```
root@controlplane:~# kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-bdtx8                1/1     Running   0          7m43s
coredns-74ff55c5b-mh6fd                1/1     Running   0          7m43s
etcd-controlplane                      1/1     Running   0          7m51s
kube-apiserver-controlplane            1/1     Running   0          7m51s
kube-controller-manager-controlplane   1/1     Running   0          7m51s
kube-flannel-ds-dczmk                  1/1     Running   0          7m15s
kube-flannel-ds-v4vfx                  1/1     Running   0          7m44s
kube-proxy-57q72                       1/1     Running   0          7m15s
kube-proxy-k6vkj                       1/1     Running   0          7m43s
root@controlplane:~#
```

When we describe a pod in `Pending` state we won't see the node information set when we describe the pod

```
root@controlplane:~# kubectl describe po nginx
Name:         nginx
Namespace:    default
Priority:     0
Node:         <none>
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:           
IPs:          <none>
Containers:
  nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-558bj (ro)
Volumes:
  default-token-558bj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-558bj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:          <none>
```

Realize that the Node is set to Node 

```
Node:         <none>
```



With scheduler we will the see pod running and assigned to a node just fine:

```
$  kubectl get pods
NAME READY STATUS RESTARTS AGE IP NODE
nginx 1/1 Running 0 9s 10.40.0.4 node02
```


Without scheduler we must manually set for a Node on the yaml  file by adding this variable:

```
nodeName: node02
```

Lets see the full file:

pod-definition.yaml

```
apiVersion: v1
kind: Pod
metadata:
name: nginx
labels:
name: nginx
spec:
containers:
- name: nginx
image: nginx
ports:
- containerPort: 8080
nodeName: node02
```


Once we set `nodeName` variable in to the yaml file we can delete the pod and create it again. We will see the status running

```
root@controlplane:~# kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2m9s
root@controlplane:~# 
```

We can also describe the pod, pay attention to Node variable now

```
root@controlplane:~# kubectl describe po nginx 
Name:         nginx
Namespace:    default
Priority:     0
Node:         node01/10.56.156.9
Start Time:   Tue, 15 Feb 2022 20:50:40 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.2
IPs:
  IP:  10.244.1.2
Containers:
  nginx:
    Container ID:   docker://9e67dfd77359f9d085ee3b73bc80880456ea75450a1a03b5ac89ef03c36f405c
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 15 Feb 2022 20:50:53 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-558bj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-558bj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-558bj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  25s   kubelet  Pulling image "nginx"
  Normal  Pulled   15s   kubelet  Successfully pulled image "nginx" in 10.148979549s
  Normal  Created  14s   kubelet  Created container nginx
  Normal  Started  14s   kubelet  Started container nginx
root@controlplane:~# 
```




