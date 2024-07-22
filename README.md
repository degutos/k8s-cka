# k8s-cka

### Certified Kubernetes Administrators Certification

This is for my studies related to kubernetes getting deeper with Certified Kubernetes Administrators Certification

I have this to add my own notes and examples I have studied 

This is for personal use and I don't intend to explain or teach anyone with this content although feel free to use it as you need.


Architecture:
- ETCD
- KUBE-API
- KUBE CONTROLLER MANAGER
- KUBE SCHEDULLER 
- KUBELET
- KUBE-PROXY


#### Nodes

Nodes are physical or virtual computers which will take the load of your application and kubernetes administration.

In a Kube world is important to have many nodes due to high availability and increase of work load.

Kubernetes pod will run within a node 

The kubelet applications usually runs on each node. The kubelet will communicate with the master node all the time to make sure the pods are healthy and if any interaction or fix is needed it will report to the master node.

#### Master

A Kubernetes cluster will need a master to monitor several nodes. If a node dies a master will monitor that node and will orchestrate our cluster. The master nodes usually runs the kube-api server.

#### API server

All users that access the cluster through CLI will send commands to the API server. Example: kubectl get pods will be received by the API server

#### ETCD

It is a key-value store database responsible to store all kubernetes cluster information. When we have several Nodes and couple of masters all data regarding to this cluster and these components will be store in the ETCD database. Logs are also stored in the ETCD.


#### Scheduler 

It is responsible for allocating and distributing  each pod to each node. It looks for new container and assign it to nodes.

#### Controller 

Controller are the brain behind the kubernetes clusters, it has to respond when some pods, container, nodes or endpoint go down. Controller take decision to bring new pods or container in such cases. 


#### Container runtime

It is called docker but we have several different apps for container runtime. It is responsible for running the containers and pods.


#### Kubelet 

It is an agent that run inside of each node. It is responsible to make sure each container is running in the nodes.


#### Kubectl 

kubectl is a command line tool that exists to the user to communicate and send commands to the cluster through the API Server. Example:

```
$ kubectl get pods

$ kubectl cluster-info

$ kubectl run hello-minikube
```


#### ContainerD x Docker

Docker was discontinued from kubernetes project, they refused to work under the kubernetes convention pattern. Docker refused to adapt his application to use Kubernetes CRI (Container runtime Interface)

- Containerd comes with ctr CLI
- Not very user friendly 
- only supports limited features 
- ctr is more used for debugging containerd
- The nerdctl tool provides stable and human-friendly user experience.

Lets see ctr commands 

```
$ ctr 
$ ctr images pull docker.io/library/redis:alpine
$ ctr run docker.io/library/redis:alpine redis
```

- nerdctl pprovides a docker cli experience
- nerdctl supports docker compose
- nerdctl suports encrypted container images
- Lazy pulling 
- P2P image distribution
- namespace in kubernetes 

Nerdctl has commands very similar to docker
```
$ nerdctl 
$ nerdctl run --name redis redis:alpine
$ nerdctl run --name webserver -p 80:80 -d nginx
```

##### crictl 

- Kubernetes recently created another CLI tool called crictl 
- crictl provides a cli for cri compatible container runtimes
- It is a independent tool developed by kubernetes 
- It has to be installed separately 
- Used to inspect and debug container runtimes but not to create containers
- works across different runtimes



```
$ crictl
$ crictl pull busybox
$ crictl images
$ crictl ps -a
$ crictl exec -i -t 1947763542940 ls
$ crictl logs 1947763542940
$ crictl pods 
```

IMPORTANT: crictl supports reading pods