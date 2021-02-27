# Logging and Monitoring


### Inspecting pods existents

```
controlplane $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          32s
lion       1/1     Running   0          33s
rabbit     1/1     Running   0          33s
```

### Cloning metrics-server from github

```
controlplane $ git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
Cloning into 'kubernetes-metrics-server'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 24 (delta 4), reused 0 (delta 0), pack-reused 12
Unpacking objects: 100% (24/24), done.
```


### Deploying metrics-server

```
controlplane $ ls
go  kubernetes-metrics-server
controlplane $ cd kubernetes-metrics-server/
controlplane $ ls
aggregated-metrics-reader.yaml  auth-reader.yaml         metrics-server-deployment.yaml  README.md
auth-delegator.yaml             metrics-apiservice.yaml  metrics-server-service.yaml     resource-reader.yaml

controlplane $ kubectl create -f .
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created

controlplane $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          110s
lion       1/1     Running   0          111s
rabbit     1/1     Running   0          111s

controlplane $ kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-5dk72                1/1     Running   0          7m8s
coredns-f9fd979d6-z87mq                1/1     Running   1          7m9s
etcd-controlplane                      1/1     Running   0          7m16s
kube-apiserver-controlplane            1/1     Running   0          7m16s
kube-controller-manager-controlplane   1/1     Running   1          7m16s
kube-flannel-ds-amd64-kln9b            1/1     Running   0          7m8s
kube-flannel-ds-amd64-rs8fn            1/1     Running   1          6m59s
kube-proxy-5jp8r                       1/1     Running   0          7m8s
kube-proxy-vdvtw                       1/1     Running   2          6m59s
kube-scheduler-controlplane            1/1     Running   1          7m16s
metrics-server-774b56d589-hp2g4        1/1     Running   0          21s
```

### Inspecting Node and Pod consumption

```
controlplane $ kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   140m         7%     1010Mi          53%       
node01         2000m        100%   628Mi           16%       

controlplane $ kubectl top pod
NAME       CPU(cores)   MEMORY(bytes)   
elephant   12m          30Mi            
lion       894m         1Mi             
rabbit     970m         1Mi       
```


## Inspecting application logs

### Checking existents Pods

```
controlplane $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          26s
```


### Checking logs on pod webapp about USER5

```
controlplane $ kubectl logs webapp-1 | grep USER5
[2021-02-27 14:42:59,828] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:04,836] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:09,845] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:14,853] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:19,861] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:24,868] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:29,876] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:34,882] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:39,890] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:44,897] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:49,905] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:54,914] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:43:59,922] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
```

### Checking logs for pod webapp container simple-webapp

