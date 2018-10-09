# _Minikube_ lab for single node cluster
> Target:
>
> - 了解基本的minikube的指令
> - 了解基本的kubectl指令
> - 尝试部署简单的应用在kubernetes集群上

```shell
# Preparation 首先在你的电脑上安装minikube。

# Step1: 查看minikube是否安装成功，并启动
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
# minikube 是一个小型的kubernetes环境，用于单机的集群进行k8s的学习和体验
# step2: 使用kubectl套件对kubernetes进行命令行的操作

# step2.1: 获取集群基本信息
$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.79:8443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
# step2.2: 获取集群中的node信息
$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
host01    Ready     <none>    33s       v1.9.0

# step3: 部署一个应用
# step3.1: 拉取镜像并暴露端口
$ kubectl run first-deployment --image=katacoda/docker-http-server --port=80
deployment "first-deployment" created
# step3.2: 获取应用基本信息
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
first-deployment-5f86d4658d-qhhvv   1/1       Running   0          5s
# step3.3: 以NodePort的部署形式，暴露first-deployment service
$ kubectl expose deployment first-deployment --port=80 --type=NodePort
service "first-deployment" exposed

# step4: 查看service，并通过内网访问服务
$ kubectl get svc
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)AGE
first-deployment   NodePort    10.105.24.97   <none>        80:31787/TCP12m
kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP17m
$ curl 172.17.0.79:31787
<h1>This request was processed by host: first-deployment-5f86d4658d-qhhvv</h1>
```