## what is `minikube` and what is k8s?

### 什么是kubernetes？

首先，他是一个全新的基于容器技术的分布式架构领先方案。Kubernetes(k8s)是Google开源的**容器集群管理系统**（谷歌内部:Borg）。实现了为容器化的应用提供：

- 部署运行、
- 资源调度、
- 服务发现
- 动态伸缩

Kubernetes是一个完备的分布式系统支撑平台，具有完备的集群管理能力，多扩多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和发现机制、內建智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制以及多粒度的资源配额管理能力。

### 什么是`minikube`

`minikube`是一个单机的k8s平台，用户可以在部署了`minikube`的机器上学习使用k8s。

### 什么是`kubectl`

`kubectl`是用于针对Kubernetes集群运行命令的命令行接口（cli），是和Kubernetes API交互的命令行程序。可以操作k8s上的应用，获取集群的信息。

#### 语法

从您的终端窗口使用以下语法运行`kubectl`命令：

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

其中command，TYPE，NAME，和flags分别是：

- `command`: 指定要在一个或多个资源进行操作，例如`create`，`get`，`describe`，`delete`。

- `TYPE`：指定[资源类型](https://kubernetes.io/cn/docs/user-guide/kubectl-overview/#%E8%B5%84%E6%BA%90%E7%B1%BB%E5%9E%8B)。资源类型区分大小写，您可以指定单数，复数或缩写形式。例如，以下命令产生相同的输出：

  ```shell
  $ kubectl get pod pod1  
  $ kubectl get pods pod1 
  $ kubectl get po pod1
  ```

- `NAME`：指定资源的名称。名称区分大小写。如果省略名称，则会显示所有资源的详细信息,比如`$ kubectl get pods`。

  - 在多个资源上执行操作时，可以按类型和名称指定每个资源，或指定一个或多个文件：

  - 按类型和名称指定资源：
      * 要分组资源，如果它们都是相同的类型：`TYPE1 name1 name2 name<#>`.
      例: `$ kubectl get pod example-pod1 example-pod2`
      
      * 要分别指定多种资源类型:  `TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name<#>`.
      例: `$ kubectl get pod/example-pod1 replicationcontroller/example-rc1`

  使用一个或多个文件指定资源： `-f file1 -f file2 -f file<#>` 使用[YAML而不是JSON](https://kubernetes.io/docs/concepts/configuration/overview/#general-config-tips)，因为YAML往往更加用户友好，特别是对于配置文件。
  例：$ kubectl get pod -f ./pod.yaml

- flags：指定可选标志。例如，您可以使用`-s`或`--serverflags`来指定Kubernetes API服务器的地址和端口。 

  **重要提示**：从命令行指定的标志将覆盖默认值和任何相应的环境变量。

如果您需要帮助，只需从终端窗口运行`kubectl help`。

#### 指令详情

##### command

对于`kubectl [command]`，所有的使用可以在[这里](https://kubernetes.io/docs/reference/kubectl/kubectl/)找到。

##### type

| 资源类型                     | 缩写别名      |
| ---------------------------- | ------------- |
| `apiservices`                |               |
| `certificatesigningrequests` | `csr`         |
| `clusters`                   |               |
| `clusterrolebindings`        |               |
| `clusterroles`               |               |
| `componentstatuses`          | `cs`          |
| `configmaps`                 | `cm`          |
| `controllerrevisions`        |               |
| `cronjobs`                   |               |
| `customresourcedefinition`   | `crd`, `crds` |
| `daemonsets`                 | `ds`          |
| `deployments`                | `deploy`      |
| `endpoints`                  | `ep`          |
| `events`                     | `ev`          |
| `horizontalpodautoscalers`   | `hpa`         |
| `ingresses`                  | `ing`         |
| `jobs`                       |               |
| `limitranges`                | `limits`      |
| `namespaces`                 | `ns`          |
| `networkpolicies`            | `netpol`      |
| `nodes`                      | `no`          |
| `persistentvolumeclaims`     | `pvc`         |
| `persistentvolumes`          | `pv`          |
| `poddisruptionbudget`        | `pdb`         |
| `podpreset`                  |               |
| `pods`                       | `po`          |
| `podsecuritypolicies`        | `psp`         |
| `podtemplates`               |               |
| `replicasets`                | `rs`          |
| `replicationcontrollers`     | `rc`          |
| `resourcequotas`             | `quota`       |
| `rolebindings`               |               |
| `roles`                      |               |
| `secrets`                    |               |
| `serviceaccounts`            | `sa`          |
| `services`                   | `svc`         |
| `statefulsets`               |               |
| `storageclasses`             |               |

### 常用操作

##### `create`

使用以下一组示例来帮助您熟悉运行常用kubectl操作：

```shell
// 使用在example-service.yaml中的定义创建一个service.
$ kubectl create -f example-service.yaml

// 使用在example-controller.yaml中的定义创建一个replication controller.
$ kubectl create -f example-controller.yaml

// 使用在<directory>目录下的any .yaml, .yml, or .json文件创建对象.
$ kubectl create -f <directory>
```

##### `get`

`kubectl get` - 列出一个或更多资源.

~~~shell
// 使用文本格式列出所有的pods.
$ kubectl get pods

// 使用文本格式列出所有的信息，包含一些额外的信息(比如节点名称).
$ kubectl get pods -o wide

// 使用文本格式列出指定名称的replicationcontroller. 注: 你可以缩短 'replicationcontroller' 资源类型使用别名 'rc'.
$ kubectl get replicationcontroller <rc-name>

// 使用文本格式列出所有的rc,services.
$ kubectl get rc,services
`kubectl describe` - 显示一个或多个资源的详情状态.

```shell
// 显示指定节点名称的详情信息.
$ kubectl describe nodes <node-name>

// 显示指定pods名称的详情信息.
$ kubectl describe pods/<pod-name>

// 显示所有被名称为<rc-name>的replication controller管理的所有pods的详情信息.
// 请记住: 任何被replication controller的pod的名称前缀为replication controller的名称.
$ kubectl describe pods <rc-name>
~~~

##### `delete`

`kubectl delete` - 从文件, stdin,或者指定的label selectors, 名称,资源选择器, 或者资源去删除资源.

```shell
// 通过 pod.yaml 文件中的资源类型和名称删除一个pod.
$ kubectl delete -f pod.yaml

// 删除所有label名称为name=<label-name>的pods和services.
$ kubectl delete pods,services -l name=<label-name>

// 删除所有pods.
$ kubectl delete pods --all
```

##### `exec`

`kubectl exec` - 针对pod中的某个容器执行命令.

```shell
// 在名称为<pod-name>的pod允许 'date' 命令获得输出. 默认返回的是pod中第一个容器的终端.
$ kubectl exec <pod-name> date

// 在<pod-name>的pod中的<container-name>容器中运行'date'获取输出.
$ kubectl exec <pod-name> -c <container-name> date

// 从名称为<pod-name>的pod获取一个交互的终端和运行/bin/bash. 默认返回的是pod中第一个容器的终端.
$ kubectl exec -ti <pod-name> /bin/bash
```

##### `logs`

`kubectl logs` - 输出一个pod的容器日志.

```shell
// 从名称为<pod-name>的pod返回日志快照.
$ kubectl logs <pod-name>

// 从名称为<pod-name>的pod中获取日志流. 这个和Linux命令'tail -f'相似.
$ kubectl logs -f <pod-name>
```