# How to deploy one application and what is _deployment_ , _pod_ and etc?

## what is `kubectl run `

The `run` command creates a `deployment` based on the parameters specified, such as the image or replicas. 

## what is `Pod`, `deployment` and etc?

到1.3节，我们的指令里出现了很多不同的概念，比如`Pod`，`service`，`deployment`，`replicationcontroller`，这些都是上文中提到的kubernetes中管理的资源类型。

另外我们还应注意到，资源中还有在kubernetes中被屏蔽掉细节的`container`。

###  `node`

kubernetes集群中的主机节点，是物理的概念。最终一个个任务与命令是由他们执行的。Node 是 Kubernetes 的工作机器，可以是虚拟机或物理机，这取决于在集群的安装情况。

每个节点由 Master 管理。

### `container`

在kubernetes中，所有任务，服务的运行，最终的落脚点，是虚拟化技术的container。kubernetes默认的容器技术是docker，容器化技术除了docker还有rocket等技术，而docker已成为了事实标准。

我们最终所有的应用的部署，都是通过部署docker container来实现的。

### `Pod`

Pod是Kubernetes创建或部署的最小/最简单的基本单位，一个Pod代表集群上正在运行的一个进程。我们可以认为Pod就是应用容器，它是container在kubernetes中的抽象概念，是对container的抽象统一，正如上面所说，Pod的runtime有多种，默认为docker。

**使用方式**

Kubernetes中的Pod使用可分两种主要方式：

- Pod中运行一个容器。

  “one-container-per-Pod”模式是Kubernetes最常见的用法; 在这种情况下，你可以将Pod视为单个封装的容器，但是Kubernetes是直接管理Pod而不是容器。

- Pods中运行多个需要一起工作的容器。

  Pod可以封装紧密耦合的应用，它们需要由多个容器组成，它们之间能够共享资源，这些容器可以形成一个单一的内部service单位 - 一个容器共享文件，另一个“sidecar”容器来更新这些文件。Pod将这些容器的存储资源作为一个实体来管理。

**水平拓展**

每个Pod都是运行应用的单个实例，如果需要水平扩展应用（例如，运行多个实例），则应该使用多个Pods，每个实例一个Pod。在Kubernetes中，这样通常称为Replication。Replication的Pod通常由Controller创建和管理。

**提供资源**

- 网络

  每个Pod被分配一个独立的IP地址，Pod中的每个容器共享网络命名空间，包括IP地址和网络端口。Pod内的容器可以使用localhost相互通信。当Pod中的容器与Pod 外部通信时，他们必须协调如何使用共享网络资源（如端口）。

- 存储

  Pod可以指定一组共享存储*volumes*。Pod中的所有容器都可以访问共享*volumes*，允许这些容器共享数据。*volumes* 还用于Pod中的数据持久化，以防其中一个容器需要重新启动而丢失数据。有关Kubernetes如何在Pod中实现共享存储的更多信息，请参考Volumes。

**controller**

我们一般不直接管理Pod，而是通过一个controller去对它进行控制。比如：

- Deployment
- StatefulSet
- DaemonSet
- Replica Sets
- Replication controller

### `Replication controller`

是一个旧版本的用以控制Pod实例数目的controller，我们现在被推荐使用使用Deployment配置 ReplicaSet（简称RS）方法来控制副本数。

通过使用Replication controller我们可以实现

- 根据运行环境的不同的情况替换、增加或删除Pod实例。
- Rescheduling（重新规划）
- 扩展
- 滚动更新
- 多版本跟踪
- 使用ReplicationControllers与关联的Services

### `service`

Kubernetes Pod 是有生命周期的，他们有自己的IP可以被人发现，但由于他们自身可能的创建销毁导致这些 IP 地址不总是稳定可依赖的。

service就是为了解决这个这个问题而出现的。

