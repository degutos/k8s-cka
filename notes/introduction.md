# Introduction

## Overview

These are the main Kubernetes Components summarized:

- Pod
- Service
- Ingress
- ConfigMap
- Secrets
- Volumes
- Deployment
- StatefulSet

Lets summarize each of these main core K8s components

#### Pod

Pod is the basic and smallest element k8s will manage. Each pod has it own IP and it usually have one container inside. Pod runs containers with your application, example: nginx, db-mongo, etc. Pods Ips are changed when the pod dies, a new pod is created and a new IP is assigned.

#### Service

Service will allow communication between Pods, is uses permanent IP, each Pod has it own service. If we have a node with two pods inside one for my-app and another one for DB each of these pods will have his own service. A Pod can die and the service wouldn't. 

We can have also external Service which will allow external communication to the Node, for example when you would like to allow external user to access your application inside of the Cluster. So we can have external service for the just commented situation and internal service to allow internal communication between Pods, for example to your application pod access the DB pod.


#### Ingress

Ingress can be used very similar to External service and its own purpose is to allow external communication between the user and the application pod. Instead of creating external service we can create an ingress component on k8s and allow communication to internal service. 

Ingress will give an external address like https://my-app.com

So the external user communication to the Ingress through the address https://my-app.com which will redirect the user to the internal service for my-app, then my-app service will communicate with db service


#### ConfigMap

Lets consider we have one Node with 02 pods: my-app and db, each of these pods have it own service (internal). The application access the db using the databse URL which usually is set in the application. When the application url changes for some reason we would have to deploy (rebuil) a new application with the new URL in order to the communication work.

To fix the problem k8s has a component called ConfigMap which can store configuration data like database URL, and external configuration of your application, also user and password if needed although it is not safe store sensitive data in this file as a plain text. 
ConfigMap are attached to the pod so the pod can get data from ConfigMap file and read variables needed. 

If your DB URL changes we can just fix the ConfigMap file with the new URL and that's all.

User database and password can also change, although as mentioned before it is not safe and secure to store sensitive data into ConfigMap file, for that you may use Secrets k8s component.

#### Secrets

Secrets are like ConfigMap but it can be stored not just as a plain text but store as a secret data in 64 encode format. Secrets could config things like user and password also certificates. This would avoid us to rebuild the pod in case any password user changes.


#### Volumes


Pods data are not persistent, if a DB pod get restarted the data would be gone, this is the default behaviour of pod, k8s doesn't take care of our data, we need to manage and persist our data.

To have data persistent is using another componement called Volumes. Volumes are created from hard drives, storages, local or remote through the network outside of k8s cluster, even on cloud.

Once we have our k8s cluster with Volumes accessing an storage (remote or local) we would have our data persistent.


#### Deployments

Deployments are responsible to make my pod is running and make sure a new pod would be created in case the pod is not responding, even if you try delete the pod the deployment would create a new pod with an image for your application. 

Also it can scale up or scale pods down. We can set a number of 03 pods for my application and the deployment will make sure it has always 03 pods running for the application so if you one application (pod) dies, it has other 02 pods running until the deployment creates a new pod to replace the dead one.

If we have 02 or more nodes in our cluster the deployment would split those nodes into different nodes to garantee the cluster would be up even we one Node goes down.

Deployments uses replicas as a load balancer also if you have more pods for the same application.

DB pods can't be used as deployment since DB servers need data consistent and has state data and can not be replicated to another DB server through the deployments. For this we would need StefulSet which is useful specially for DB server in k8s 


#### StatefulSet

StatefulSet are like deployments but specially for DB server. StatefulSet can scale DB pod up or down, re-create DB pods but with data consistently.

So deployment is for application as the StatefulSet is for DB pods.



**PS: Lets talk again with more details about POD**


### Pod

This is the main elements inside of our pod yaml file that specify instruction on how to create a pod.

pod-structure.yaml
```
apiVersion:
kind:
metadata:


spec:


```


Below are the options (kind and Version) we can have for the above structure

```
POD => v1
Service => v1
ReplicaSet => apps/v1
Deployment => apps/v1
```

Example:

