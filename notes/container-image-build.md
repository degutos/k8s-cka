# Container image

## Manual steps to build your container 

1. Install OS - Ubuntu 
2. Update apt repo
3. Install dependencies using apt
4. Install python dependencies using pip
5. Copy source code to /opt folder
6. Run web server using flask command 


## Create your own Dockerfile

```
From ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql 

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

Note: Each instructions line above will generate a new layer in the container, it is important to keep those instructions consistent and short in order to create a smaller image.

## Build your container

```
$ docker build Dockerfile -t degutos/my-custom-app
```

### Send our new image to docker hub repository

```
$ docker push degutos/my-custom-app
```

### Check container layers

Once we created our new image we can observe the container layers with the following command:

```
$ docker history degutos/my-custom-app
```

Each line in the Dockerfile will represent a new layer in the container.
If you need to rebuild your image either because it failed or you need update your application source code the docker image command will re-build only those layers needed, this will optimize your time build and space.



## Another Dockerfile sample:

```
$ cat Dockerfile 
FROM python:3.6

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

## Building our container image

```
$ docker build -t webapp-color . 
Sending build context to Docker daemon  125.4kB
Step 1/6 : FROM python:3.6
 ---> 54260638d07c
Step 2/6 : RUN pip install flask
 ---> Running in 9b4d191fc7be
Collecting flask
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Collecting click>=7.1.2
  Downloading click-8.0.4-py3-none-any.whl (97 kB)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting importlib-metadata
  Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp36-cp36m-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (30 kB)
Collecting dataclasses
  Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Collecting zipp>=0.5
  Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, importlib-metadata, dataclasses, Werkzeug, Jinja2, itsdangerous, click, flask
Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 importlib-metadata-4.8.3 itsdangerous-2.0.1 typing-extensions-4.1.1 zipp-3.6.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.4; however, version 21.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container 9b4d191fc7be
 ---> a4cdc309360d
Step 3/6 : COPY . /opt/
 ---> 7c9d940f0e55
Step 4/6 : EXPOSE 8080
 ---> Running in 9bd491380cd8
Removing intermediate container 9bd491380cd8
 ---> 6b4f5a079373
Step 5/6 : WORKDIR /opt
 ---> Running in ee4799897b85
Removing intermediate container ee4799897b85
 ---> 8ac166256733
Step 6/6 : ENTRYPOINT ["python", "app.py"]
 ---> Running in 605f60ae27f5
Removing intermediate container 605f60ae27f5
 ---> cc4324a3de20
Successfully built cc4324a3de20
Successfully tagged webapp-color:latest



$ docker images
REPOSITORY                      TAG           IMAGE ID       CREATED          SIZE
webapp-color                    latest        cc4324a3de20   44 seconds ago   913MB
mysql                           latest        7ce93a845a8a   4 weeks ago      586MB
alpine                          latest        324bc02ae123   4 weeks ago      7.79MB
nginx                           latest        a72860cb95fd   2 months ago     188MB
nginx                           alpine        1ae23480369f   2 months ago     43.2MB
ubuntu                          latest        35a88802559d   2 months ago     78MB
redis                           latest        6c00f344e3ef   3 months ago     116MB
postgres                        latest        07a4ee949b9e   3 months ago     432MB
python                          3.6           54260638d07c   2 years ago      902MB
nginx                           1.14-alpine   8a2fb25a19f5   5 years ago      16MB
kodekloud/simple-webapp-mysql   latest        129dd9f67367   5 years ago      96.6MB
kodekloud/simple-webapp         latest        c6e3cd9aae36   5 years ago      84.8MB
$ 
```

## Running a container on port 8080 on the container and port 8282 on the host from a image

