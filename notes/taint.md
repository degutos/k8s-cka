# Taint  and Tolerant

##  Checking if any pod has taint

```
controlplane $ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
controlplane   Ready    master   5m20s   v1.19.0
node01         Ready    <none>   4m52s   v1.19.0


controlplane $ kubectl  describe  node  controlplane | grep  -i  taint
Taints:             node-role.kubernetes.io/master:NoSchedule

controlplane $ kubectl describe node node01  | grep -i taint
Taints:             <none>
```

## Running a shell script  in  one line  to  check if  any node is taint

```
controlplane $ for node in $(kubectl get nodes --no-headers |  awk '{ print $1 }' ) ;  do echo $node ;  kubectl describe node $node | grep -i  taint ;  echo   "==================" ; done
controlplane
Taints:             node-role.kubernetes.io/master:NoSchedule
==================
node01
Taints:             <none>
==================
controlplane $

```


## Tainting a Node node01 with key mortein and type NoSchedule

```
controlplane $ kubectl taint node node01 spray=mortein:NoSchedule
node/node01 tainted
```

## How to  untaint a taint from master node:

```
controlplane $ kubectl describe node controlplane | grep -i taint
Taints:             node-role.kubernetes.io/master:NoSchedule
controlplane $ kubectl taint  node controlplane node-role.kubernetes.io/master:NoSchedule-
node/controlplane untainted
```