```
apiVersion: v1
kind: Pod
metadata:
   name: myapp-pod
   labels:
      app: myapp
      type: front-end
spec:
   containers:
     - name: nginx-container
       image: nginx
```

To run the kubectl with yaml file:
```
$ kubectl create -f pod-structure.yaml
```

To see the pods:
```
$ kubectl get pods
```

To see details (describe) of a pod:
```
$ kubectl describe pod myapp-pod
```

To see the pods in real time with --watch

```
controlplane ~ ➜  k get pod --watch
NAME            READY   STATUS         RESTARTS   AGE
nginx           1/1     Running        0          18m
newpods-dvrd5   1/1     Running        0          15m
newpods-f8zbp   1/1     Running        0          15m
newpods-5tlnx   1/1     Running        0          15m
redis           0/1     ErrImagePull   0          11s
redis           0/1     ImagePullBackOff   0          14s
redis           0/1     ErrImagePull       0          25s
redis           0/1     ImagePullBackOff   0          38s
redis           0/1     ErrImagePull       0          50s
redis           0/1     ImagePullBackOff   0          64s
```


#### How to exec into a pod

```
controlplane ~ ➜  kubectl exec -it redis -- /bin/bash
root@redis:/data# 
```





## ReplicaSets

Notes:

- Help us to keep one or more pods running
- Even if it is only one pod running inside a Node and this pod or application dies for some reason the Replication controller will make sure to start a new pod inside of the Node.
- If we have hundreds of pods of the same application and one pod dies, the replication controller will also make sure that the pod get created again to keep the number of replicasSet running.
- ReplicasSet can be used also to scale your environment up or down by adding or removing pods of your application
- ReplicaSets is also used to keep structure in order to have a load balancing working to balance the load across the cluster

Lets create a replicationController definition:


#### replicationController

```
apiVersion: v1
kind: ReplicationController
metadata:
   name: myapp-rc
   labels:
      app: myapp
      type: front-end

spec:
   template:
      metadata:
         name: myapp-pod
         labels:
            app: myapp
            type: front-end
      spec:
        containers:
          - name: nginx-container
            image: nginx

   replicas: 3
```


Note: Realize that under template we have all the sintaxe of the pod-definition (with NO apiVersion and kind). Also realize that the spec of the replicationController we have template and replicas


To create ReplicationController:
```
$ kubectl create -f rc-definition.yaml
replicationcontroller/myapp-rc created
```

To check replicationController created:
```
$ kubectl get replicationcontroller
NAME       DESIRED   CURRENT   READY   AGE
myapp-rc   3         3         3       16s
```

We still can check all the pods created by the replicationcontroller:
```
$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
myapp-rc-2frz4   1/1     Running   0          5s
myapp-rc-m6kz2   1/1     Running   0          5s
myapp-rc-wtl4t   1/1     Running   0          5s
```

#### ReplicaSets

Note:
- ReplicasSets and and ReplicationController are very similar. ReplicasSet is currently used
- They are usually used for High Availability, load balancing and scaling
- Very similar to replicationController
- Replicaset will control the number of pod by using the field selector and matchLabels
- The field matchLabels should matches with the pod label. Example:
- If we have a pod with label type: front-end we should have matchLabels: and type: front-end also.
- So, replicaset control and guarantee that the number of pods running be the exactly what is required. We can create a replicaset even after we have already running pods
- Replicaset will use labels to monitor amount of pods running. Example:
- Pod-definition says labels: tier: front-end. Replicaset says selector: matchLabels: tier: front-end


Example:
```
$ pod-definition.yaml
metadata:
   name: myapp-pod
   labels:
      tier: front-end
```

In this case the replicaset will have the same label:
```
$ replicaset-definition.yaml
selector:
   matchLabels:
      tier: front-end
```

Lets see now the full yaml file for this replicaset

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: myapp-replicaset
   labels:
      app: myapp
      type: front-end
spec:
   template:
      metadata:
         name: myapp-pod
         labels:
            app: myapp
            type: front-end
      spec:
         containers:
           - name: nginx-container
             image: nginx

   replicas: 3
   selector:
      matchLabels:
         type: front-end