```
$ docker run -p 8282:8080 webapp-color
 This is a sample web application that displays a colored background. 
 A color can be specified in two ways. 

 1. As a command line argument with --color as the argument. Accepts one of red,green,blue,blue2,pink,darkblue 
 2. As an Environment variable APP_COLOR. Accepts one of red,green,blue,blue2,pink,darkblue 
 3. If none of the above then a random color is picked from the above list. 
 Note: Command line argument precedes over environment variable.


No command line argument or environment variable. Picking a Random Color =darkblue
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.12.0.2:8080/ (Press CTRL+C to quit)
 ```


## Concept about container definition file


Containers are not mean to run forever like virtual machines, container are mean to run some process or task and close, the container exits.

In the dockerfile we have an instruction called `CMD` that define the default command that the container will run when it is in running state.

In nginx container we have the following line:

```
CMD ["nginx]
```

In the mysql image we have:

```
CMD ["mysqld]
```

In the sample of ubuntu image we have:

```
CMD ["bash"]
```

The problem is the bash is a command that wait for terminal or a command and it will exit with it doesn't find a terminal to attach.
One option is to append a command to docker run command to override the default command in the container.

```
docker run ubuntu [command]

docker run ubuntu slep 5
```

This command will make the container sleeps for 5 seconds and then it exists.

To create a dockerfile with that specific command we can use

```
FROM Ubuntu

CMD sleep 5
```

Then we run:

```
docker build -t ubuntu-sleeper .
```

```
docker run ubuntu-sleeper
```

We should also use the `ENTRYPOINT`  and `CMD` to build a dockerfile

```
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

This way we can run simply:
```
$ docker run ubuntu-sleeper 10
```

This will make the ubuntu sleep 10sec 



## Concept about kubernetes definition file


### In docker we can do

```
docker run --name ubuntu-sleeper ubuntu-sleeper

docker run --name ubuntu-sleeper ubuntu-sleeper 10
```

## In kubernetes world 

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```

Then we run 

```
kubectl create -f pod-definition.yml 
```

So lets compare variables in docker with variable in kubernetes


| Docker  | Kubernetes   | 
|---|---|
| ENTREYPOINT ["sleep"]  | command: ["sleep"]  | 
| CMD ["5"]  | args: ["10"]  | 



## Important Note regarding to editing Pods and Deployment

We can not edit specifications of existing POD other than the following :

```
spec.containers[*].image

spec.initContainers[*].image

spec.activeDeadlineSeconds

spec.tolerations
```

If we run kubectl edit pod we won't be able to edit other fields it will deny to save the change because we are trying to edit a field on the pod that is not editable. 

We can save the pod definition in /tmp file, then we can delete pod and recreate with the new file created.

Second option is to generate a new pod definition file:

```
kubectl get pod webapp -o yaml > my-new-pod.yaml
```

Then we edit the new file and change it then we delete the pod and recreate it

```
kubectl create -f my-new-pod.yaml
```


### Deployments

With deployments you can easily edit any field of a POD. Since the pod template is a child of the deployment , with every change the deployment will automatically delete and create a new pod with new changes. 

So if you are asked to edit a property of a POD, change it in the deployment 

```
kubectl edit deployment my-deployment 
```


## Lab Commands and arguments

Lets create a pod with ubuntu image to run container to sleep for 5000 seconds.

```
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep","5000"]
```

Lets start the pod

```
$ kubectl apply -f ubuntu-sleeper-2.yaml
```

```
controlplane ~ ➜  kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
ubuntu-sleeper-2   1/1     Running   0          5s
```

