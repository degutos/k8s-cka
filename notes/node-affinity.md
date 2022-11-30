# nodeAffinity


Node affinity allow us to define a Node to receive a Pod
We can also use labels to set pod in a node for simple scenario but more complicated scenario we will need Node Affinity

### Set Node Affinity to the deployment to place the pods on node01 only
```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blue 
  name: blue
spec:
  replicas: 6
  selector:
    matchLabels:
      app: blue
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
      affinity:
         nodeAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                  - blue
status: {}
```

PS: in the above example we are using:

```
Name: blue
Replicas: 6
Image: nginx
NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
Key: color
values: blue
```

Example of allowing pods to go to master node with key node-role.kubernetes.io/master

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: red
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      app: red
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: red
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
      affinity:
         nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
               nodeSelectorTerms:
               - matchExpressions:
                 - key: node-role.kubernetes.io/master
                   operator: Exists
status: {}
```


Validations:
Name: red
Replicas: 3
Image: nginx
NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
Key: node-role.kubernetes.io/master
Use the right operator
