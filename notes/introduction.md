# Introduction

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