Kubernetes Service 定义了这样一种抽象：一个 Pod 的逻辑分组，一种可以访问它们的策略 —— 通常称为微服务。 这一组 Pod 能够被 Service 访问到，通常是通过 Label Selector（查看[2.3.3](#2.3.3 what happened when we input `kubectl run` to run one application?)，查看详情）实现的。

**service的创建**

一个 Service 在 Kubernetes 中是一个 REST 对象，和 Pod 类似。 像所有的 REST 对象一样， Service 定义可以基于 POST 方式，请求 apiserver 创建新的实例。

**定义与解释**

```yaml
kind: Service # 定义了一组Pod的service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp # 标签为app=MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376 # 这组pod的服务暴露在cluster IP的9376端口上
```

### `deployment`

Deployment为Pod和Replica Set（升级版的 Replication Controller）提供声明式更新。你只需要在 Deployment 中描述您想要的目标状态是什么，Deployment controller 就会帮您将 Pod 和ReplicaSet 的实际状态改变到您的目标状态。

但是需要注意的是我们不该手动管理由 Deployment 创建的 Replica Set。

#### **典型用例**

- 创建应用

  使用Deployment来创建ReplicaSet。ReplicaSet在后台创建pod。检查启动状态，看它是成功还是失败。

- 更新应用

  然后，通过更新Deployment的PodTemplateSpec字段来声明Pod的新状态。这会创建一个新的ReplicaSet，Deployment会按照控制的速率将pod从旧的ReplicaSet移动到新的ReplicaSet中。

- 版本回滚

  如果当前状态不稳定，回滚到之前的Deployment revision。每次回滚都会更新Deployment的revision。

- 应用扩容

  扩容Deployment以满足更高的负载。

- 状态监控与rescheduling

  - 暂停Deployment来应用PodTemplateSpec的多个修复，然后恢复上线。
  - 根据Deployment 的状态判断上线是否hang住了。
  - 清除旧的不必要的 ReplicaSet。

#### **定义与解释**

**yaml内容**

```yaml
# https://kubernetes.io/docs/user-guide/nginx-deployment.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

**创建示例**

```shell
# 下面是一个 Deployment 示例，它创建了一个 ReplicaSet 来启动3个 nginx pod。
$ kubectl create -f https://kubernetes.io/docs/user-guide/nginx-deployment.yaml --record
deployment "nginx-deployment" created

# 查看deployment的情况
# 根据deployment中的.spec.replicas配置 我们需要的repalica数是3
# 结果显示了“当前”、“已更新到最新”和“可用”的repalica数量
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s

# 显示Replica Set
# ReplicaSet 的名字总是 <Deployment的名字>-<pod template的hash值>。
# 下面这个被创建的Replica Set将保证总是有3个 nginx 的 pod 存在。
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-2035384211   3         3         0       18s

# 显示已创建的Pod
# pod label 里的 pod-template-hash label 不是用户指定，是Deployment controller 会自动为 Pod 添加的
# 目的是防止 Deployment 的子ReplicaSet 的 pod 名字重复
$ kubectl get pods --show-labels
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-2035384211-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
```

**更新**

```shell
# 我们现在想要让 nginx pod 使用nginx:1.9.1的镜像来代替原来的nginx:1.7.9的镜像。
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated

# 我们可以使用edit命令来编辑 Deployment
# kubectl edit deployment/nginx-deployment

# 查看 rollout 的状态
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out

# Rollout 成功后，get Deployment
# UP-TO-DATE为3 表示已经更新到最新
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           36s

# 查看replica set
# 可以看到 Deployment 更新了Pod，通过创建一个新的 ReplicaSet 并扩容了3个 replica，同时将原来的 ReplicaSet 缩容到了0个 replica。
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         0       6s
nginx-deployment-2035384211   0         0         0       36s

# 执行 get pods只会看到当前的新的 pod
# deployment的更新策略是，当新的Pod创建出来之前不会杀掉旧的Pod。这样能够确保可用的 Pod 数量至少有2个，Pod的总数最多4个。
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

除此之外还有回滚等策略，在这里不详述。

## what happened when we input `kubectl run` to run one application?

![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174216.png)

上图可以看到如下组件，使用特别的图标表示Service和Label：

- Pod
- Container（容器）
- Label(![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174236.png))（标签）
- Replication Controller（复制控制器）
- Service（![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174247.png)）（服务）
- Node（节点）
- Kubernetes Master（Kubernetes主节点）

### Pod

Pod（上图绿色方框）安排在节点上，包含一组容器和卷。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信。Pod是短暂的，不是持续性实体。你可能会有这些问题：

- 如果Pod是短暂的，那么我怎么才能持久化容器数据使其能够跨重启而存在呢？ 是的，Kubernetes支持[卷](http://kubernetes.io/v1.1/docs/user-guide/volumes.html)的概念，因此可以使用持久化的卷类型。
- 是否手动创建Pod，如果想要创建同一个容器的多份拷贝，需要一个个分别创建出来么？可以手动创建单个Pod，但是也可以使用Replication Controller使用Pod模板创建出多份拷贝，下文会详细介绍。
- 如果Pod是短暂的，那么重启时IP地址可能会改变，那么怎么才能从前端容器正确可靠地指向后台容器呢？这时可以使用Service，下文会详细介绍。

### Label

正如图所示，一些Pod有Label（![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174236.png)）。一个Label是attach到Pod的一对键/值对，用来传递用户定义的属性。比如，你可能创建了一个"tier"和“app”标签，通过Label（`tier=frontend, app=myapp`）来标记前端Pod容器，使用Label（`tier=backend, app=myapp`）标记后台Pod。然后可以使用`Selectors`选择带有特定Label的Pod，并且将Service或者Replication Controller应用到上面。

### Replication Controller

是否手动创建Pod，如果想要创建同一个容器的多份拷贝，需要一个个分别创建出来么，能否将Pods划到逻辑组里？

Replication Controller确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们。如果某个Pod不响应，那么Replication Controller会替换它，保持总数为3.如下面的动画所示：

![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/gif.gif)

如果之前不响应的Pod恢复了，现在就有4个Pod了，那么Replication Controller会将其中一个终止保持总数为3。如果在运行中将副本总数改为5，Replication Controller会立刻启动2个新Pod，保证总数为5。还可以按照这样的方式缩小Pod，这个特性在**执行滚动**升级时很有用。

当创建Replication Controller时，需要指定两个东西：

1. Pod模板：用来创建Pod副本的模板
2. Label：Replication Controller需要监控的Pod的标签。

现在已经创建了Pod的一些副本，那么在这些副本上如何均衡负载呢？我们需要的是Service。

### Service

如果Pods是短暂的，那么重启时IP地址可能会改变，怎么才能从前端容器正确可靠地指向后台容器呢？Service是定义一系列Pod以及访问这些Pod的策略的一层抽象。Service通过Label找到Pod组。因为Service是抽象的，所以在图表里通常看不到它们的存在，这也就让这一概念更难以理解。

现在，假定有2个后台Pod，并且定义后台Service的名称为‘backend-service’，lable选择器为（`tier=backend, app=myapp`）。backend-service 的Service会完成如下两件重要的事情：

- 会为Service创建一个本地集群的DNS入口，因此前端Pod只需要DNS查找主机名为 ‘backend-service’，就能够解析出前端应用程序可用的IP地址。
- 现在前端已经得到了后台服务的IP地址，但是它应该访问2个后台Pod的哪一个呢？Service在这2个后台Pod之间提供透明的负载均衡，会将请求分发给其中的任意一个（如下面的动画所示）。通过每个Node上运行的代理（kube-proxy）完成。

![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/service.gif)
