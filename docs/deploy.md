# _Kubectl_ to deploy the service and archive scale 

```shell
# preparation: You are already in the master environment
# preparation1: 检查节点
$ kubectl get nodes

NAME      STATUS     ROLES     AGE       VERSION
node    NotReady   master    3m        v1.10.0

# step1: Use kubectl run to deploy services. Do not choose to expose ports externally.
# Specifies the Docker Image to build, and the number of copies expected
$ kubectl run http --image=katacoda/docker-http-server:latest --replicas=1

# The output shows that the deployment named http is created.
deployment "http" created

# step2: View the services you created
# step2.1: display the service general info
$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
http      1         1         1            0           3s

# step2.2: display the detailed info
$ kubectl describe deployment http
Name:                   http
Namespace:              default
CreationTimestamp:      Mon, 17 Sep 2018 08:50:34 +0000
Labels:                 run=http
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=http
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  run=http
  Containers:
   http:
    Image:        katacoda/docker-http-server:latest
    Port:         <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   http-85ddfcb674 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  1m    deployment-controller  Scaled up replica set http-85ddfcb674 to 1

# step3: Expose the http deployment to the outside world
# Here, ip refers to the ip address of the master in the cluster.
# The following command binds host 8000 port to port 80 of the container and exposes it.
# This command will create a service named http, and use this to expose the service.
$ kubectl expose deployment http --external-ip="172.17.0.6" --port=8000 --target-port=80
service "http" exposed

# step4: verify whether be exposed successfully
$ curl http://172.17.0.6:8000
<h1>This request was processed by host: http-85ddfcb674-jb6qz</h1>

# step5: Use kubectl run to deploy services and expose ports and verify
$ kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001
deployment "httpexposed" created
$ curl http://172.17.0.6:8001
<h1>This request was processed by host: httpexposed-7797565fc6-9xcs7</h1>
# You can see that there is only http in the services list and there is no newly created httpexposed
$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
http         ClusterIP   10.96.219.78   172.17.0.6    8000/TCP   13m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    27m
# The operation that can expose the port is not directly exposed to the docker container. The role of the Pause container is to define the network-related services of the Pod.，
$ docker ps | grep httpexposed
4c3d33c8dbf0        katacoda/docker-http-server                   "/app"                   5 minutes ago       Up 5 minutes                               k8s_httpexposed_httpexposed-7797565fc6-9xcs7_default_7f53f475-ba59-11e8-9eb7-0242ac110006_0
c659b0a719fd        gcr.io/google_containers/pause-amd64:3.0      "/pause"                 5 minutes ago       Up 5 minutes        0.0.0.0:8001->80/tcp   k8s_POD_httpexposed-7797565fc6-9xcs7_default_7f53f475-ba59-11e8-9eb7-0242ac110006_0

# step6: scale container and verify
$ kubectl scale --replicas=3 deployment http
deployment "http" scaled
$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
http-85ddfcb674-r7lqd          0/1       Pending   0          1s
http-85ddfcb674-vg8pv          0/1       Pending   0          5s
http-85ddfcb674-zm2jz          0/1       Pending   0          1s
httpexposed-7797565fc6-985hm   0/1       Pending   0          4s
# loadbalance
$ $ curl http://172.17.0.35:8000
<h1>This request was processed by host: http-85ddfcb674-r7lqd</h1>
$ curl http://172.17.0.35:8000
<h1>This request was processed by host: http-85ddfcb674-zm2jz</h1>
$ curl http://172.17.0.35:8000
<h1>This request was processed by host: http-85ddfcb674-vg8pv</h1>
```