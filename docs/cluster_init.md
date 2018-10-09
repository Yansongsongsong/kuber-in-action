# _Kubeadm_ to initialise Master and Slave cluster

> Target:
>
> - Know the basic `Kubeadm` command
> - Know how to build one k8s cluster

```shell

# preparation: you has installed kubeadm, kubelet and kubectl and docker is necessary.
# step1: initialise on master
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

# step1.1: Sometimes, in order for hosts to discover each other, we need to install a Network Service (CNI) to let them discover each other.
# There are a lot of CNIs that can be installed, see
# https://kubernetes.io/docs/concepts/cluster-administration/addons/
# command is
# kubectl apply -f [podnetwork].yaml

# step1.2: In order to get the management rights of the cluster, we need to use k8s `admin.conf` to configure kubectl. The operation is to copy admin.conf to the `$HOME/.kube/config` of the machine you need to use kubectl.


# step2: Select to join the cluster on other nodes
node01 $ kubeadm join 172.17.0.49:6443 --token 102952.1a7dd4cc8d1f4cc5 --discovery-token-ca-cert-hash sha256:b351a0c4a11fb0f278e036bd94b65c7b955c3e94a47a402a259632b4112e3220

# step3: verify
master $ kubectl get nodes

NAME      STATUS    ROLES     AGE       VERSION
master    Ready     master    3m        v1.10.2
node01    Ready     <none>    2m        v1.10.2
```

