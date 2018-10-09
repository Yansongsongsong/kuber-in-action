# _Kubeadm_ to initialise Master and Slave cluster

> Target:
>
> - 了解基本的`Kubeadm`的指令
> - 了解k8s集群建立的方式

```shell

# preparation 你已在电脑上安装kubeadm, kubelet and kubectl 以及docker
# step1: 在master节点初始化
master $ kubeadm init 
[init] Using Kubernetes version: v1.10.0
[init] Using Authorization modes: [Node RBAC]
...

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following oneach node
as root:

  kubeadm join 172.17.0.49:6443 --token 102952.1a7dd4cc8d1f4cc5 --discovery-token-ca-cert-hash sha256:b351a0c4a11fb0f278e036bd94b65c7b955c3e94a47a402a259632b4112e3220

# step1.1: 有时，为了令主机之间相互发现，我们需要安装一个network服务（The Container Network Interface (CNI) ）供以他们相互发现
# 有很多的CNI可以安装，方式见https://kubernetes.io/docs/concepts/cluster-administration/addons/
# 命令为 kubectl apply -f [podnetwork].yaml

# step1.2: 我们为了可以获得集群的管理权，需要使用k8s的admin.conf进行kubectl的配置，操作是将admin.conf拷贝至你需要使用kubectl的机器的$HOME/.kube/config


# step2: 在其他节点上选择加入集群
node01 $ kubeadm join 172.17.0.49:6443 --token 102952.1a7dd4cc8d1f4cc5 --discovery-token-ca-cert-hash sha256:b351a0c4a11fb0f278e036bd94b65c7b955c3e94a47a402a259632b4112e3220

# step3: 验证
master $ kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    3m        v1.10.2
node01    Ready     <none>    2m        v1.10.2
```

