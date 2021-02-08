

controlplane $ kubectl get pods --show-labels
NAME          READY   STATUS    RESTARTS   AGE   LABELS
app-1-jghm4   1/1     Running   0          10m   bu=finance,env=dev,tier=frontend
app-1-n9xfq   1/1     Running   0          10m   bu=finance,env=dev,tier=frontend
app-1-z245z   1/1     Running   0          10m   bu=finance,env=dev,tier=frontend
app-1-zzxdf   1/1     Running   0          10m   bu=finance,env=prod,tier=frontend
app-2-bww8s   1/1     Running   0          10m   env=prod,tier=frontend
auth          1/1     Running   0          10m   bu=finance,env=prod
db-1-5blzz    1/1     Running   0          10m   env=dev,tier=db
db-1-b5x9l    1/1     Running   0          10m   env=dev,tier=db
db-1-kqhx7    1/1     Running   0          10m   env=dev,tier=db
db-1-nj5x9    1/1     Running   0          10m   env=dev,tier=db
db-2-7spm5    1/1     Running   0          10m   bu=finance,env=prod,tier=db


controlplane $ kubectl get pods --selector=env=dev --show-labels
NAME          READY   STATUS    RESTARTS   AGE   LABELS
app-1-jghm4   1/1     Running   0          11m   bu=finance,env=dev,tier=frontend
app-1-n9xfq   1/1     Running   0          11m   bu=finance,env=dev,tier=frontend
app-1-z245z   1/1     Running   0          11m   bu=finance,env=dev,tier=frontend
db-1-5blzz    1/1     Running   0          11m   env=dev,tier=db
db-1-b5x9l    1/1     Running   0          11m   env=dev,tier=db
db-1-kqhx7    1/1     Running   0          11m   env=dev,tier=db
db-1-nj5x9    1/1     Running   0          11m   env=dev,tier=db


We have PODs labelled with 'tier', 'env' and 'bu'. How many PODs exist in the 'dev' environment?

controlplane $ kubectl get pods --selector=env=dev --show-labels --no-headers
app-1-jghm4   1/1   Running   0     14m   bu=finance,env=dev,tier=frontend
app-1-n9xfq   1/1   Running   0     14m   bu=finance,env=dev,tier=frontend
app-1-z245z   1/1   Running   0     14m   bu=finance,env=dev,tier=frontend
db-1-5blzz    1/1   Running   0     14m   env=dev,tier=db
db-1-b5x9l    1/1   Running   0     14m   env=dev,tier=db
db-1-kqhx7    1/1   Running   0     14m   env=dev,tier=db
db-1-nj5x9    1/1   Running   0     14m   env=dev,tier=db


controlplane $ kubectl get pods --selector=env=dev --show-labels --no-headers | wc -l
7

We can also use -l instead :
controlplane $ kubectl get pods -l env=dev



How many PODs are in the 'finance' business unit ('bu')?

controlplane $ kubectl get pods --selector=bu=finance
NAME          READY   STATUS    RESTARTS   AGE
app-1-jghm4   1/1     Running   0          15m
app-1-n9xfq   1/1     Running   0          15m
app-1-z245z   1/1     Running   0          15m
app-1-zzxdf   1/1     Running   0          15m
auth          1/1     Running   0          15m
db-2-7spm5    1/1     Running   0          15m


How many objects are in the 'prod' environment including PODs, ReplicaSets and any other objects?

controlplane $ kubectl get all --selector=env=prod
NAME              READY   STATUS    RESTARTS   AGE
pod/app-1-zzxdf   1/1     Running   0          16m
pod/app-2-bww8s   1/1     Running   0          16m
pod/auth          1/1     Running   0          16m
pod/db-2-7spm5    1/1     Running   0          16m

NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.98.8.52   <none>        3306/TCP   16m

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/app-2   1         1         1       16m
replicaset.apps/db-2    1         1         1       16m


Identify the POD which is part of the prod environment, the finance BU and of frontend tier?

controlplane $ kubectl get pods --selector=env=prod,bu=finance,tier=frontend
NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          19m
