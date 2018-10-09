# _Minikube_ lab for single node cluster
> Target:
>
> - To know the basic minikube command
> - To know the basic kubectl command
> - Try to deploy app on k8s

```shell
# Preparation: installl the minikube

# Step1: verify and start minikube
$ minikube version
minikube version: v0.25.0
$ minikube start
There is a newer version of minikube available (v0.28.2).  Download it here:
https://github.com/kubernetes/minikube/releases/tag/v0.28.2

To disable this notification, run the following:
minikube config set WantUpdateNotification false
Starting local Kubernetes v1.9.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
# `minikube` is one single node k8s platform. user can learn how to use k8s on one machine that deployed `minikube`.
# step2: use kubectl to control k8s

# step2.1: get basic info of cluster
$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.79:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
# step2.2: get info of node
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    33s       v1.9.0

# step3: deploy one app
# step3.1: pull image and expose port
$ kubectl run first-deployment --image=katacoda/docker-http-server --port=80
deployment "first-deployment" created
# step3.2: get pod info
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
first-deployment-5f86d4658d-qhhvv   1/1       Running   0          5s
# step3.3: expose first-deployment service with NodePort
$ kubectl expose deployment first-deployment --port=80 --type=NodePort
service "first-deployment" exposed

# step4: display service, and query the service 
$ kubectl get svc
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)AGE
first-deployment   NodePort    10.105.24.97   <none>        80:31787/TCP12m
kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP17m
$ curl 172.17.0.79:31787
<h1>This request was processed by host: first-deployment-5f86d4658d-qhhvv</h1>
```