# Resource Requirements and Limits


- Pods consume real physical resources, like CPU memory and disks
- The scheduler job is responsible to schedule which pod goes to which Node based on resources available in the Node and required for the pod
- If host has no enough resource the scheduler will schedule the pod to sit into another Host.
- If there is no enough resources in any host, scheduler will hold back the pod, you will see the pod in pending state with message Insufficient cpu (or memory according to resource)
- A container can consume all the resources inside of the node, there is no limit, the limit is the host physical limit


## Resources Requirements

- By default each Pod consumes
* 0,5 cpu
* 256 Mi memory
* Disk

- When we start a new pod the scheduler will check if any host has this amount of resources available.
- If we know our pod will consume more than this we can modify and specify that into the pod definition file (YAML file) under spec session we can create a `resoures` session

```
spec:
   resources:
      requests:
         memory: "1Gi"
         cpu: 1
```


1G (gigabyte) = 1,000,000,000 bytes

1Gi (gibibyte) = 1,073,741,824 bytes


- A container has not limit of resources, it can consume all the host resources
- We can limit the maximum limit allowed to a container to consume


- "When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi". For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.


```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

and also

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```


References:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource



## Extras

Edit a POD
Remember, you CANNOT edit specifications of an existing POD other than the below.

spec.containers[*].image

spec.initContainers[*].image

spec.activeDeadlineSeconds

spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the kubectl edit pod <pod name> command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.


A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

```
kubectl delete pod webapp
```


Then create a new pod with your changes using the temporary file

```
kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
```


2. The second option is to extract the pod definition in YAML format to a file using the command

```
kubectl get pod webapp -o yaml > my-new-pod.yaml
```


Then make the changes to the exported file using an editor (vi editor). Save the changes

```
vi my-new-pod.yaml
```

Then delete the existing pod

```
kubectl delete pod webapp
```

Then create a new pod with the edited file

```
kubectl create -f my-new-pod.yaml
```




Edit Deployments
With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification,  with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

kubectl edit deployment my-deployment
