## What is `minikube` and what is k8s?

### What is kubernetes？

Firstly, it is a new leader in distributed architecture based on container technology. Kubernetes (k8s) is Google's open source container cluster management system (Google inside: Borg). Implemented for containerized applications:

- Deployment and running
- Resource Scheduling
- Service discovery
- Dynamic scaling

Kubernetes is a complete distributed system support platform with complete cluster management capabilities, multiple levels of security protection and access mechanisms, multi-tenant application support capabilities, transparent service registration and discovery mechanisms, and built-in intelligent load balancer. Strong fault discovery and self-healing capabilities, service rolling upgrade and online capacity expansion, scalable resource automatic scheduling mechanism and multi-granular resource quota management capabilities.

### What is `minikube`

`minikube` is one single node k8s platform. user can learn how to use k8s on one machine that deployed `minikube`.

### What is `kubectl`

`kubectl` is a command line interface (cli) for running commands against the Kubernetes cluster and is a command line program that interacts with the Kubernetes API. You can operate the application on k8s to get the information of the cluster.

#### Grammar

Input the `kubectl` command on the termianl.

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

For this, the command, TYPE, NAME and FLAGS are:

- `command`: the operation on resources，such as `create`，`get`，`describe`，`delete`.

- `TYPE` define [type of resources](https://kubernetes.io/cn/docs/user-guide/kubectl-overview/#%E8%B5%84%E6%BA%90%E7%B1%BB%E5%9E%8B). Resource types are case sensitive and you can specify singular, plural or abbreviated forms. For example, the following command produces the same output:

  ```shell
  $ kubectl get pod pod1  
  $ kubectl get pods pod1 
  $ kubectl get po pod1
  ```

- `NAME`：Specify the name of the resource. The name is case sensitive. If the name is omitted, the details of all resources are displayed, such as`$ kubectl get pods`。

  - When performing operations on multiple resources, you can specify each resource by type and name, or specify one or more files:

  - Specify resources by type and name:
      * To group resources if they are all of the same type: `TYPE1 name1 name2 name<#>`.
      e.g.: `$ kubectl get pod example-pod1 example-pod2`
      
      * To specify multiple resource types separately:  `TYPE1/name1 TYPE1/name2 TYPE2/name3 TYPE<#>/name<#>`.
      e.g.: `$ kubectl get pod/example-pod1 replicationcontroller/example-rc1`

  Specify resources using one or more files: `-f file1 -f file2 -f file<#>` 
  e.g.：$ kubectl get pod -f ./pod.yaml

- flags：define marks. For example, you can use `-s` or `--serverflags` to specify the address and port of Kubernetes API server. 

  **Important**：The flag specified from the command line overrides the default value and any corresponding environment variables.

if you need help, run `kubectl help`。

#### Detials

##### command

for `kubectl [command]`，every usage can be found [here](https://kubernetes.io/docs/reference/kubectl/kubectl/).

##### type

| type of resoureces                     | alias      |
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

### Common operation

##### `create`

Use the following set of examples to help you get familiar with running common kubectl operations:

```shell
// Create a service using the definition in example-service.yaml.
$ kubectl create -f example-service.yaml

// Create a replication controller using the definition in example-controller.yaml.
$ kubectl create -f example-controller.yaml

// Create an object using the any .yaml, .yml, or .json file in the <directory> directory.
$ kubectl create -f <directory>
```

##### `get`

`kubectl get` - List one or more resources.

~~~shell
// List all pods in text format.
$ kubectl get pods

// List all the information in text format with some extra information (such as node name).
$ kubectl get pods -o wide

// Use the text format to list the replicationcontroller with the specified name. Note: You can shorten the 'replicationcontroller' resource type using the alias 'rc'.
$ kubectl get replicationcontroller <rc-name>

// List all rc, services using text format.
$ kubectl get rc,services
`kubectl describe` - displays the details of one or more resources.

```shell
// Display details of the specified node name.
$ kubectl describe nodes <node-name>

// Display the details of the specified pods name.
$ kubectl describe pods/<pod-name>

// Display details of all pods managed by the replication controller named <rc-name>.
// Remember: Any name of the pod of the replication controller is prefixed with the name of the replication controller.
$ kubectl describe pods <rc-name>
~~~

##### `delete`

`kubectl delete` - removes resources from files, stdin, or specified label selectors, names, resource selectors, or resources.

```shell
// Delete a pod by the resource type and name in the pod.yaml file.
$ kubectl delete -f pod.yaml

// Delete all pods and services whose label name is name=<label-name>.
$ kubectl delete pods,services -l name=<label-name>

// Remove all pods.
$ kubectl delete pods --all
```

##### `exec`

`kubectl exec` - executes a command against a container in the pod.

```shell
// The pod with the name <pod-name> allows the 'date' command to get the output. The default is to return the terminal of the first container in the pod.
$ kubectl exec <pod-name> date

// Run 'date' in the <container-name> container in the <pod-name> pod to get the output.
$ kubectl exec <pod-name> -c <container-name> date

// Get an interactive terminal from the pod named <pod-name> and run /bin/bash. The default is to return the terminal of the first container in the pod.
$ kubectl exec -ti <pod-name> /bin/bash
```

##### `logs`

`kubectl logs` - Outputs a pod container log.

```shell
// Return a log snapshot from the pod named <pod-name>.
$ kubectl logs <pod-name>

// Get the log stream from the pod named <pod-name>. This is similar to the Linux command 'tail -f'.
$ kubectl logs -f <pod-name>
```