```


Creating ReplicaSet
```
$ kubectl create -f replicaset-definition.yaml
replicaset.apps/myapp-replicaset created
```

Checking replicasets created:
```
$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
myapp-replicaset   3         3         3       13s
```


Checking pods created by this replicaset:
```
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-replicaset-8tr6r   1/1     Running   0          4m33s
myapp-replicaset-dn9hh   1/1     Running   0          4m33s
myapp-replicaset-p5k9f   1/1     Running   0          4m33s
```

How to scale replicas

- First we change the file replicaset-definition.yaml file by changing replicas to 6, then we run:

```
$ kubectl replace -f replicaset-definition.yaml
```

or

```
kubectl scale --replicas=6 -f replicaset-definition.yaml
```

PS: on the above example the original file will not be changed to 6.

or

```
kubectl scale --replicas=6 replicaset my-app-replicaset
```


Note: In case we have a kubernete resource already created but we don't have any definition file, we can create a definition file from the resource created. Example:

```
kubectl get replicaset my-app-replicaset -o yaml > my-app-replicaset.yaml
```


Note:
If we have a replicaset created and we have an error with the container image we can fix the container image in the definition file, delete the replicaset and create again or edit the replicaset with live configuration, change the image name and delete all the pods. This will recreate new pods with new image name. Example:

```
kubectl edit rs my-app-replicaset
```

Then we change the image name under container and save the file

Then we delete all pods related to this replicaset

```
kubectl delete po my-pod-xxxx
```

Repeat the above command till delete all pods.



## Deployment


- Deployment is quiet similar to replicaset but still better
- Deployment we can also have multiples pods (replicas) of our application
- We can scale pods up or down
- We can update new application version when we have new container image available in our repository (docker hub).
- We can update new application version one pod at the time without causing the full cluster to go down. This is called Rolling Deployment
- We can undo deployment update if something went wrong by rolling back to last version stable.
- We can pause all the environment make a change and resume again, in case we need of this resource
- When we create a deployment, kubernetes create deployment, replicaset and pods all together
- The command "Kubectl get all" show us all the above resources created
- The deployment definition file is similar to replicaset definition file except for the kind which is Deployment instead ReplicaSet

#### Example of yaml file to create a deployment


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```




#### Tips on how to create yaml file in a easier way

```
Create an NGINX Pod



ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl run nginx --image=nginx
pod/nginx created

ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          13s



Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run) and print the output on the screen

ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl run nginx --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



Create a deployment (also create replicaset and pod)

kubectl create deployment --image=nginx nginx



Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

ubuntu@andre-vsi-dal-ubuntu-master:~$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

ubuntu@andre-vsi-dal-ubuntu-master:~$ ls -l nginx-*
-rw-rw-r-- 1 ubuntu ubuntu 384 Nov 24 22:13 nginx-deployment.yaml

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.




Create an NGINX Pod

kubectl run --generator=run-pod/v1 nginx --image=nginx



Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml



Create a deployment

kubectl create deployment --image=nginx nginx


Generate Deployment YAML file (-o yaml) Dont create it (dry-run=client) with 03 Replicas

kubectl create deployment --image=httpd:2.4-alpine httpd-frontend --replicas=3 --dry-run=client -o yaml > deployment-httpd-frontend.yaml


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

kubectl create deployment --image=nginx nginx --dry-run -o yaml



Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml

Save it to a file, make necessary changes to the file (for example, adding more replicas) and then create the deployment.


Create pod using dry-run to create a yaml file

$ kubectl run redis --image=redis --dry-run=client -o yaml > pod.yaml

To execute the yaml file and create a pod

$ kubectl apply -f pod.yaml


How to edit pod while Running

$ kubectl edit pod redis



```



## Namespaces

- Namespaces are virtual environment inside our cluster used to separate resources. For example, we can have a namespace called production and another namespace dev or staging or testing,
- We can have quota per namespace, this is a good way of limiting resources per group or teams or customers.
- Each namespace could belong to specific customer in a cloud environment.
- We have the default namespace which is the first namespace used if we don't specify which namespace we want to reach or create resources
- kube-system is a namespace used by kubernetes to manger the cluster
- we can reach resources in different Namespaces by addressing the full name, Example: db-service.dev.svc.cluster.local
Where:
cluster.local -> domain
svc -> service
dev -> namespace
dev-service -> service name


List Namespaces
```
kubectl get ns
```

