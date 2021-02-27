# DaemonSets


```
controlplane $ kubectl get daemonsets -A
NAMESPACE     NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-flannel-ds-amd64     2         2         2       2            2           <none>                   2m18s
kube-system   kube-flannel-ds-arm       0         0         0       0            0           <none>                   2m18s
kube-system   kube-flannel-ds-arm64     0         0         0       0            0           <none>                   2m18s
kube-system   kube-flannel-ds-ppc64le   0         0         0       0            0           <none>                   2m17s
kube-system   kube-flannel-ds-s390x     0         0         0       0            0           <none>                   2m17s
kube-system   kube-proxy                2         2         2       2            2           kubernetes.io/os=linux   2m20s
```



```
controlplane $ kubectl describe ds kube-flannel-ds-amd64 -n kube-system
Name:           kube-flannel-ds-amd64
Selector:       app=flannel
Node-Selector:  <none>
Labels:         app=flannel
                tier=node
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           app=flannel
                    tier=node
  Service Account:  flannel
  Init Containers:
   install-cni:
    Image:      quay.io/coreos/flannel:v0.12.0-amd64
    Port:       <none>
    Host Port:  <none>
    Command:
      cp
    Args:
      -f
      /etc/kube-flannel/cni-conf.json
      /etc/cni/net.d/10-flannel.conflist
    Environment:  <none>
    Mounts:
      /etc/cni/net.d from cni (rw)
      /etc/kube-flannel/ from flannel-cfg (rw)
  Containers:
   kube-flannel:
    Image:      quay.io/coreos/flannel:v0.12.0-amd64
    Port:       <none>
    Host Port:  <none>
    Command:
      /opt/bin/flanneld
    Args:
      --ip-masq
      --kube-subnet-mgr
    Limits:
      cpu:     100m
      memory:  50Mi
    Requests:
      cpu:     100m
      memory:  50Mi
    Environment:
      POD_NAME:        (v1:metadata.name)
      POD_NAMESPACE:   (v1:metadata.namespace)
    Mounts:
      /etc/kube-flannel/ from flannel-cfg (rw)
      /run/flannel from run (rw)
  Volumes:
   run:
    Type:          HostPath (bare host directory volume)
    Path:          /run/flannel
    HostPathType:  
   cni:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/cni/net.d
    HostPathType:  
   flannel-cfg:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-flannel-cfg
    Optional:  false
Events:
  Type    Reason            Age    From                  Message
  ----    ------            ----   ----                  -------
  Normal  SuccessfulCreate  9m15s  daemonset-controller  Created pod: kube-flannel-ds-amd64-hgl6h
  Normal  SuccessfulCreate  9m7s   daemonset-controller  Created pod: kube-flannel-ds-amd64-fwvlx
controlplane $ kubectl describe ds kube-flannel-ds-amd64 -n kube-system | grep -i image
    Image:      quay.io/coreos/flannel:v0.12.0-amd64
    Image:      quay.io/coreos/flannel:v0.12.0-amd64
```



```
controlplane $ kubectl create deploy elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system  --dry-run=client -o yaml | sed '/null\|{}\|replicas/d;/status/,$d;s/Deployment/DaemonSet/g' > daemonset.yaml
```

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: k8s.gcr.io/fluentd-elasticsearch:1.20
        name: fluentd-elasticsearch
```





```
controlplane $ kubectl apply -f daemonset.yaml  
daemonset.apps/elasticsearch created
```



```
controlplane $ kubectl get ds -A
NAMESPACE     NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   elasticsearch             1         1         1       1            1           <none>                   15s
kube-system   kube-flannel-ds-amd64     2         2         2       2            2           <none>                   48m
kube-system   kube-flannel-ds-arm       0         0         0       0            0           <none>                   48m
kube-system   kube-flannel-ds-arm64     0         0         0       0            0           <none>                   48m
kube-system   kube-flannel-ds-ppc64le   0         0         0       0            0           <none>                   48m
kube-system   kube-flannel-ds-s390x     0         0         0       0            0           <none>                   48m
kube-system   kube-proxy                2         2         2       2            2           kubernetes.io/os=linux   48m
```
