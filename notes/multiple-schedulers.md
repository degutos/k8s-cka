# Multiple Scheduler


- Kube-scheduler-controlplane is the POD repossible for scheduler pods into Nodes.


```
controlplane $ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-f9fd979d6-g88k4                1/1     Running   0          34m
kube-system   coredns-f9fd979d6-s9pnh                1/1     Running   0          34m
kube-system   etcd-controlplane                      1/1     Running   0          34m
kube-system   kube-apiserver-controlplane            1/1     Running   0          34m
kube-system   kube-controller-manager-controlplane   1/1     Running   0          34m
kube-system   kube-flannel-ds-amd64-2mfcs            1/1     Running   2          34m
kube-system   kube-flannel-ds-amd64-jbglq            1/1     Running   0          34m
kube-system   kube-proxy-6t9qw                       1/1     Running   0          34m
kube-system   kube-proxy-j4bq8                       1/1     Running   2          34m
kube-system   kube-scheduler-controlplane            1/1     Running   0          34m
```



- Scheduler uses the image called `k8s.gcr.io/kube-scheduler:v1.19.0`

```
controlplane $ kubectl describe pod kube-scheduler-controlplane -n kube-system | grep -i image
    Image:         k8s.gcr.io/kube-scheduler:v1.19.0
    Image ID:      docker-pullable://k8s.gcr.io/kube-scheduler@sha256:529a1566960a5b3024f2c94128e1cbd882ca1804f222ec5de99b25567858ecb9
controlplane $
```

### How to create second schedule in the same master node

```

controlplane $ cd /etc/kubernetes/manifest/
controlplane $ cp kube-scheduler.yaml my-schedule.yaml
```

```
controlplane $ cat my-schedule.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: my-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --scheduler-name=my-scheduler
    - --port=10282
    - --secure-port=0
    image: k8s.gcr.io/kube-scheduler:v1.19.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}
```

* We basically change: component, name, leader-elect and Port
* Also add secure-port and scheduler-name
