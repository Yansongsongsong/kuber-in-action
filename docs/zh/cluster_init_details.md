# What is `kubeadm` and what is the overall structure of k8s?

## 什么是kubeadm

`Kubeadm`是一个提供了 `kubeadm init` and `kubeadm join` 命令作为构建k8s集群的最佳实践与最快路径。

## k8s的整体架构是怎样的

我们目前已经了解到了`kubectl`与`kubeadm`这个两个`cli`工具，他们只是k8s的一部分。k8s由多个部分提供的服务构成。我们首先要了解到的是k8s的架构体现了很多分布式系统设计的最佳实践，比如组件之间松耦合不存在直接的依赖关系；专注于服务编排，对于存储网络等方面留下插件接口，可以让用户进行拓展。

### 结构构成

k8s运行于两种节点之上，`master`用于管理节点，`minion`用于运行容器。

### 重要组件

Master 节点上运行

- `APIServer`

  作为整个系统的控制入口，以REST API服务提供接口。 

  主要负责暴露Kubernetes API，不管是kubectl还是HTTP调用来操作Kubernetes集群各种资源，都是通过kube-apiserver提供的接口进行操作的。

  - 相应用户的管理请求
  - 进行指挥协调

- `scheduler`

  调度器负责Kubernetes集群的具体调度工作，接收来自于管理控制器（kube-controller-manager）触发的调度操作请求，然后根据请求规格、调度约束、整体资源情况等因素进行调度计算，最后将任务发送到目标节点的kubelet组件执行。

- `controller manager`

  一组控制器（controller）的合集，负责控制管理对应资源。用来执行整个系统中的后台任务，包括节点状态状况、Pod个数、Pods和Service的关联等。 主要由以下几部分组成：

  - 节点控制器（Node Controller）
  - 副本控制器（Replication Controller）
  - 端点控制器（Endpoints Controller）
  - 命名空间控制器（Namespace Controller）
  - 身份认证控制器（Serviceaccounts Controller）

- `etcd`

  etcd是一款用于共享配置和服务发现的高效KV存储系统，具有分布式、强一致性等特点。在Kubernetes环境中主要用于存储所有需要持久化的数据。

minion节点运行

- `kubelet`

  运行在每个计算节点上，作为agent，接受分配该节点的Pods任务及管理容器，周期性获取容器状态，反馈给kube-apiserver。 

- `kube-proxy`

  运行在每个计算节点上，负责Pod网络代理。定时从etcd获取到service信息来做相应的策略。 

其他

- `kubectl`

  客户端命令行工具，将接受的命令格式化后发送给kube-apiserver，作为整个系统的操作入口。 

- `kubeadm`

  一个构建安装以上组件的官方命令行工具

- `DNS `

  一个可选的DNS服务，用于为每个Service对象创建DNS记录，这样所有的Pod就可以通过DNS访问服务了。

- Dashboard

  官方提供的kubernetes集群的UI界面，提供了一些基础的查看及简单操作。

![kubernetes structure](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917163110.png)

## k8s的作用

Kubernetes提供完善的管理工具，涵盖了包括开发、部署测试、运维监控在内的各个环节。提高了大规模容器集群管理的便捷性。作用：

1. 自动化容器的部署和复制 
2. 随时扩展或收缩容器规模 
3. 将容器组织成组，并且提供容器间的负载均衡 
4. 很容易地升级应用程序容器的新版本 
5. 提供容器弹性，如果容器失效就替换它
