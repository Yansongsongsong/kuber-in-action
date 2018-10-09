# How to deploy one application and what is _deployment_ , _pod_ and etc?

## what is `kubectl run `

The `run` command creates a `deployment` based on the parameters specified, such as the image or replicas. 

## what is `Pod`, `deployment` and etc?

As of Section 1.3, there are many different concepts in our instructions, such as `Pod`, `service`, `deployment`, `replicationcontroller`, which are the resource types managed in kubernetes mentioned above.

In addition, we should also note that there are also `container` in the resource that are masked out of detail in kubernetes.

###  `node`

The host node in the kubernetes cluster is a physical concept. In the end, each task and command is executed by them. Node is the working machine of Kubernetes, which can be a virtual machine or a physical machine, depending on the installation in the cluster.

Each node is managed by the Master.

### `container`

In kubernetes, all tasks, the operation of the service, and the final destination are the containers of virtualization technology. Kubernetes' default container technology is docker, containerization technology in addition to docker and rocket technology, and docker has become the de facto standard.

The deployment of all our final applications is achieved by deploying a docker container.

### `Pod`

Pod is the smallest/simplest basic unit created or deployed by Kubernetes, and a Pod represents a process running on the cluster. We can think of Pod as the application container, which is the abstract concept of container in kubernetes, and the abstract unification of container. As mentioned above, there are many kinds of Pod runtime, the default is docker.

**useage**

The use of Pod in Kubernetes can be divided into two main ways:

- run one container on Pod

  The "one-container-per-Pod" mode is the most common usage of Kubernetes; in this case, you can think of Pods as a single-packaged container, but Kubernetes manages Pods directly rather than containers.

- run many containers that work together on Pod.

  Pods can encapsulate tightly coupled applications. They need to be composed of multiple containers that share resources between them. These containers can form a single internal service unit - one container shares files and another "sidecar" container to update these files. Pods manage the storage resources of these containers as an entity.

**Horizontal expansion**

Each Pod is a single instance of the running application. If you need to scale the application horizontally (for example, to run multiple instances), you should use multiple Pods, one for each instance. In Kubernetes, this is often referred to as Replication. Replication's Pod is usually created and managed by the Controller.

**提供资源**

- network

  Each Pod is assigned a separate IP address, and each container in the Pod shares a network namespace, including an IP address and a network port. Containers within the Pod can communicate with each other using localhost. When containers in a Pod communicate externally with the Pod, they must coordinate how to use shared network resources such as ports.

- storage

  Pods can specify a set of shared storage *volumes*. All containers in the pod can access the shared *volumes*, allowing these containers to share data. *volumes* is also used for data persistence in Pods in case one of the containers needs to be restarted to lose data. For more information on how Kubernetes implement shared storage in Pods, please refer to Volumes.

**controller**

We generally do not directly manage Pod, but control it through a controller. such as:

- Deployment
- StatefulSet
- DaemonSet
- Replica Sets
- Replication controller

### `Replication controller`

s an older version of the controller used to control the number of Pod instances, we are now recommended to use the Deployment configuration ReplicaSet (RS) method to control the number of copies.

We can do this by using the Replication controller.

- Replace, add, or remove Pod instances depending on the environment.
- Rescheduling (re-planning)
- expansion
- Rolling updates
- Multi-version tracking
- Use ReplicationControllers with associated Services

### `service`

Kubernetes Pods are life-cycled, they have their own IPs that can be discovered, but these IP addresses are not always stable and dependable because of their own possible creation and destruction.

Service is to solve this problem.

