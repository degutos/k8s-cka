# MongoDB Project

## Overview

User Browser -> Access MongoExpress (External Service) -> Mongo Express -> MongoDB Internal Service -> MongoDb Pod


### Creating MongoDB deployment yaml 


```
apiVersion: apps/v1
kind: Deployment
metadata:
   name: mongodb-deployment
   labels:
      app: mongodb
spec:
  replicas: 1
  selector:
      matchLabels:
          app: mongodb
  template:
      metadata:
          labels:
              app: mongodb
      spec:
          containers:
          - name: mongodb
            image: mongo
            ports:
            - containerPort: 27017
            env:
            - name: MONGO_INITDB_ROOT_USERNAME
              value: 
            - name: MONGO_INITDB_ROOT_PASSWORD
              value:

```



### How to encode secrets
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ echo -n 'something' | base64
dXNlcm5hbWU=
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ echo -n 'something' | base64
cGFzc3dvcmQxMjM=



### Creating our Secrets yaml file

Add the encoded secrets created before into this ymal file


```
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU=
  mongo-root-password: cGFzc3dvcmQxMjM=
```



### Creating our Secrets

We need create our secrets first than our deployment otherwise we will get an error during deployment creating.


```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl apply -f secret.yaml 
secret/mongodb-secret created
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-ff68l   kubernetes.io/service-account-token   3      26m
mongodb-secret        Opaque                                2      8s
```



### Creating our deployment

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl apply -f mongodb-deploy.yaml 
deployment.apps/mongodb-deployment created
service/mongodb-service created


degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get deployments
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
mongodb-deployment   1/1     1            1           50s

degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
mongodb-deployment-8f6675bc5-v2bfk   1/1     Running   0          57s
```

### Adding Service to yaml file (deployment file)

We can use `---` three dashes in order to add a new configuration to the same yaml file. See the below example has Deployment and Service together 

```
apiVersion: apps/v1
kind: Deployment
metadata:
   name: mongodb-deployment
   labels:
      app: mongodb
spec:
  replicas: 1
  selector:
      matchLabels:
          app: mongodb
  template:
      metadata:
          labels:
              app: mongodb
      spec:
          containers:
          - name: mongodb
            image: mongo
            ports:
            - containerPort: 27017
            env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-password

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```




### Checking the service port matches with the Port and container port

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl describe svc mongodb-service
Name:              mongodb-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=mongodb
Type:              ClusterIP
IP Families:       <none>
IP:                10.103.144.233
IPs:               10.103.144.233
Port:              <unset>  27017/TCP
TargetPort:        27017/TCP
Endpoints:         172.17.0.3:27017
Session Affinity:  None
Events:            <none>


degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
mongodb-deployment-8f6675bc5-vz2f2   1/1     Running   0          13m   172.17.0.3   minikube   <none>           <none>
```


**Realize pod IP is 172.17.0.3 and it matches with the IP in Endpoints**



### Checking Deployment, Service, pod and Secrets


```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get all,secrets
NAME                                     READY   STATUS    RESTARTS   AGE
pod/mongodb-deployment-8f6675bc5-vz2f2   1/1     Running   0          7m49s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP     54m
service/mongodb-service   ClusterIP   10.103.144.233   <none>        27017/TCP   6m49s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb-deployment   1/1     1            1           7m49s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-deployment-8f6675bc5   1         1         1       7m49s

NAME                         TYPE                                  DATA   AGE
secret/default-token-ff68l   kubernetes.io/service-account-token   3      53m
secret/mongodb-secret        Opaque                                2      27m

```

Checking again the service and port 

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP     57m
mongodb-service   ClusterIP   10.103.144.233   <none>        27017/TCP   10m
```



### Creating ConfigMap to store the MongoDB URL

Creating a configmap with the variable mongodb url we can later change the mongodb address and just set the new address into the configmap file and then we don't need re-deploy a new mongoexpress pod with the new address

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```

**Creating the resource**

```
egutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl apply -f configMap.yaml 
configmap/mongodb-configmap created
```


### Creating MongoExpress Deployment

To create a new pod MongoExpress with Deployment, we will need to use an image called mongo-express (from dockerhub) and set 03 variables during creating:
- ME_CONFIG_MONGODB_ADMINUSERNAME
- ME_CONFIG_MONGODB_ADMINPASSWORD
- ME_CONFIG_MONGODB_SERVER

These variables will be necessary in the mongo-express app to reach the mongodb 


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
                configMapKeyRef:
                  name: mongodb-configmap
                  key: database_url

```


**Lets apply and create the mongoexpress deployment**

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl apply -f mongoexpress.yaml 
deployment.apps/mongo-express created
```

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get deploy
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
mongo-express        0/1     1            0           8m35s
mongodb-deployment   1/1     1            1           26h

degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get pods
NAME                                 READY   STATUS                       RESTARTS   AGE
mongo-express-8565796ccb-mtc96       0/1     CreateContainerConfigError   0          8m40s
mongodb-deployment-8f6675bc5-vz2f2   1/1     Running                      0   
```

** Lets check the logs and see if the connection was successfully **

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl logs mongo-express-78fcf796b8-7g88t
Waiting for mongodb-service:27017...
Welcome to mongo-express
------------------------


Mongo Express server listening at http://0.0.0.0:8081
Server is open to allow connections from anyone (0.0.0.0)
basicAuth credentials are "admin:pass", it is recommended you change this in your config.js!
Database connected
Admin Database connected
```

**Lets create now a Service for the mongoexpress deployment**

We should always have a service for a Deployment, so we can create the service in the same deployment yaml file by adding `---` to separe them.

We will need also set add a `type` variable to service yaml instructions as `LoadBalancer` and then this service will exposed outside of the cluster to users to access it. We will also add a variable `nodePort` set, this will be the exposed port to end users. 

The intersting thing is every service is a loadbalancer even those internal ones. If we have a service for a deployment with 02 instances it will work as a LB since it will distribuite requests to each of the pods internally.


Adding service to yaml file

```
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000
```

** Lets create the service just wrote above **


```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl apply -f mongoexpress.yaml 
deployment.apps/mongo-express unchanged
service/mongo-express-service created


degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ kubectl get svc
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP          28h
mongo-express-service   LoadBalancer   10.103.144.109   <pending>     8081:30000/TCP   8s
mongodb-service         ClusterIP      10.103.144.233   <none>        27017/TCP        27h
```

Consideration:
In this example we are using `minikube` to deploy our cluster. Minikube works in a different way and it doesn't show our external-ip for the mongo-express-service LoadBalancer service, as we are seeing above it is showing `pending`, in a production environment we would see the proper address and port to end user to access the service.

For `minikube` env we run:

```
degutos@degutos-laptop:~/git/k8s-cka/mongodb-project$ minikube service mongo-express-service
|-----------|-----------------------|-------------|---------------------------|
| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| default   | mongo-express-service |        8081 | http://192.168.49.2:30000 |
|-----------|-----------------------|-------------|---------------------------|
🎉  Opening service default/mongo-express-service in default browser...
```

As we see above we can reach through the browser the address  http://192.168.49.2:30000



In the Mongo-express app UI (browser) we can create a new Database by using the resource in the UI on top of the page on right side.
If you are able to create a new DB our cluster is working well using the simple method of :

`User Browser -> Access MongoExpress (External Service) -> Mongo Express -> MongoDB Internal Service -> MongoDb Pod`

