```
controlplane $ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          99s
webapp-2   2/2     Running   0          12s


controlplane $ kubectl logs webapp-2 -c
db             simple-webapp  
controlplane $ kubectl logs webapp-2 -c simple-webapp
[2021-02-27 14:44:18,064] INFO in event-simulator: USER2 is viewing page3
[2021-02-27 14:44:19,065] INFO in event-simulator: USER4 is viewing page3
[2021-02-27 14:44:20,067] INFO in event-simulator: USER4 is viewing page3
[2021-02-27 14:44:21,069] INFO in event-simulator: USER2 is viewing page1
[2021-02-27 14:44:22,070] INFO in event-simulator: USER2 logged in
[2021-02-27 14:44:23,072] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:23,072] INFO in event-simulator: USER4 logged in
[2021-02-27 14:44:24,074] INFO in event-simulator: USER2 logged out
[2021-02-27 14:44:25,075] INFO in event-simulator: USER4 is viewing page2
[2021-02-27 14:44:26,077] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2021-02-27 14:44:26,077] INFO in event-simulator: USER3 logged in
[2021-02-27 14:44:27,078] INFO in event-simulator: USER1 is viewing page2
[2021-02-27 14:44:28,080] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:28,080] INFO in event-simulator: USER3 is viewing page1
[2021-02-27 14:44:29,081] INFO in event-simulator: USER3 logged out
[2021-02-27 14:44:30,083] INFO in event-simulator: USER4 logged out
[2021-02-27 14:44:31,084] INFO in event-simulator: USER4 is viewing page3
[2021-02-27 14:44:32,086] INFO in event-simulator: USER1 logged out
[2021-02-27 14:44:33,087] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:33,088] INFO in event-simulator: USER1 logged in
[2021-02-27 14:44:34,089] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2021-02-27 14:44:34,090] INFO in event-simulator: USER3 logged out
[2021-02-27 14:44:35,091] INFO in event-simulator: USER3 logged out
[2021-02-27 14:44:36,093] INFO in event-simulator: USER4 logged in
[2021-02-27 14:44:37,094] INFO in event-simulator: USER1 is viewing page3
[2021-02-27 14:44:38,096] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:38,096] INFO in event-simulator: USER4 logged in
[2021-02-27 14:44:39,097] INFO in event-simulator: USER2 is viewing page1
[2021-02-27 14:44:40,099] INFO in event-simulator: USER2 logged out
[2021-02-27 14:44:41,101] INFO in event-simulator: USER3 is viewing page2
[2021-02-27 14:44:42,103] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2021-02-27 14:44:42,103] INFO in event-simulator: USER3 logged in
[2021-02-27 14:44:43,104] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:43,105] INFO in event-simulator: USER2 is viewing page2
[2021-02-27 14:44:44,106] INFO in event-simulator: USER4 is viewing page1
[2021-02-27 14:44:45,108] INFO in event-simulator: USER2 is viewing page3
[2021-02-27 14:44:46,110] INFO in event-simulator: USER4 is viewing page2
[2021-02-27 14:44:47,110] INFO in event-simulator: USER2 logged out
[2021-02-27 14:44:48,112] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:48,112] INFO in event-simulator: USER4 logged out
[2021-02-27 14:44:49,113] INFO in event-simulator: USER3 is viewing page3
[2021-02-27 14:44:50,115] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2021-02-27 14:44:50,115] INFO in event-simulator: USER2 is viewing page1
[2021-02-27 14:44:51,117] INFO in event-simulator: USER4 is viewing page1
[2021-02-27 14:44:52,118] INFO in event-simulator: USER3 logged in
[2021-02-27 14:44:53,120] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:53,121] INFO in event-simulator: USER2 logged in
[2021-02-27 14:44:54,122] INFO in event-simulator: USER1 logged in
[2021-02-27 14:44:55,124] INFO in event-simulator: USER1 is viewing page3
[2021-02-27 14:44:56,126] INFO in event-simulator: USER2 logged in
[2021-02-27 14:44:57,127] INFO in event-simulator: USER3 is viewing page2
[2021-02-27 14:44:58,129] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:44:58,129] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2021-02-27 14:44:58,129] INFO in event-simulator: USER1 logged in
[2021-02-27 14:44:59,131] INFO in event-simulator: USER4 logged in
[2021-02-27 14:45:00,132] INFO in event-simulator: USER4 is viewing page3
[2021-02-27 14:45:01,134] INFO in event-simulator: USER2 is viewing page3
[2021-02-27 14:45:02,135] INFO in event-simulator: USER4 logged out
[2021-02-27 14:45:03,139] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:45:03,139] INFO in event-simulator: USER1 is viewing page1
[2021-02-27 14:45:04,141] INFO in event-simulator: USER2 is viewing page1
[2021-02-27 14:45:05,142] INFO in event-simulator: USER2 logged in
[2021-02-27 14:45:06,144] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2021-02-27 14:45:06,144] INFO in event-simulator: USER3 logged in
[2021-02-27 14:45:07,145] INFO in event-simulator: USER1 is viewing page1
[2021-02-27 14:45:08,147] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2021-02-27 14:45:08,147] INFO in event-simulator: USER2 is viewing page3
[2021-02-27 14:45:09,149] INFO in event-simulator: USER1 is viewing page3
[2021-02-27 14:45:10,150] INFO in event-simulator: USER2 is viewing page3
[2021-02-27 14:45:11,151] INFO in event-simulator: USER2 logged out
```

* we can see in the above log USER30 had Order failed as the item is OUT OF STOCK.
