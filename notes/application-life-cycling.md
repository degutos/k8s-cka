# Application LifeCycle Management


## Rolling updates and rollback


### Creating a Deployment


```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl create deployment nginx --image nginx
deployment.apps/nginx created
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           14s


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get deployments,pods
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           26s

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-f89759699-x74jh   1/1     Running   0          26s


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-f89759699-x74jh   1/1     Running   0          44s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   238d

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           44s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-f89759699   1         1         1       44s
```  


```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get deployments,replicasets,pods
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           3m30s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-f89759699   1         1         1       3m30s

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-f89759699-x74jh   1/1     Running   0          3m30s
```



### Replicas on deployments


```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl scale deployment nginx --replicas=3
deployment.apps/nginx scaled


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get deployments,replicasets,pods
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           18m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-f89759699   3         3         3       18m

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-f89759699-tkz8f   1/1     Running   0          23s
pod/nginx-f89759699-wrn6m   1/1     Running   0          23s
pod/nginx-f89759699-x74jh   1/1     Running   0          18m
```



### Checking Deployment Strategy

```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl describe deployment nginx
Name:                   nginx
Namespace:              default
CreationTimestamp:      Wed, 03 Mar 2021 18:51:05 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-f89759699 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  22m    deployment-controller  Scaled up replica set nginx-f89759699 to 1
  Normal  ScalingReplicaSet  4m12s  deployment-controller  Scaled up replica set nginx-f89759699 to 3
  ```


* There are two Strategy Type when rollout:
- RollingUpdate (Default): application will be always available, kill one pod and spin another one
- Recreate: application won't be available for a short period, kill all the pod and then spin another new pod

```
  ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl describe deployment nginx | grep -i strategy
  StrategyType:           RollingUpdate
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
```

### Checking revisions/history of a Rollout

```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
```

### Checking and changing image

```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl describe pod nginx-f89759699-tkz8f | grep -i image
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:f3693fe50d5b1df1ecd315d54813a77afd56b0245a404055a946574deb6b34fc
  Normal  Pulling    19m   kubelet, andre-vsi-dal-ubuntu-worker2  Pulling image "nginx"
  Normal  Pulled     19m   kubelet, andre-vsi-dal-ubuntu-worker2  Successfully pulled image "nginx"


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl set image deployment nginx nginx=nginx:1.9.1
deployment.apps/nginx image updated


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get deployments,replicasets,pods
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           40m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6fbb48c46   3         3         3       47s
replicaset.apps/nginx-f89759699   0         0         0       40m

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-6fbb48c46-klm62   1/1     Running   0          26s
pod/nginx-6fbb48c46-n544c   1/1     Running   0          48s
pod/nginx-6fbb48c46-vnlhz   1/1     Running   0          37s
```

```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl describe pod nginx-6fbb48c46-klm62 | grep -i image
    Image:          nginx:1.9.1
    Image ID:       docker-pullable://nginx@sha256:2f68b99bc0d6d25d0c56876b924ec20418544ff28e1fb89a4c27679a40da811b
  Normal  Pulling    4m4s  kubelet, andre-vsi-dal-ubuntu-worker1  Pulling image "nginx:1.9.1"
  Normal  Pulled     4m3s  kubelet, andre-vsi-dal-ubuntu-worker1  Successfully pulled image "nginx:1.9.1"
```



root@controlplane:~# kubectl set image deployment frontend simple-webapp=kodekloud/webapp-color:v2                              
deployment.apps/frontend image updated
root@controlplane:~#




### Deployment rollout undo

Getting back to previous Version

```
ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get deployments,replicasets,pods

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           47m

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6fbb48c46   0         0         0       8m11s
replicaset.apps/nginx-f89759699   3         3         3       47m

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-f89759699-2wbvv   1/1     Running   0          24s
pod/nginx-f89759699-8g5kk   1/1     Running   0          27s
pod/nginx-f89759699-hl477   1/1     Running   0          30s


ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl describe pod nginx-f89759699-hl477 | grep -i image
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:f3693fe50d5b1df1ecd315d54813a77afd56b0245a404055a946574deb6b34fc
  Normal  Pulling    65s   kubelet, andre-vsi-dal-ubuntu-worker2  Pulling image "nginx"
  Normal  Pulled     64s   kubelet, andre-vsi-dal-ubuntu-worker2  Successfully pulled image "nginx"
  ```
