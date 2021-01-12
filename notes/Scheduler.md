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

With scheduler we will the pod running and assigned to a node just fine:

```
$  kubectl get pods
NAME READY STATUS RESTARTS AGE IP NODE
nginx 1/1 Running 0 9s 10.40.0.4 node02
```


Without scheduler we must for a Node on the yaml  file by adding this variable:

```
nodeName: node02
```

Lets see the full file:

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
pod-definition.yaml
nodeName: node02
```