List pods in default namespace
```
kubectl get pods
```

List pods in different namespace
```
kubectl get pods --namespace=kube-system
```

Create a pod in different namespace
```
kubectl create -f pod-definition.yaml --Namespaces=dev
```

Define namspace in pod definition file
```
apiVersion: v1
kind: Pod
metadata:
   name: myapp
   namespace: dev
```

Create namespace
```
kubectl create ns dev
```

Create Yaml file to create namespace
```
apiVersion: v1
kind: Namespace
metadata:
    name: dev
```


Creating namespace with yaml file
```
kubectl create -f namespace.yaml
```

Change default namespace
```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

List resources in all Namespaces
```
kubectl get all --all-namespaces
```

Creating quota specification file

```
apiVersion: v1
kind: ResourceQuota
metadata:
   name: compute-quota
   namespace: dev
spec:
   hard:
      pods: "20"
      requests.cpu: "6"
      requests.memory: 8Gi
      limits.cpu: "20"
      limits.memory: 20Gi
```

Checking how many pods there are on spcecific namespace

$ kubectl get pods --namespace=research
NAME    READY   STATUS             RESTARTS   AGE
dna-1   0/1     CrashLoopBackOff   2          68s
dna-2   0/1     CrashLoopBackOff   2          68s

Showing pods without headers. Useful when we need to count pods with `| wc -l`

```
kubectl get pods --no-headers | wc -l
```

How to create a pod in a finance namespace

```
controlplane ~ ➜  kubectl run redis --image=redis -n finance
pod/redis created
```

How to list all namespaces and filter by pod name

```
controlplane ~ ➜  kubectl get pods -A | grep blue
marketing       blue                                      1/1     Running            0               5m40s
```





## Services

- Services exposes your application to outside of the application.
- Services enables a user outside of the cluster to access an application in the pod inside of the cluster. Example when a user access your web application inside of the cluster in a pod
- Services helps communication between other Services


### There are 03 types of Services

- NodePort
- ClusterIP
- LoadBalancer



* NodePort is were the service makes an internal POD accessible on a Port on the Node.


* ClusterIP Service creates a virtual IP inside the cluster to enable communication between different services such as a set of front end servers to a set of back end servers.

- LoadBalancer is where it provisions a load balancer for our service in supported cloud providers. A good example of that would be to distribute load across the different web servers in your front end tier.


#### NodePort

- Lets consider we have a container running nginx and exposes port 80 to the pod. We call this port as "Target Port".

- The service communicate with the container through the port 80 which we call as a "Port", this is the Service port itself. It has a specific IP inside of the Cluster.

- The service has a "NodePort" which is the port that the Service exposes to world outside of the node. This NodePort should be between 30000 to 32767. We use labels and Selectors to link the service to the Pod. The pod has labels and we need bring those labels as Selector in the Service definition file.

Lets see the service definition file

nodepod-service-definition.yaml
```
apiVersion: v1
kind: Service
metadata:
   name: app-service

spec:
   type: NodePort
   ports:
     - targetPort: 80
       port: 80
       nodePort: 30008  
   selector:
      app: my-app
      type: front-end
```

Creating a service
```
kubectl create  -f nodepod-service-definition.yaml
```


Showing services created
```
kubectl get services
```

Note:
We use the Node IP and NodePort to access the application. Example:
```
curl https://192.168.1.2:30008
```


If we have a our application in 03 pods in 03 Nodes, one pod each node, when we create a service it is going to create this service in those 03 differents nodes. So we can access the service by using one of those 03 Nodes IPs followed by a port ( 30008 ). Example:
```
curl https://192.168.0.2:30008
curl https://192.168.0.3:30008
curl https://192.168.0.4:30008
```



#### Cluster IP


- Lets consider we have 03 pod running front-end and 03 pods running back end
- We can have a service clusterIP called back-end which will make the 03 backend pod available to frontend pods. This way even the backend pod dies and get a new IP the service will be the same and will choose the better pod to forward the request.
- Each service has an IP and name.



Lets see our definition file

service-clusterip-definition.yaml
```
apiVersion: v1
kind: Service
metadata:
   name: myapp-service
spec:
   type: ClusterIP
   ports:
     - targetPort: 80
       port: 80
   selector:
      app: my-app
      type: back-end