Kubernetes Service defines an abstraction: a logical grouping of Pods, a strategy that can access them - often referred to as microservices. This set of Pods can be accessed by the Service, usually through the Label Selector (see [2.3.3](#2.3.3 what happened when we input `kubectl run` to run one application?), see details).
**create service**

A Service is a REST object in Kubernetes, similar to a Pod. Like all REST objects, the Service definition can request a new instance of apiserver based on the POST method.

**defination and explaination**

```yaml
kind: Service # Define a set of Pod services
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp # the tag is `app=MyApp`
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376 # This group of pod services is exposed on port 9376 of cluster IP.
```

### `deployment`

Deployment provides declarative updates for Pod and Replica Sets (upgraded Replication Controllers). You only need to describe what the target state you want in Deployment, and the deployment controller will help you change the actual state of the Pod and ReplicaSet to your target state.

However, it's important to note that we should not manually manage Replica Sets created by Deployment.


#### **classical useages**

- Create an app

   Use Deployment to create a ReplicaSet. ReplicaSet creates a pod in the background. Check the startup status to see if it succeeds or fails.

- Update app

   Then, declare the new state of the Pod by updating the PodTemplateSpec field of the Deployment. This will create a new ReplicaSet, which will move the pod from the old ReplicaSet to the new ReplicaSet at a controlled rate.

- Version rollback

   If the current state is unstable, roll back to the previous deployment revision. The release of the deployment is updated for each rollback.

- Application expansion

   Expand Deployment to meet higher loads.

- Status monitoring and rescheduling

   - Suspend Deployment to apply multiple fixes for PodTemplateSpec and then resume online.
   - Judging whether the online line has been hanged according to the status of the Deployment.
   - Clear old unnecessary ReplicaSets.

#### **defination and explaination**

**yaml contents**

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
# Below is a Deployment example that creates a ReplicaSet to launch 3 nginx pods.
$ kubectl create -f https://kubernetes.io/docs/user-guide/nginx-deployment.yaml --record
deployment "nginx-deployment" created

# View the situation of deployment
# According to the .spec.replicas configuration in the deployment we need the number of repalica is 3
#结果 shows the number of repaicas for "current", "updated to latest" and "available"
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s

# displayReplica Set
# ReplicaSet's name is always  <Deployment's name>-<pod hashOfTemplate>。
# The following Replica Set will ensure that there are always 3 nginx pods present.
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-2035384211   3         3         0       18s

# display the created Pods
# The pod-template-hash label in the pod label is not user-specified. It is automatically added by the deployment controller for the Pod.
# The purpose is to prevent duplicates of the pod name of the deployed child ReplicaSet
$ kubectl get pods --show-labels
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-2035384211-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
nginx-deployment-2035384211-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=2035384211
```

**Upate**

```shell
# We now want the nginx pod to use the nginx:1.9.1 image instead of the original nginx:1.7.9 image.
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated

# We can use the edit command to edit the Deployment
# kubectl edit deployment/nginx-deployment

# display the status of rollout
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out

# Rollout After successful, get Deployment
# UP-TO-DATE is 3 means it has been updated to the latest
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           36s

# View replica set
# You can see that Deployment has updated the Pod by creating a new ReplicaSet and expanding the three replicas, while shrinking the original ReplicaSet to 0 replicas.
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         0       6s
nginx-deployment-2035384211   0         0         0       36s

# Execute get pods and we will only see the current new pod
# The deployment update strategy is that the old Pod will not be killed until the new Pod is created. This ensures that there are at least 2 Pods available and a maximum of 4 Pods.
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
```

In addition to this, there are strategies such as rollback, which will not be talked.

## what happened when we input `kubectl run` to run one application?

![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174216.png)

Looking at the diagram you can spot the following components, I used icons to represent Service & Label:

- Pod
- Container（容器）
- Label(![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174236.png))（标签）
- Replication Controller（复制控制器）
- Service（![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174247.png)）（服务）
- Node（节点）
- Kubernetes Master（Kubernetes主节点）

### Pod

Pods (green boxes) are scheduled to Nodes and contain a group of co-located Containers and Volumes. Containers in the same Pod share the same network namespace and can communicate with each other using localhost. Pods are considered to be ephemeral rather than durable entities. You might be asking yourself a few questions:

- If Pods are ephemeral how can I persist my container data across container restarts? Well, Kubernetes supports the concept of Volumes so you can use a Volume type that is persistent.
- Do I create Pods manually, what if I want to create a few copies of the same container do I have to create each one individually? You can create individual Pods manually, but you can use a Replication Controller to rollout multiple copies using a Pod template, which will be explained below.
- If Pods are ephemeral and their IP address might change if they get restarted how can I reliability reference my backend container from a frontend container? In this case you will use a Service as explained below.

### Label

As you can see from the diagram, some of the Pods have labels(![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/20180917174236.png)).  A Label is a key/value pair attached to Pods and convey user-defined attributes. For example you might create a `tier` and an `app` tags to tag your containers by applying the Labels (`tier=frontend, app=myapp`) to your frontend Pods and Labels (`tier=backend, app=myapp`) to backend Pods. You can then use Selectors to select Pods with particular Labels and apply Services or Replication Controllers to them.

### Replication Controller

Do I create Pods manually, what if I want to create a few copies of the same Pod, do I have to create each one individually, can I group Pods into logical groups?

Replication Controllers ensure the specified number of Pod “replicas” are running at any one time. If you created a Replication Controller for a Pod and specified 3 replicas, it will create 3 Pods and will continuously monitor them. If one Pod dies then the Replication Controller will replace it to maintain a total count of 3. This is illustrated in the animated image below:

![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/gif.gif)

If the Pod that died comes back then you have 4 Pods, consequently the Replication Controller will terminate one so the total count is 3. If you change the number of replicas to 5 on the fly, the Replication Controller will immediately start 2 new Pods so the total count is 5. You can also scale down Pods this way, a handy feature performing rolling updates.

When creating a Replication Controller you need to specify two things:

1. Pod Template: the template that will be used to create the Pods replicas.

2. Labels: the labels for the Pods that this Replication Controller should monitor.

So now you have created a few replicas of a Pod, how do you load balance between them? Enter Services.

### Service

If Pods are ephemeral and their IP address might change if they get restarted how can I reliably reference my backend container from a frontend container?

Service is an abstraction that defines a set of Pods and a policy to access them. Services find their group of Pods using Labels. Because Services are abstraction you don’t usually see them in diagrams which makes the concept hard to understand.

Now, imagine you have 2 backend Pods and you defined a backend Service named ‘backend-service’ with label selector (`tier=backend, app=myapp`). Service backend-service will facilitate two key things:

- A cluster-local DNS entry will be created for the Service so your frontend Pod only need to do a DNS lookup for hostname ‘backend-service’ this will resolve to a stable IP address that your frontend application can use.
- So now your frontend has got an IP address for the backend-service, but which one of the 2 backend Pods will it access? The Service will provide transparent load balancing between the 2 backend Pods and forward the request to any one of them (see the animated diagram below). This is done by using a proxy (kube-proxy) that runs on each Node.

![](https://raw.githubusercontent.com/Yansongsongsong/PicsHub/archive/service.gif)
