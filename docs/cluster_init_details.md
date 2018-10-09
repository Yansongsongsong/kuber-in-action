# What is `kubeadm` and what is the overall structure of k8s?

## What is kubeadm?

`Kubeadm` provides two commands, `kubeadm init` and `kubeadm join`, to build one k8s cluster which is the best practice.

## The architecture of the whole k8s

We have used two `cli` tools, `kubectl` and `kubeadm`, which are parts of k8s. K8s consists of many parts and has many service. The architecture of k8s embodies many of the best practices for distributed system design, such as there are no direct dependancies on components; k8s focuses on the service choreography and leaves interface, such as storage, network and etc, which can be implemented with plugin by users.

### Structure of k8s

k8s runs on two types of nodes, `master` and `minion`. The former is used for managing nodes and the latter for supporting container running.

### Necessary components

The procedures or services runs on MASTER:

- `APIServer`

  It is the controller entry for the whole system and provides the interface with REST API.

  It is mainly responsible for exposing kubernetes API. No matter how you operate the resources in kubernetes, using `kubectl` command or HTTP calls, u have to use the interfaces of apiserver to use the resources.

  - Handle the requests from different users
  - The Cooperation for different resources

- `scheduler`

  The responsibility of scheduler is gaving one scheduling for the requests that triggered by kube-controller-manager in accordance with some factors, such as Request specifications, scheduling constraints, overall resources, and passing the jobs to kubelet components of the worker node to wait for execution.

- `controller manager`

  The controllers are used for executing the background tasks and displaying the status for worker nodes, Pods and services.

  Here are one collerction for different controller, which is mainly responsible for managements of resources. 

  - Node Controller
  - Replication Controller
  - Endpoints Controller
  - Namespace Controller
  - Serviceaccounts Controller

- `etcd`

  Etcd is one high performance Key-Value storage system for sharing configuration, which is distributed and has Strong consistency.

  Etcd stored all data that needs to be persisted in k8s.

The procedures or services runs on MINION:

- `kubelet`

  It is running on every computing node. It accepts the Pods tasks dispatched to itself, manages the containers and gives the status of containers back to kube-apiserver.

- `kube-proxy`

  The kube-proxy runs on each compute node and is responsible for the Pod network proxy. Timing from the etcd to get the service information to do the corresponding strategy. 

Others

- `kubectl`

  Client command line tool. It is responsible for formatting the accepted commands and sending them to the kube-apiserver as an operational entry for the entire system.

- `kubeadm`

  An official command line tool for building and installing the above components

- `DNS `

  An optional DNS service for creating DNS records for each Service object so that all Pods can access the service through DNS.

- Dashboard

  The official UI interface of the kubernetes cluster provides some basic viewing and simple operations.

![kubernetes structure](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917163110.png)

## k8s useage

Kubernetes provides comprehensive management tools covering all aspects including development, deployment testing, and operation and maintenance monitoring. Improve the convenience of large-scale container cluster management.

1. Automated container deployment and replication
2. Expand or shrink the container size at any time
3. Organize containers into groups and provide load balancing between containers
4. Easily upgrade new versions of the application container
5. Provide container flexibility and replace it if the container fails