```


#### LoadBalancer

- When we have services of type NodePort provisioned in different hosts hypervisors we can access the service through the Node IP, Example:
```
http://102.168.56.70:30008
http://102.168.56.71:30008
http://102.168.56.72:30008
```

Although this is not what customer wants, they want to access through something like http://example-volte.com, for this we will need a LoadBalancer Service to get the request and forward to the pods.

Lets see an example of Loadbalancer service definition file

service-loadbalancer-definition.yaml

```
apiVersion: v1
kind: Service
metadata:
   name: myapp-service
spec:
   type: LoadBalancer
   ports:
     - targetPort: 80
       port: 80
       nodePort: 30008
   selector:
      app: my-app
      type: back-end
```



# Hands on with Deployment and Service


## Lets create a deployment from this yaml file

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: httpd-frontend
  name: httpd-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd-frontend
  template:
    metadata:
      labels:
        app: httpd-frontend
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: httpd
```

Note: To access this deployment / pod running httpd-alpine we need to expose a port outside of the Node, and we need a service for this.

## Lets create a service that will expose that httpd-alpine to the world by using a service of type NodePort

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
          app: httpd-frontend
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
```

Note: Realise that we added selector "app: httpd-frontend" which is the same label of the deployment create and consequently same label of the pod





##  Checking Services existing

```
controlplane $ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m17s
controlplane $ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2m26s
```


## Getting more information about a service

```
controlplane $ kubectl describe service kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         172.17.0.12:6443
Session Affinity:  None
Events:            <none>
```


## Lets create a Service when we already have a pod/Deployment running. Consider the following Pod/Deployment running

```
controlplane $ kubectl describe deployment simple-webapp-deployment
Name:                   simple-webapp-deployment
Namespace:              default
CreationTimestamp:      Sat, 05 Dec 2020 15:27:19 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=simple-webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=simple-webapp
  Containers:
   simple-webapp:
    Image:        kodekloud/simple-webapp:red
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   simple-webapp-deployment-b56f88b77 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  67s   deployment-controller  Scaled up replica set simple-webapp-deployment-b56f88b77 to 4
```

## To create the service according to the above deployment created we need the following yaml file:

```
controlplane $ cat service-definition-1.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 8080
      port: 8080
      nodePort: 30080
  selector:
    name: simple-webapp
```



## Creating the service according to the yaml file:

```
controlplane $ kubectl create -f service-definition-1.yaml
service/webapp-service created
```

We could also use  the expose command to create a service or create a yaml file

```
$ kubectl expoese deployment simple-webapp-deployment --name=webapp-service --type=NodePort --target-port=8080 port=8080  --dry-run=client -o yaml > svc.yaml
```

After creating the file, we need to edit it and add nodePort: 30080 to the file



## Imperative vs Declarative approach

### Imperative

On Imperative approach we instruct the kubernetes what do to. Example:

To create a pod:

```
$ kubectl run --image=nginx nginx
```

To create a deployment:

```
$  kubectl create deploymeent  --image=nginx nginx
```

To expose deployment by creating a service

```
$ kubecttl expose deployment nginx --port 80
```


To edit deployment

```
kubectl edit deployment nginx  
```


To scale a deployment or replicaSet:

```
$ kubectl  scale deployment nginx --replicas=5
```


To change a image on a  deployment:

```
$ kubectl set image deployment nginx nginx=nginx:1.18
```

We can also use files to create object by using:

```
$ kubectl create -f nginx.yaml
```

or  we still can edit or  delete objects:

```
$ kubectl  edit deployment nginx
$ kubecttl replace -f nginx.yaml
$ kubectl delete -f nginx.YAML
```


Note:
When we use edit command above we edit the kubernete cache object and not the yaml file, every modification will reflect direct to kubernetes memory  and  not the nginx.yaml file. If  we want to track modifications we should change the  yaml file instead using the edit command, after modifying the nginx.yaml file we have to run the command "kubectl replace -f nginx.yaml"


All the above examples are  Imperative Approaches to manage object in kubernetes


### Declarative

We can use apply command to create, edit and delete objects:

``` 
$ kubectl apply -f nginx.yaml
```

The apply command is inteligent enough to make changes without errors, doensn't matter if the object exists or not, if not it will create the object if exists  it will modify according to yaml file



# Important note about editing PODs and Deployments

## Editing POD

`Remember, you CANNOT edit specifications of an existing POD other than the below.`

```
spec.containers[*].image