```
controlplane ~ ➜  kubectl describe po ubuntu-sleeper-2 
Name:             ubuntu-sleeper-2
Namespace:        default
Priority:         0
Service Account:  default
Node:             controlplane/192.9.33.3
Start Time:       Mon, 02 Sep 2024 05:42:07 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.42.0.10
IPs:
  IP:  10.42.0.10
Containers:
  ubuntu:
    Container ID:  containerd://f244bae227381a7dd18a0796275f93aea48b222c44183016bbcd8dbb7f23cdc0
    Image:         ubuntu
    Image ID:      docker.io/library/ubuntu@sha256:8a37d68f4f73ebf3d4efafbcf66379bf3728902a8038616808f04e34a9ab63ee
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      5000
    State:          Running
      Started:      Mon, 02 Sep 2024 05:42:08 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jnrdv (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-jnrdv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  16s   default-scheduler  Successfully assigned default/ubuntu-sleeper-2 to controlplane
  Normal  Pulling    16s   kubelet            Pulling image "ubuntu"
  Normal  Pulled     16s   kubelet            Successfully pulled image "ubuntu" in 149ms (149ms including waiting). Image size: 29710357 bytes.
  Normal  Created    16s   kubelet            Created container ubuntu
  Normal  Started    16s   kubelet            Started container ubuntu
  ```

  Notice that we have the Command set for the container ubuntu

  ```
     Command:
      sleep
      5000
```

### Causing an error in the yaml file

```
apiVersion: v1
kind: Pod 
metadata:
  name: ubuntu-sleeper-3
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - 1200
```

Lets fix the mistake above

```
controlplane ~ ➜  cat ubuntu-sleeper-3.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-3 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "1200"
```

```
~ ➜  kubectl apply -f ubuntu-sleeper-3.yaml 
pod/ubuntu-sleeper-3 created
```

```
 ~ ➜  kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
ubuntu-sleeper     1/1     Running   0          13m
ubuntu-sleeper-2   1/1     Running   0          8m50s
ubuntu-sleeper-3   1/1     Running   0          5s
```

Lets confirm the describe command and the command set

```
~ ➜  kubectl describe po ubuntu-sleeper-3 | grep -i command: -A2
    Command:
      sleep
      1200
```

### Editing and changing pod ubuntu-sleeper-3 to sleep 2000 seconds

```
~ ➜  kubectl edit po ubuntu-sleeper-3 
error: pods "ubuntu-sleeper-3" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1810510277.yaml"
error: Edit cancelled, no valid changes were saved.
```

We changed the command:

```
   Command:
      sleep
      2000
```

Then we need to delete the running pod to start up it again from the file saved in /tmp

```
~ ➜  kubectl delete po ubuntu-sleeper-3
pod "ubuntu-sleeper-3" deleted

~ ➜  kubectl apply -f /tmp/kubectl-edit-1810510277.yaml 
pod/ubuntu-sleeper-3 created
```

We could also use the replace command instead of deleting the pod and apply with -f and set the new /tmp yaml file. Lets have a look how to use the replace command:

```
$ kubectl replace --force -f /tmp/kubectl-edit-1810510277.yaml 
```

```
 ~ ➜  kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
ubuntu-sleeper     1/1     Running   0          20m
ubuntu-sleeper-2   1/1     Running   0          16m
ubuntu-sleeper-3   1/1     Running   0          35s

~ ➜  kubectl describe po ubuntu-sleeper-3 | grep -i command: -A2
    Command:
      sleep
      2000
```



### Dockerfile sample

```
➜  cat Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```

Notice that the command to run during container startup is:

```
ENTRYPOINT ["python", "app.py"]
```

lets take another sample 

```
➜  cat Dockerfile2 
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```

The above command has the following command at startup:

```
python app.py --color red
```

#### Docker file and pod yaml file 

Consider the following dockerfile 

```
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```

The image will be created. After that we will consider the following yaml file

```
apiVersion: v1
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```

Considering these 2 files above the command to run during pod execution is

```
--color green
```


#### Docker file and pod yaml file - another sample


Consider this dockerfile
```
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```


Consider this pod yaml file:

```
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["python", "app.py"]
    args: ["--color", "pink"]
```

The command to run at pod running level is:

```
python app.py --color pink 
```


#### Create a pod that displays a blue background but set arguments to green 

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels: 
    name: webapp-green
spec:
  containers:
   - name: webapp-green
     image: kodekloud/webapp-color
     args: ["--color","green"]

```

Another way of creating this pod yml file is to run the kubectl command with the arguments we want:

```
$ kubectl run webapp-green --image=kodekloud/webapp-color -- --color green
```