spec.initContainers[*].image

spec.activeDeadlineSeconds

spec.tolerations
```

When we run `kubectl edit pod <pod name>` we will not be able to edit properties required, we will be denied to save modifications.

Solution 1:

Save the modifications to a temp file `/tmp/kubectl-edit-ccvrq.yaml`
Delete the pod `kubectl delete pod webapp`
Then create the new pod with the changes made kubectl create -f `/tmp/kubectl-edit-ccvrq.yaml`

Solutions 2:

```
kubectl get pod webapp -o yaml > my-new-pod.yaml
vi my-new-pod.yaml
kubectl delete pod webapp
kubectl create -f my-new-pod.yaml
```

## Editing Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

`kubectl edit deployment my-deployment`




# How to autocomplete on kubectl commands

```
$ source <(kubectl completion zsh)
```



## Imperative commands


#### create a pod with nginx image with imperative 

```
 ~ ➜  kubectl run nginx-pod --image=nginx:alpine
pod/nginx-pod created
```


#### how to create pod redis with labels

```
➜  kubectl run redis --image=redis:alpine --labels=tier=db
pod/redis created
```

and showing labels

```
➜  kubectl get po redis --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
redis   1/1     Running   0          84s   tier=db
```


#### how to expose a pod creating a service 

```
✖ kubectl expose pod redis --name=redis-service --port=6379 --type=ClusterIP
service/redis-service exposed
```

```
➜  kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.43.0.1      <none>        443/TCP    14m
redis-service   ClusterIP   10.43.97.172   <none>        6379/TCP   48s
```

#### create a deployment with image webapp-color and 3 replicas with imperative command

```
 ➜  kubectl create deployment webapp --image kodekloud/webapp-color --replicas=3
deployment.apps/webapp created
```

```
➜  kubectl get deploy
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
webapp   3/3     3            3           59s

 ➜  kubectl get rs
NAME                DESIRED   CURRENT   READY   AGE
webapp-7bc4ff899b   3         3         3       70s

➜  kubectl get pods | grep webapp
webapp-7bc4ff899b-lgbqb   1/1     Running   0          2m16s
webapp-7bc4ff899b-7mrnj   1/1     Running   0          2m16s
webapp-7bc4ff899b-t96db   1/1     Running   0          2m16s

```

Instead of grep we can also show labels and filter by app=deployment_name

```
➜  kubectl get pods -l app=webapp
NAME                      READY   STATUS    RESTARTS   AGE
webapp-7bc4ff899b-k4pjm   1/1     Running   0          4m6s
webapp-7bc4ff899b-vtlg6   1/1     Running   0          4m6s
webapp-7bc4ff899b-7plv5   1/1     Running   0          4m6s
```

#### Create a custom-pod running nginx on port 8080

```
➜  kubectl run custom-nginx --image=nginx --port=8080
pod/custom-nginx created
```

#### Create a new namespace

```
➜  kubectl create ns dev-ns
namespace/dev-ns created
```


#### Create a new deployment on a dev-ns namespace with the redis image and 2 replicas

```
➜  kubectl create deployment redis-deploy -n dev-ns --image=redis --replicas=2
deployment.apps/redis-deploy created
```

```
controlplane ~ ➜  kubectl get deploy -n dev-ns
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-deploy   2/2     2            2           3m33s

controlplane ~ ➜  kubectl get rs -n dev-ns
NAME                      DESIRED   CURRENT   READY   AGE
redis-deploy-546bf56c5f   2         2         2       3m40s

controlplane ~ ➜  kubectl get pods -n dev-ns
NAME                            READY   STATUS    RESTARTS   AGE
redis-deploy-546bf56c5f-gzzks   1/1     Running   0          3m49s
redis-deploy-546bf56c5f-p9b8f   1/1     Running   0          3m49s
```


#### Create a pod and expose it in a service on port 80

```
$ kubectl run httpd --image=httpd:alpine --port=80 --expose=true
```



