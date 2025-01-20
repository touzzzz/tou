# 1、k8s有哪些核心组件？各自作用？

k8s官网文档：[Kubernetes 文档 | Kubernetes](https://kubernetes.io/zh-cn/docs/home/)

里面内容很详细，多看看

## 控制平面组件

### kube-apiserver

API 服务器是 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件负责公开了 Kubernetes API，负责处理接受请求的工作。 API 服务器是 Kubernetes 控制平面的前端。

### etcd

一致且高可用的键值存储，用作 Kubernetes 所有集群数据的后台数据库。

### kube-scheduler

`kube-scheduler` 是[控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 负责监视新创建的、未指定运行[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)的 [Pods](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)， 并选择节点来让 Pod 在上面运行。

### kube-controller-manager

[kube-controller-manager](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/) 是[控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 负责运行[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)进程。

## 节点组件

### kubelet

`kubelet` 会在集群中每个[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上运行。 它保证[容器（containers）](https://kubernetes.io/zh-cn/docs/concepts/containers/)都运行在 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 中。

### kube-proxy

[kube-proxy](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-proxy/) 是集群中每个[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上所运行的网络代理， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/) 概念的一部分

### 容器运行时

这个基础组件使 Kubernetes 能够有效运行容器。 它负责管理 Kubernetes 环境中容器的执行和生命周期。

容器运行时就是容器运行的环境

k8s -1.24版本之后就不支持dockershim了

![image-20241130145524027](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130145524027.png)

k8s 1.24版本之后还想用docker，该怎么办？

参考链接：[Kubernetes1.24还想使用Docker怎么办？ - Layzer - 博客园](https://www.cnblogs.com/layzer/articles/kubernetes_1_24_for_docker.html)

参考链接：[从 dockershim 迁移 | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/migrating-from-dockershim/)

# 2、什么是pod？什么是namespace?什么是label？

## Pod

**Pod** 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。**Pod**（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） [容器](https://kubernetes.io/zh-cn/docs/concepts/containers/)； 这些容器共享存储、网络、以及怎样运行这些容器的规约。

Kubernetes 集群中的 Pod 主要有两种用法：

- **运行单个容器的 Pod**。"每个 Pod 一个容器"模型是最常见的 Kubernetes 用例； 在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。

- **运行多个协同工作的容器的 Pod**。 Pod 可以封装由紧密耦合且需要共享资源的[多个并置容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)组成的应用。 这些位于同一位置的容器构成一个内聚单元。

  将多个并置、同管的容器组织到一个 Pod 中是一种相对高级的使用场景。 只有在一些场景中，容器之间紧密关联时你才应该使用这种模式。

下面是一个 Pod 示例，它由一个运行镜像 `nginx:1.14.2` 的容器组成。

[`pods/simple-pod.yaml` ](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/pods/simple-pod.yaml)![复制 pods/simple-pod.yaml 到剪贴板](https://kubernetes.io/images/copycode.svg)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

要创建上面显示的 Pod，请运行以下命令：

```shell
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

Pod 通常不是直接创建的，而是使用工作负载资源创建的，通常你不需要直接创建 Pod，甚至单实例 Pod。相反，你会使用诸如 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 或 [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 这类工作负载资源来创建 Pod

## Deployments

Deployment 用于管理运行一个应用负载的一组 Pod，通常适用于不保持状态的负载。

下面是一个 Deployment 示例。其中创建了一个 ReplicaSet，负责启动三个 `nginx` Pod：

[`controllers/nginx-deployment.yaml` ](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/controllers/nginx-deployment.yaml)![复制 controllers/nginx-deployment.yaml 到剪贴板](https://kubernetes.io/images/copycode.svg)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

通过运行以下命令创建 Deployment ：

```shell
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

运行 `kubectl get deployments` 检查 Deployment 是否已创建



## Namespace

命名空间，Namespace是一种用于将集群划分为多个虚拟集群的方法。它允许将不同的资源组织到不同的逻辑分区中，从而实现资源隔离、多租户支持和访问控制，**总的来说就是起到一个隔离作用**

### 1. **Namespace 的作用**

- **资源隔离：** Namespace 使得不同的应用或团队可以在同一个 Kubernetes 集群中工作，而互不干扰。比如，开发团队可以在 `dev` 命名空间中创建资源，生产团队可以在 `prod` 命名空间中创建资源。
- **组织资源：** 在大型集群中，命名空间帮助组织和管理大量资源。它使得不同的资源集群在逻辑上分开，有助于避免命名冲突和提高资源的可管理性。
- **权限控制：** 通过设置 Kubernetes 的访问控制（RBAC），可以控制不同命名空间中的资源访问权限，确保不同团队或用户仅能访问和操作他们有权限的资源。

### 2. **默认命名空间**

Kubernetes 集群默认有一些命名空间，例如：

- **default:** 用于没有指定命名空间的资源。默认情况下，资源都会创建在 `default` 命名空间中。
- **kube-system:** 系统级资源所在的命名空间，通常包含 Kubernetes 控制平面的核心组件，比如 kube-dns、kube-proxy 等。
- **kube-public:** 用于公共资源的命名空间，所有用户都可以访问。一般用于存放集群级别的公共资源。

### 3. **如何使用 Namespace**

#### 创建命名空间

你可以通过 YAML 文件创建一个新的命名空间：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

#### 创建资源在指定命名空间

你可以在指定命名空间下创建资源，比如 Pod：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: dev  # 指定命名空间为 dev
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

#### 查看指定命名空间的资源

查看某个命名空间下的所有 Pod：

```bash
kubectl get pods -n dev
```

#### 切换命名空间

如果你不想每次都指定命名空间，可以使用 `kubectl` 设置默认命名空间：

```bash
kubectl config set-context --current --namespace=dev
```

之后，你可以不指定命名空间就直接访问 `dev` 命名空间中的资源。

### 4. **命名空间和资源的关系**

- **命名空间是用于逻辑隔离的**：虽然资源（如 Pods、Services 等）是物理存在的，但它们可以根据命名空间来进行逻辑分组和隔离。
- **资源在同一命名空间内共享：** 例如，Pod 和 Service 在同一命名空间下可以通过 Service 名称互相访问。



## Label

Labels 是 Kubernetes 中用于标识对象的关键/值对，常用于为对象（如 Pod、Service、Deployment 等）打标签。通过标签，你可以对不同的对象进行分类、筛选、管理或查找。

### 1. **标签是什么**

标签是一种键值对，键和值都可以是用户自定义的字符串，用于为对象加上元数据。举个例子，一个 Pod 可以有一个标签 `app=nginx`，表示该 Pod 运行的是 nginx 应用。

### 2. **标签的用途**

- **筛选和选择：** Labels 常用于选择资源，例如，基于标签来选择一组 Pod，或者在 Service 中通过标签选择哪些 Pod 被暴露。
- **分组：** 可以用标签将不同的资源进行分组，方便管理和排查问题。

### 3. **实例**

例如，定义一个 Pod 时，可以给它加上标签：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx  # 给Pod加标签 app=nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

### 4. **如何使用标签选择资源**

使用 `kubectl` 命令时，可以根据标签选择资源：

```bash
kubectl get pods -l app=nginx
```

这条命令会返回所有带有 `app=nginx` 标签的 Pod。



# 3、k8s有哪些常见的资源对象

Kubernetes (K8s) 提供了多种资源对象来管理集群中的应用和服务。下面是一些最常见的 Kubernetes 资源对象及其用途：

### 1. **Pod**

- **定义**：Pod 是 Kubernetes 中的最小部署单元，一个 Pod 可以包含一个或多个容器，这些容器共享同一网络命名空间和存储卷。
- **用途**：Pod 是 Kubernetes 中运行应用的基础单元，通常每个 Pod 运行一个应用实例。
- **常见操作**：创建、查看、更新、删除 Pod。

### 2. **Deployment**

- **定义**：Deployment 是一种控制器，它管理多个副本的 Pod。它可以确保指定数量的 Pod 副本在运行并且保持一致性。
- **用途**：用于声明应用的期望状态，例如副本数、镜像版本等，Kubernetes 会自动管理 Pod 的生命周期，确保副本数不低于预期。
- **常见操作**：创建、查看、更新、删除 Deployment。

### 3. **Service**

- **定义**：Service 是一种抽象，它定义了如何访问运行在 Pods 上的应用程序。Service 提供了一个稳定的访问入口，即使 Pod 被删除或重新调度。
- **用途**：用来暴露应用程序，可以设置为 `ClusterIP`（集群内部访问）、`NodePort`（通过节点端口访问）、`LoadBalancer`（云服务商提供的负载均衡器）或 `ExternalName`（通过 DNS 名称访问外部服务）。
- **常见操作**：创建、查看、更新、删除 Service。

### 4. **ReplicaSet**

- **定义**：ReplicaSet 确保指定数量的 Pod 副本在集群中始终存在。
- **用途**：它与 Deployment 一起工作，确保 Pod 副本数始终符合预期。Deployment 会控制 ReplicaSet 的创建、更新和管理。
- **常见操作**：创建、查看、更新、删除 ReplicaSet。

### 5. **StatefulSet**

- **定义**：StatefulSet 是用于管理有状态应用的控制器，与 Deployment 类似，但提供了有状态的应用需求，例如持久化存储、稳定的网络标识符等。
- **用途**：用于管理数据库、缓存等有状态应用。
- **常见操作**：创建、查看、更新、删除 StatefulSet。

### 6. **DaemonSet**

- **定义**：DaemonSet 确保集群中的每个节点上都运行一个 Pod 实例。
- **用途**：用于运行一些需要在每个节点上都存在的应用，例如日志收集、监控代理等。
- **常见操作**：创建、查看、更新、删除 DaemonSet。

### 7. **Job**

- **定义**：Job 是一个用于运行一次性任务的控制器。它确保任务按期望的次数成功完成。
- **用途**：用于运行批处理任务，Kubernetes 会保证任务执行的可靠性。
- **常见操作**：创建、查看、更新、删除 Job。

### 8. **CronJob**

- **定义**：CronJob 是基于时间的 Job，用于按照预定时间表定期执行任务。
- **用途**：用于定期运行任务，类似 Linux 中的 cron。
- **常见操作**：创建、查看、更新、删除 CronJob。

### 9. **ConfigMap**

- **定义**：ConfigMap 用于存储非机密的配置信息，供应用程序使用。
- **用途**：可以将配置数据分离出来，供 Pod 中的容器作为环境变量、命令行参数或配置文件使用。
- **常见操作**：创建、查看、更新、删除 ConfigMap。

### 10. **Secret**

- **定义**：Secret 用于存储敏感数据（如密码、密钥等），它与 ConfigMap 类似，但它以加密的形式存储数据。
- **用途**：用于存储应用所需的敏感信息，例如数据库密码、API 密钥等。
- **常见操作**：创建、查看、更新、删除 Secret。

### 11. **Ingress**

- **定义**：Ingress 是 Kubernetes 中的 API 资源，管理外部访问服务的方式，暴露service服务到外网，通常是 HTTP 或 HTTPS 流量。
- **用途**：提供 HTTP 和 HTTPS 路由规则，支持负载均衡、SSL/TLS 终结等功能。
- **常见操作**：创建、查看、更新、删除 Ingress。

### 12. **Namespace**

- **定义**：Namespace 用于将 Kubernetes 资源分组到多个虚拟集群中。每个 Namespace 内的资源名称都是唯一的。
- **用途**：用于逻辑上隔离不同的工作负载、应用或团队。
- **常见操作**：创建、查看、更新、删除 Namespace。

### 13. **PersistentVolume (PV)**

- **定义**：PersistentVolume 是一个集群中的存储资源，可以是 NFS、iSCSI、云存储等。
- **用途**：用于管理持久化存储，供 Pod 使用。PV 是与存储类型和生命周期无关的抽象层。
- **常见操作**：创建、查看、更新、删除 PersistentVolume。

### 14. **PersistentVolumeClaim (PVC)**

- **定义**：PersistentVolumeClaim 是对 PV 资源的请求，指定所需存储的大小和访问模式。
- **用途**：Pod 通过 PVC 申请存储，PVC 会绑定到适合的 PV 上。
- **常见操作**：创建、查看、更新、删除 PersistentVolumeClaim。

### 15. **NetworkPolicy**

- **定义**：NetworkPolicy 用于定义 Pod 之间的网络通信规则。
- **用途**：限制不同 Pod 之间或外部网络对 Pod 的访问，提高网络安全性。
- **常见操作**：创建、查看、更新、删除 NetworkPolicy。

### 16. **ResourceQuota**

- **定义**：ResourceQuota 用于限制 Kubernetes 集群或 Namespace 中使用的资源（如 CPU、内存等）。
- **用途**：为不同的团队或应用设置资源配额，以避免资源耗尽。
- **常见操作**：创建、查看、更新、删除 ResourceQuota。

### 17. **HorizontalPodAutoscaler (HPA)**

- **定义**：HPA 用于根据应用的负载自动调整 Pod 的副本数。
- **用途**：根据 CPU 使用率或其他指标动态扩缩容。
- **常见操作**：创建、查看、更新、删除 HorizontalPodAutoscaler。

### 18. **Affinity 和 Toleration**

- **定义**：用于控制 Pod 的调度策略。`Affinity` 是对 Pod 调度规则的设置，`Toleration` 是允许 Pod 调度到有 taint 的节点。
- **用途**：控制 Pod 运行在哪些节点上，或避免某些节点。
- **常见操作**：在 Pod 定义中设置调度规则。



# 4、pod的作用是什么？

### Pod 的作用和概念

在 Kubernetes 中，**Pod** 是一个最基本的工作负载单元。Pod 是 Kubernetes 中执行应用程序的最小和最简单的部署单位。Pod 中可以运行一个或多个容器，这些容器共享同一网络命名空间、存储卷和资源配置。Pod 使得 Kubernetes 能够实现容器的高效管理、部署和扩展。

### Pod 的作用：

1. **容器管理单元**：
   - Pod 是容器的封装器，它可以包含一个或多个容器，并管理这些容器的生命周期、资源分配和调度等。
   - 即使 Pod 中包含多个容器，这些容器也共享网络和存储，因此它们可以轻松地相互通信并访问共享数据。
2. **共享网络命名空间**：
   - Pod 内的所有容器共享同一个 IP 地址、端口空间以及网络接口。
   - 容器间的通信可以通过本地 `localhost` 来进行，不需要跨容器进行复杂的网络通信配置。
   - Pod 会分配一个唯一的 IP 地址，从而使得容器之间的通信变得简便。
3. **存储卷共享**：
   - Pod 内的多个容器可以共享同一个存储卷。即使容器被重新启动，存储数据仍然会保留在存储卷中。
   - 这对于需要共享数据的应用（例如数据库、日志服务等）非常重要。
4. **调度和扩展**：
   - Kubernetes 的调度器会根据定义的策略把 Pod 安排到集群中的某个节点上执行。
   - 通过定义副本数，Kubernetes 可以自动扩展（或收缩）Pod，保证应用在不同节点间高可用。
5. **生命周期管理**：
   - Pod 会根据其定义的副本数和更新策略进行生命周期管理。例如，通过 Deployment 或 StatefulSet 管理的 Pod，会自动进行滚动升级或回滚。
   - Kubernetes 会确保 Pod 总是按照期望的数量运行，并且自动重启故障的 Pod。
6. **支持有状态和无状态应用**：
   - Pod 既可以用于无状态应用（如 Web 应用、API 服务等），也可以用于有状态应用（如数据库、缓存等），通过 StatefulSet 配合持久化存储来保证有状态服务的稳定性。
7. **控制器支持**：
   - Kubernetes 通过 Deployment、ReplicaSet、StatefulSet 等控制器来管理 Pod 的副本和状态，从而自动处理 Pod 的创建、删除、更新和扩展等任务。
8. **Pod 与服务（Service）结合**：
   - Pod 通常不会直接暴露给外部访问。相反，它们会通过 Kubernetes 的 `Service` 对象对外提供访问。`Service` 会自动选择一组 Pod 来处理请求，并负载均衡流量。

### Pod 的使用场景：

- **单容器应用**：一个 Pod 可以包含一个容器，用来运行单个应用实例。
- **多容器应用**：一个 Pod 可以包含多个容器，这些容器可以共享存储和网络资源。适用于具有紧密耦合关系的多个容器，通常用于微服务架构中。
  - 比如，Nginx 和一个后端应用程序容器可以放在同一个 Pod 内，并共享存储卷用于持久化数据。
- **批处理任务**：Pod 可以用于批处理任务，例如短暂运行的 Job。
- **系统管理和监控**：如日志收集、监控、代理等容器可以放在同一个 Pod 内。

### Pod 与其他 Kubernetes 资源的关系：

- **Pod 和 ReplicaSet**：ReplicaSet 用于确保一定数量的 Pod 副本始终在集群中运行。它通过监视 Pod 的状态，并在需要时创建或删除 Pod 来保持所需的副本数。
- **Pod 和 Deployment**：Deployment 是管理应用副本和更新的一种控制器，Deployment 使用 ReplicaSet 来实现 Pod 的扩展和管理。
- **Pod 和 StatefulSet**：StatefulSet 用于管理有状态应用，其创建和管理的 Pod 保持持久化存储和稳定的网络标识符。
- **Pod 和 Service**：Service 用于提供 Pod 的访问入口，通过负载均衡请求流量到 Pod 的多个副本，隐藏了 Pod 的变化和 IP 地址的变动。



# 5、简述pod的生命周期？

Pod 的生命周期可以分为几个阶段，从创建到删除。每个阶段的状态和行为有其特定的含义。以下是 Pod 生命周期的简要概述：

### 1. **Pending（待定）**

- **定义**：Pod 被创建并且调度到某个节点，但容器尚未开始运行。

- 可能原因

  ：

  - 节点资源不足，导致调度延迟。
  - 容器镜像正在拉取。
  - 存储卷尚未挂载。

### 2. **Running（运行中）**

- **定义**：Pod 中的容器至少有一个已经启动并在运行。

- 状态

  ：

  - 如果所有容器都在运行，Pod 状态为 Running。
  - 如果有容器正在运行，但其他容器处于启动或等待状态，Pod 仍然会处于 Running 状态。

### 3. **Succeeded（成功）**

- **定义**：Pod 中的所有容器都已经成功终止，并且没有失败或错误,不会重启。
- **常见于**：Pod 中的容器执行完一次性的任务，如批处理工作。

### 4. **Failed（失败）**

- **定义**：Pod 中的容器之一或多个以非零状态退出，表示容器执行失败。

- 常见原因

  ：

  - 应用崩溃或退出错误。
  - 配置问题（例如，缺少环境变量或资源问题）。

### 5. **Unknown（未知）**

- **定义**：由于某些原因，Kubernetes 无法确定 Pod 的状态。

- 常见原因

  ：

  - 集群控制平面或节点出现故障，无法获取 Pod 状态。

# 6.容器的状态？



**Waiting （等待）** 

如果容器并不处在 Running 或 Terminated 状态之一，它就处在 Waiting 状态。 处于 Waiting 状态的容器仍在运行它完成启动所需要的操作：例如，从某个容器镜像 仓库拉取容器镜像，或者向容器应用 Secret 数据等等。

**Running（运行中）** 

Running 状态表明容器正在执行状态并且没有问题发生。 如果配置了 postStart 回调，那么该回调已经执行且已完成。 如果你使用 kubectl来查询包含 Running 状态的容器的 Pod 时，你也会看到 关于容器进入 Running 状态的信息。

**Terminated（已终止）** 

处于 Terminated 状态的容器已经开始执行并且或者正常结束或者因为某些原因失败。 如果你使用 kubectl 来查询包含 Terminated 状态的容器的 Pod 时，你会看到 容器进入此状态的原因、退出代码以及容器执行期间的起止时间

# 7、pod的健康检查有哪些方式？

**① liveness probes：存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，k8s会重启容器。**

**② readiness probes：就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，k8s不会转发流量。**

livenessProbe 决定是否重启容器，readinessProbe 决定是否将请求转发给容器。

上面两种探针目前均支持三种探测方式：

**exec**

在容器内执行指定命令。如果命令退出时返回码为 0 则认为诊断成功。

**httpGet**

对容器的 IP 地址上指定端口和路径执行 HTTP GET 请求。如果响应的状态码大于等于 200 且小于 400，则诊断被认为是成功的。

**tcpSocket**

对容器的 IP 地址上的指定端口执行 TCP 检查。如果端口打开，则诊断被认为是成功的。 如果远程系统（容器）在打开连接后立即将其关闭，这算作是健康的。



明白了，下面是每种探测方式（`exec`、`httpGet`、`tcpSocket`）在 `livenessProbe` 和 `readinessProbe` 中的完整示例，每种探测方式都用一个单独的实例。

### 1. **使用 `exec` 进行 Liveness Probe 和 Readiness Probe 探测**

#### Liveness Probe（执行命令检查容器是否活着）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-exec-liveness
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - name: nginx-port
          containerPort: 80
      # Liveness Probe：通过执行命令来判断容器是否健康
      livenessProbe:
        exec:
          command:
            - "/bin/cat"
            - "/tmp/1.txt"  # 检查容器内的 /tmp/1.txt 文件是否存在
        initialDelaySeconds: 5  # 启动后等待5秒开始探测
        periodSeconds: 10  # 每隔10秒进行一次探测
```

#### Readiness Probe（执行命令检查容器是否准备好）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-exec-readiness
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - name: nginx-port
          containerPort: 80
      # Readiness Probe：通过执行命令来判断容器是否准备好接收流量
      readinessProbe:
        exec:
          command:
            - "/bin/cat"
            - "/tmp/2.txt"  # 检查容器内的 /tmp/2.txt 文件是否存在
        initialDelaySeconds: 5  # 启动后等待5秒开始探测
        periodSeconds: 10  # 每隔10秒进行一次探测
```

### 2. **使用 `httpGet` 进行 Liveness Probe 和 Readiness Probe 探测**

#### Liveness Probe（通过 HTTP 请求检查容器是否健康）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-http-liveness
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - name: nginx-port
          containerPort: 80
      # Liveness Probe：通过 HTTP 请求检测容器是否健康
      livenessProbe:
        httpGet:
          path: /healthz  # 健康检查路径
          port: 80  # 健康检查端口
        initialDelaySeconds: 5  # 启动后等待5秒开始探测
        periodSeconds: 10  # 每隔10秒进行一次探测
```

#### Readiness Probe（通过 HTTP 请求检查容器是否准备好）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-http-readiness
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - name: nginx-port
          containerPort: 80
      # Readiness Probe：通过 HTTP 请求检查容器是否准备好接收流量
      readinessProbe:
        httpGet:
          path: /readiness  # 检查容器提供的 /readiness 路径
          port: 80  # 端口为 80
        initialDelaySeconds: 5  # 启动后等待5秒开始探测
        periodSeconds: 10  # 每隔10秒进行一次探测
```

### 3. **使用 `tcpSocket` 进行 Liveness Probe 和 Readiness Probe 探测**

#### Liveness Probe（通过 TCP 套接字检测容器是否活着）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-tcp-liveness
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - name: nginx-port
          containerPort: 80
      # Liveness Probe：通过 TCP 连接检查容器是否活着
      livenessProbe:
        tcpSocket:
          port: 80  # 通过端口 80 检查 TCP 连接
        initialDelaySeconds: 5  # 启动后等待5秒开始探测
        periodSeconds: 10  # 每隔10秒进行一次探测
```

#### Readiness Probe（通过 TCP 套接字检测容器是否准备好）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-tcp-readiness
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - name: nginx-port
          containerPort: 80
      # Readiness Probe：通过 TCP 连接检查容器是否准备好接收流量
      readinessProbe:
        tcpSocket:
          port: 80  # 通过端口 80 检查 TCP 连接
        initialDelaySeconds: 5  # 启动后等待5秒开始探测
        periodSeconds: 10  # 每隔10秒进行一次探测
```



# 8、使用kubeadm部署k8s集群



## Kubernetes之快速部署 K8s 集群 v1.28.0（使用docker）

**环境说明：**这次我们依旧想用docker命令，要另外下载组件cri-dockerd

| 主机名      | 私网地址       | 主机配置                | 备注     |
| ----------- | -------------- | ----------------------- | -------- |
| k8s-master  | 192.168.100.13 | 2核，2GiB，系统盘 40GiB | 控制节点 |
| k8s-worker1 | 192.168.100.18 | 2核，2GiB，系统盘 40GiB | 工作节点 |
| k8s-worker2 | 192.168.100.19 | 2核，2GiB，系统盘 50GiB | 工作节点 |

### 一、系统配置（所有节点执行）

#### 1.1关闭防火墙及相关配置

```bash
# 关闭和禁用防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux，selinux 是 Linux 系统下的一个安全服务，需要关闭
setenforce 0  # 临时
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久

# 关闭 Linux 的 swap 分区，提升 k8s 的性能
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
```

#### 1.2修改主机名

```bash
# 在 192.168.100.13 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-master

# 在 192.168.100.18 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-worker1

# 在 192.168.100.19 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-worker2
```

#### 1.3主机名DNS解析

```bash
 # 添加各个节点的解析，IP 地址需要替换为你自己服务器的内网 IP 地址。
cat >> /etc/hosts << EOF
192.168.100.13 k8s-master
192.168.100.18 k8s-worker1
192.168.100.19 k8s-worker2
EOF
```

#### 1.4时间同步

```bash
# 安装ntp服务
yum install ntpdate -y

# 安装完成后使用阿里云的时间服务同步时间 
ntpdate ntp1.aliyun.com
```

#### 1.5配置网络

为了让 K8s 能够转发网络流量，需要修改 iptables 的配置。执行以下命令快速编辑 “etc/sysctl.d/k8s.conf” 配置文件，增加 iptables 的配置。

```bash
# 修改 Linux 内核参数，添加网桥过滤和地址转发功能
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 使配置生效
sudo sysctl --system
```

#### 1.6重启服务器

运行 “reboot” 命令，重启服务器，让以上配置生效。

```shell
reboot
```

### 二、安装软件（所有节点执行）

#### 2.1安装docker

```bash
# 通过 wget 命令获取 docker 软件仓库信息
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 安装 docker-ce
yum -y install docker-ce

# 开启 docker 服务
systemctl enable docker && systemctl start docker
```

Docker 安装完成之后，配置阿里云提供的镜像库来加速镜像下载。

docker镜像加速持续更新：https://xuanyuan.me/blog/archives/1154?from=tencent

镜像加速很关键，会影响到后面网络插件的镜像拉取

```bash
cat <<-EOF > /etc/docker/daemon.json 
{
  "registry-mirrors": [
  	"https://docker.linkedbus.com",
  	"https://docker.xuanyuan.me"
  	]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

#### 2.2安装cri-dockerd

cri-dockerd 用于为 Docker 提供一个能够支持 K8s 容器运行时标准的工具，从而能够让 Docker 作为 K8s 容器引擎。通过 wget 下载 cri-dockerd 安装包，然后通过 rpm 命令进行安装。

```bash
# 通过 wget 命令获取 cri-dockerd软件
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.12/cri-dockerd-0.3.12-3.el7.x86_64.rpm

# 通过 rpm 命令执行安装包
rpm -ivh cri-dockerd-0.3.12-3.el7.x86_64.rpm
```

安装完成后修改配置文件（/usr/lib/systemd/system/cri-docker.service），在 “ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd://” 这一行增加 

“–pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9”。

修改如下

```bash
# 打开 cri-docker.service 配置文件
vi /usr/lib/systemd/system/cri-docker.service

# 修改对应的配置项
ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd:// --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9
```

配置文件修改后，重新加载配置并开启 cri-dockerd 服务。

```bash
# 加载配置并开启服务
systemctl daemon-reload
systemctl enable cri-docker && systemctl start cri-docker
```

#### 2.3添加国内的k8s 的yum源

编辑 “/etc/yum.repos.d/kubernetes.repo” 配置文件，添加阿里云的镜像库。

```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

#### 2.4安装kubeadm、kubelet、kubectl

所有K8S服务器上都需要安装 kubeadm、kubelet 和 kubectl 这三个工具。

- kubeadm 用来初始化 K8s 集群；
- kubelet 是每个节点的 K8s 管理员；
- kubectl 是 K8s 的命令行交互工具。

由于版本更新比较快，这里指定安装的较新的版本`1.28.0`。

```bash
# 指定了安装的版本是 1.28.0
yum install -y kubelet-1.28.0 kubeadm-1.28.0 kubectl-1.28.0

# 开机启动 kubelet 服务
systemctl enable kubelet
```

### 三、Master节点初始化k8s

#### 3.1初始化k8s

所有准备工作都完成了，于可以进行 K8s 的初始化了，只需要在 Master 节点上执行以下初始化命令。

```bash
kubeadm init \
  --apiserver-advertise-address=192.168.100.13 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.28.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --cri-socket=unix:///var/run/cri-dockerd.sock \
  --ignore-preflight-errors=all
```

注意：“apiserver-advertise-address” 参数需要替换成你的 Master 节点服务器的内网 IP。

相关参加解释：

- apiserver-advertise-address：集群广播地址，用 master 节点的内网 IP。
- image-repository：由于默认拉取镜像地址 k8s.gcr.io 国内无法访问，这里指定阿里云镜像仓库地址。
- kubernetes-version： K8s 版本，与上面安装的软件版本一致。
- service-cidr：集群 Service 网段。
- pod-network-cidr：集群 Pod 网段。
- cri-socket：指定 cri-socket 接口，我们这里使用 unix:///var/run/cri-dockerd.sock。（很关键，不要漏了）

执行命令后耐心等待，直到安装完成，会出现以下内容。

![image-20241130171603720](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130171603720.png)

#### 3.2设置 kubectl 工具的管理员权限

```bash
# 在 Master 节点上执行以下命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



### 四、Worker 节点加入k8s集群

K8s 初始化之后，就可以在其他 2 个工作节点上执行 “kubeadm join” 命令，因为我们使用了 cri-dockerd ，需要在命令加上 “–cri-socket=unix:///var/run/cri-dockerd.sock” 参数。

```bash
kubeadm join 192.168.100.13:6443 --token jgtkvv.7odlur8h8p8qas2w --discovery-token-ca-cert-hash sha256:80c711a5ae70241f75ee9eb7ccfb848b6e21c6c5341148bd5d0cfc2e88a03af2 --cri-socket=unix:///var/run/cri-dockerd.sock
```

如果无法查看初始化信息，重新生成加入集群命令

```shell
kubeadm token create --print-join-command
```

此时，我们的集群就部署成功了。你可以使用 “kubectl get node” 命令来查看集群节点状态。

```shell
[root@k8s-master ~]# kubectl get node
NAME          STATUS     ROLES           AGE     VERSION
k8s-master    NotReady   control-plane   11m     v1.28.0
k8s-worker1   NotReady   <none>          4m48s   v1.28.0
k8s-worker2   NotReady   <none>          4m46s   v1.28.0
```

注意到所有节点的状态都是 “NotReady”，这是由于集群还缺少网络插件，集群的内部网络还没有正常运作。

### 五、安装k8s网络插件

K8s 网络插件，也称为容器网络接口（CNI）插件，是实现 K8s 集群中容器间通信和网络连接的关键组件，否则 Pod 之间无法通信。Kubernetes 支持多种网络方案，这里我们使用 calico。

Calico 是通过执行一个 YAML 文件部署到 K8s 集群里的，所以我们首先需要通过 “wget” 命令下载这个 YAML 文件。

```bash
# 下载 Calico 插件部署文件
wget https://docs.projectcalico.org/manifests/calico.yaml
```

通过 vi 编辑器修改 “calico.yaml” 文件中的 “CALICO_IPV4POOL_CIDR” 参数，需要与前面 “kubeadm init” 命令中的 “–pod-network-cidr” 参数一样（10.244.0.0/16）。如果文件里的 “CALICO_IPV4POOL_CIDR” 参数前面有 “#”，表示被注释了，需要删除 “#”。

```bash
# 修改文件时注意格式对齐
4598 # The default IPv4 pool to create on startup if none exists. Pod IPs will be
4599 # chosen from this range. Changing this value after installation will have
4600 # no effect. This should fall within `--cluster-cidr`.
4601 - name: CALICO_IPV4POOL_CIDR
4602   value: "10.244.0.0/16"
4603 # Disable file logging so `kubectl logs` works.
4604 - name: CALICO_DISABLE_FILE_LOGGING
4605   value: "true"
```

最后，使用 “kubectl apply” 命令将 Calico 插件部署到集群里，部署 YAML 文件命令如下：

```bash
kubectl apply -f calico.yaml
```

查看pod

```shell
[root@k8s-master ~]# kubectl get pod -A 
NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE
kube-system   calico-kube-controllers-658d97c59c-l76dc   1/1     Running   0             22m
kube-system   calico-node-88q4j                          1/1     Running   0             22m
kube-system   calico-node-8mx64                          1/1     Running   0             22m
kube-system   calico-node-p59gh                          1/1     Running   0             22m
kube-system   coredns-66f779496c-9svrm                   1/1     Running   0             65m
kube-system   coredns-66f779496c-pgzkt                   1/1     Running   0             65m
kube-system   etcd-k8s-master                            1/1     Running   1 (25m ago)   65m
kube-system   kube-apiserver-k8s-master                  1/1     Running   1 (25m ago)   65m
kube-system   kube-controller-manager-k8s-master         1/1     Running   1 (25m ago)   65m
kube-system   kube-proxy-kcvvl                           1/1     Running   1 (25m ago)   65m
kube-system   kube-proxy-nqnvj                           1/1     Running   1 (25m ago)   59m
kube-system   kube-proxy-zrknn                           1/1     Running   1 (25m ago)   59m
kube-system   kube-scheduler-k8s-master                  1/1     Running   1 (25m ago)   65m

```

查看node节点

```shell
[root@k8s-master ~]# kubectl get nodes -owide
NAME          STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k8s-master    Ready    control-plane   73m   v1.28.0   192.168.100.13   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://26.1.4
k8s-worker1   Ready    <none>          66m   v1.28.0   192.168.100.18   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://26.1.4
k8s-worker2   Ready    <none>          66m   v1.28.0   192.168.100.19   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://26.1.4

#可以看见CONTAINER-RUNTIME  依然使用docker！！
```

![image-20241130172128436](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130172128436.png)

### 六、calico镜像拉取失败

注意：如果 Calico 无法安装成功，可能是由于 Calico 镜像无法拉取。本文提供了离线镜像包（[下载地址](https://pan.baidu.com/s/1T0iO-SmsfxzerrtLI3ZC4w?pwd=u155)），可以下载两个压缩文件，然后传输到集群中的所有 3 个节点上。

“calico-image-3.25.1.zip” 中是离线镜像文件，解压后在每个节点上执行导入命令。

```bash
ls *.tar |xargs -i docker load -i {}
```

“calico-yaml.zip” 中是部署文件，在 Master 节点上依次部署两个 YAML 文件。

```bash
# 先删除之前部署不成功的 Calico
kubectl delete -f calico.yaml

# 依次部署两个 yaml 文件
kubectl apply -f tigera-operator.yaml --server-side
kubectl apply -f custom-resources.yaml
```



## Kubernetes部署集群v1.26.3（使用containerd）

环境说明：**这次我们下载containerd作为容器运行时

| 主机名      | 私网地址       | 主机配置                | 备注     |
| ----------- | -------------- | ----------------------- | -------- |
| k8s-master  | 192.168.100.13 | 2核，2GiB，系统盘 40GiB | 控制节点 |
| k8s-worker1 | 192.168.100.18 | 2核，2GiB，系统盘 40GiB | 工作节点 |
| k8s-worker2 | 192.168.100.19 | 2核，2GiB，系统盘 50GiB | 工作节点 |

### 1.关闭防火墙及相关配置

```shell
# 关闭和禁用防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux，selinux 是 Linux 系统下的一个安全服务，需要关闭
setenforce 0  # 临时
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久

# 关闭 Linux 的 swap 分区，提升 k8s 的性能
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
```

### 2.修改主机名

```shell
# 在 192.168.100.13 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-master

# 在 192.168.100.18 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-worker1

# 在 192.168.100.19 这个机器上执行修改主机名命令
hostnamectl set-hostname k8s-worker2
```

### 3.修改主机名DNS解析

```shell
 # 添加各个节点的解析，IP 地址需要替换为你自己服务器的内网 IP 地址。
cat >> /etc/hosts << EOF
192.168.100.13 k8s-master
192.168.100.18 k8s-worker1
192.168.100.19 k8s-worker2
EOF
```

### 4.时间同步

```bash
# 安装ntp服务
yum install ntpdate -y

# 安装完成后使用阿里云的时间服务同步时间 
ntpdate ntp1.aliyun.com
```

### 5.配置网络

为了让 K8s 能够转发网络流量，需要修改 iptables 的配置。执行以下命令快速编辑 “etc/sysctl.d/k8s.conf” 配置文件，增加 iptables 的配置。

```bash
# 修改 Linux 内核参数，添加网桥过滤和地址转发功能
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 使配置生效
sudo sysctl --system
```

### 6.重启服务器

运行 “reboot” 命令，重启服务器，让以上配置生效。

```bash
reboot
```

### 7.安装docker

```8hell
# 通过 wget 命令获取 docker 软件仓库信息
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 安装 docker-ce
yum -y install docker-ce

# 开启 docker 服务
systemctl enable docker && systemctl start docker
```

### 8.docker镜像加速

Docker 安装完成之后，配置阿里云提供的镜像库来加速镜像下载。

docker镜像加速持续更新：https://xuanyuan.me/blog/archives/1154?from=tencent

镜像加速很关键，会影响到后面网络插件的镜像拉取

```bash
cat <<-EOF > /etc/docker/daemon.json 
{
  "registry-mirrors": [
  	"https://docker.linkedbus.com",
  	"https://docker.xuanyuan.me"
  	]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

### 9.添加k8s的yum源

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 10.安装kubeadm、kubelet、kubectl

```shell
# 指定了安装的版本是 1.26.3
yum install -y kubelet-1.26.3 kubeadm-1.26.3 kubectl-1.26.3 --disableexcludes=kubernetes
# 开机启动 kubelet 服务
systemctl enable kubelet
```

### 11.修改核心组件containerd配置文件

```shell
cd /etc/containerd
mv config.toml config.toml.bak
containerd config default > config.toml
```

vim config.toml 修改如下内容：

~~~shell
```
# sandbox_image = "registry.k8s.io/pause:3.6"

sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"

SystemdCgroup = true                     #注意大小写
```
~~~

![image-20241130181617272](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130181617272.png)

![image-20241130181033345](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130181033345.png)

```shell
#修改完后重启容器服务：
systemctl daemon-reload
systemctl restart containerd
systemctl status containerd
```

### 12.初始化k8s集群（master执行）

```shell
kubeadm init \
  --apiserver-advertise-address=192.168.100.13 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.26.3 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all
```

### 13.设置 kubectl 工具的管理员权限（master执行）

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

### 14.worker节点加入集群（master执行）

另外两个worker节点执行

```shell
kubeadm join 192.168.100.13:6443 --token v6vhql.k09lfzv15ulctzhh --discovery-token-ca-cert-hash sha256:2fa9597e0ff0258a2f274ad1fe30fe9877bac822935ca069bc1ff0d7db861217
```

```shell
[root@k8s-master ~]# kubectl get nodes
NAME          STATUS     ROLES           AGE     VERSION
k8s-master    NotReady   control-plane   6m39s   v1.26.3
k8s-worker1   NotReady   <none>          20s     v1.26.3
k8s-worker2   NotReady   <none>          16s     v1.26.3
```

### 15.安装k8s网络插件（master执行）

这次我们使用calico插件

```shell
#下载不了，就手动下载拉到服务器master节点上
wget http://manongbiji.oss-cn-beijing.aliyuncs.com/ittailkshow/k8s/download/calico.yam
kubectl apply -f calico.yaml

#如果想撤回刚刚的网络插件动作
kubectl delete -f calico.yaml
```

![image-20241130184630233](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130184630233.png)

过一会后，查看pod 和节点状态

![image-20241130184819757](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130184819757.png)

接下来看看咱们容器运行时是否用了contained

![image-20241130184905009](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130184905009.png)

接下来就采用contained命令了

举举例子

* 查看命令帮助    ctr -h

* 列出镜像   ctr images list
* 查看容器   ctr containers list 

![image-20241130185644374](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241130185644374.png)





## 添加可视化kuboard v3

参考链接：https://kuboard.cn/install/v3/install-in-k8s.html#%E5%AE%89%E8%A3%85

### 1.启动容器

```shell
#运行容器
sudo docker run -d \
  --restart=unless-stopped \
  --name=kuboard \
  -p 80:80/tcp \
  -p 10081:10081/tcp \
  -e KUBOARD_ENDPOINT="http://129.9.0.113:80" \
  -e KUBOARD_AGENT_SERVER_TCP_PORT="10081" \
  -v /root/kuboard-data:/data \
  swr.cn-east-2.myhuaweicloud.com/kuboard/kuboard:v3
  
```

> 参数解释：
>
> 　　-p 80:80/tcp：将 Kuboard Web 端口 80 映射到宿主机的 80 端口（您可以根据自己的情况选择宿主机的其他端口）；
>
> 　　-p 10081:10081/tcp：将 Kuboard Agent Server 的端口 10081/tcp 映射到宿主机的 10081 端口（您可以根据自己的情况选择宿主机的其他端口）；
>
> 　　-e KUBOARD_ENDPOINT="http://内网IP:80"：指定 KUBOARD_ENDPOINT 为 http://内网IP，如果后续修改此参数，需要将已导入的 Kubernetes 集群从 Kuboard 中删除，再重新导入；

```shell
#如果输错了上述IP地址，可以先删除容器
sudo docker stop kuboard
sudo docker rm kuboard
```

在浏览器输入 http://your-host-ip:80 即可访问 Kuboard v3.x 的界面，登录方式：

用户名： admin
　　密 码： Kuboard123

![image-20241205220211411](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241205220211411.png)

### 2.添加集群

![image-20241205220219223](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241205220219223.png)

**KubeConfig（Kuboard 可以访问 Kubernetes APIServer）**

这种方式需要在master节点中执行 cat ~/.kube/config 查看配置，然后填入相应的地方进行导入 

![image-20241205220513317](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241205220513317.png)

添加成功

![image-20241205220547420](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241205220547420.png)





# 9.部署自己的第一个pod

1.首先拉取nginx镜像到本地

```shell
docker pull docker.1ms.run/nginx:latest
docker tag docker.1ms.run/nginx nginx
```

2.编写pod的yaml文件

```yaml
vim nginx_pod.yaml

apiVersion: v1               # API 版本，指定使用的 Kubernetes API 版本。在这里，使用的是 `v1` 版本。
kind: Pod                     # 资源类型，表示这是一个 Pod 对象。
metadata:
  name: nginx                 # 元数据部分，指定 Pod 的名称为 "nginx"。
  labels:
    app: nginx                # 通过标签定义，指定这个 Pod 的标签为 "app: nginx"。标签可以用来组织和选择资源。
spec:                         # 规格部分，定义 Pod 的详细内容。
  containers:                 # Pod 中运行的容器列表。
  - name: nginx               # 容器的名称为 "nginx"。
    image: nginx:latest       # 容器使用的镜像为 `nginx:latest`。这是官方的 Nginx 镜像，版本为 latest。
    imagePullPolicy: IfNotPresent  #优先从本地拉取镜像，没有再去远程仓库拉取
    ports:                    # 容器暴露的端口。
    - containerPort: 80       # 在容器内暴露端口 80，Nginx 默认使用 80 端口来提供 HTTP 服务。

```

执行yaml文件，部署pod

```shell
kubectl apply -f nginx_pod.yaml
pod/nginx created
```

查看pod状态

```shell
[root@k8s-master ~]# kubectl get pods nginx -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP              NODE          NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          5m59s   10.244.194.67   k8s-worker1   <none>           <none>
```

进入容器查看nginx状态

```shell
[root@k8s-master ~]# kubectl exec -it pod nginx bash
```

```shell
root@nginx:/# curl 10.244.194.67
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

删除上面的pod

```shell
kubectl delete -f nginx_pod.yaml
```

如何强制删除pod

```shell
kubectl delete pod [pod 名字] --grace-period=0 --force
```



# 10.部署自己的第一个deployment

1.编写deployment的yaml文件

```yaml
vim nginx_deployment.yaml

apiVersion: apps/v1  # 定义API版本，指定这是一个应用相关的API版本
kind: Deployment  # 资源类型，这里是一个 Deployment，它用于管理一组Pod
metadata:
  name: nginx-deployment  # Deployment的名称，标识这个Deployment
spec:
  selector:  # 选择器用于定义Deployment如何选择Pod
    matchLabels:
      app: nginx  # Pod选择器，基于标签“app: nginx”来选择Pod
  replicas: 1  # 指定Pod的副本数，表示我们要运行1个nginx的Pod副本
  template:  # Pod模板，用来定义Pod的规范和元数据
    metadata:
      labels:
        app: nginx  # 给Pod添加标签“app: nginx”，用于Deployment选择该Pod
    spec:  # Pod的详细定义，描述Pod内容器的行为
      containers:  # 容器列表
      - name: nginx  # 容器的名称
        image: nginx:latest  # 容器使用的镜像，这里是nginx:latest版本
        imagePullPolicy: IfNotPresent  # 镜像拉取策略，表示如果本地已有该镜像，则不重新拉取，若本地没有，则从远程仓库拉取
        ports:
        - containerPort: 80  # 容器内部开放的端口，这里是80端口，通常是nginx服务使用的端口

```

2.执行yaml文件，部署deployment

```shell
[root@k8s-master ~]# kubectl apply -f nginx_deployment.yaml 
deployment.apps/nginx-deployment created
```

3.查看你deployment的pod的状态

```shell
kubectl get pod -A -owide | grep nginx
```

4.删除deployment

```shell
kubectl delete -f nginx_deployment.yaml
```

如何强制删除pod

```shell
kubectl delete pod [pod 名字] --grace-period=0 --force
```

# 11.暴露自己的第一个服务（service）

1.编写service的yaml文件

```yaml
vim nginx_service.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx  # 选择与Deployment匹配的Pod
  ports:
    - protocol: TCP
      port: 80      # 服务暴露的端口（容器内部使用的端口）
      targetPort: 80  # 容器的端口（nginx服务监听的端口）
      nodePort: 30080  # 指定暴露到节点的端口，这个端口会在所有节点的 IP 上开放
  type: NodePort  # 通过 NodePort 类型暴露服务

```

2.执行yaml文件

```shell
kubectl apply -f nginx_service.yaml
```

3.验证服务暴露

```shell
kubectl get svc nginx-service  # 查看服务的详细信息
```

![image-20241206221014504](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241206221014504.png)

4.访问浏览器

通过任何一个 Kubernetes 节点的 IP 地址和 NodePort 端口，你就可以访问到你的 nginx 服务

![image-20241206221139505](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241206221139505.png)

在kuboard也可以查看到相应的nginx的pod

![image-20241206221242182](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241206221242182.png)



## 使用ClusterIP（默认）

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-cluster-service  # 服务的名字
  namespace: default  # 可以指定命名空间，默认为default
spec:
  selector:
    app: myapp  # 服务选择的 Pod 标签
  ports:
    - protocol: TCP  # 使用 TCP 协议
      port: 80       # 暴露服务的端口
      targetPort: 8080  # Pod 中服务的端口
  type: ClusterIP  # 默认为 ClusterIP，表示该服务仅对集群内部可访问
```

可以用kubectl expose快速暴露单个pod的服务

```
kubectl expose pod pod1 --name=pod1svc --port=80 -n default
```



## 使用NodePort

```yaml
vim nginx-deployment-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx  # 选择与Deployment匹配的Pod
  ports:
    - protocol: TCP
      port: 80        # 服务暴露的端口（集群内使用的端口）
      targetPort: 80   # 容器的端口（nginx服务监听的端口）
      nodePort: 30080  # 指定暴露到节点的端口，这个端口会在所有节点的 IP 上开放
  type: NodePort  # 通过 NodePort 类型暴露服务

---
apiVersion: apps/v1  # 定义API版本，指定这是一个应用相关的API版本
kind: Deployment  # 资源类型，这里是一个 Deployment，它用于管理一组Pod
metadata:
  name: nginx-deployment  # Deployment的名称，标识这个Deployment
spec:
  selector:  # 选择器用于定义Deployment如何选择Pod
    matchLabels:
      app: nginx  # Pod选择器，基于标签“app: nginx”来选择Pod
  replicas: 1  # 指定Pod的副本数，表示我们要运行1个nginx的Pod副本
  template:  # Pod模板，用来定义Pod的规范和元数据
    metadata:
      labels:
        app: nginx  # 给Pod添加标签“app: nginx”，用于Deployment选择该Pod
    spec:  # Pod的详细定义，描述Pod内容器的行为
      containers:  # 容器列表
      - name: nginx  # 容器的名称
        image: nginx:latest  # 容器使用的镜像，这里是nginx:latest版本
        imagePullPolicy: IfNotPresent  # 镜像拉取策略，表示如果本地已有该镜像，则不重新拉取，若本地没有，则从远程仓库拉取
        ports:
        - containerPort: 80  # 容器内部开放的端口，这里是80端口，通常是nginx服务使用的端口

```

应用yaml文件

```
kubectl apply -f nginx-deployment-service.yaml
```

验证服务暴露

```
kubectl get svc nginx-service  # 查看服务的详细信息
```

![image-20241206221014504](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241206221014504.png)

访问浏览器

通过任何一个 Kubernetes 节点的 IP 地址和 NodePort 端口，你就可以访问到你的 nginx 服务

![image-20241206221139505](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241206221139505.png)



## 使用LoadBalancer

使用 `LoadBalancer` 的前提：

- 如果您的 Kubernetes 集群运行在支持负载均衡的云平台（如 AWS、GCP、Azure）上，Kubernetes 会自动创建并配置负载均衡器。
- 如果是在本地或不支持外部负载均衡器的环境（例如 bare metal），可能需要使用 MetalLB 等工具来实现负载均衡功能。



## 使用Ingress（重要）

完善了上面讲到的Nodeport、LoadBalancer等等

\> NodePort方式的缺点是会占用很多集群机器的端口，那么当集群服务变多的时候，这个缺点就愈发明显。

\> LB方式的缺点是每个service需要一个LB，浪费、麻烦，并且需要kubernetes之外设备的支持。

**Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。**

### 什么是Ingress

`Ingress` 是 Kubernetes 中的资源对象，用于管理外部访问集群内服务的规则。它通过定义路由规则，允许外部 HTTP/HTTPS 流量根据域名和路径转发到不同的服务。`Ingress` 通常配合 **Ingress Controller** 使用，提供负载均衡、SSL/TLS 终止等功能。

### Ingress核心概念

 ingress：kubernetes中的一个对象，作用是定义请求如何转发到service的规则；

 ingress controller：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现方式有很多，比如Nginx, Contour, Haproxy等；

### Ingress工作原理

以Nginx类型为例，Ingress工作原理如下：

 编写Ingress规则，说明哪个域名对应kubernetes集群中哪个Service；

 Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nginx反向代理配置；

 Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并动态更新。

![image-20241215213716567](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241215213716567.png)

### Ingress 控制器

为了让 [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/) 在你的集群中工作， 必须有一个 Ingress 控制器正在运行，Kubernetes 作为一个项目，目前支持和维护 [AWS](https://github.com/kubernetes-sigs/aws-load-balancer-controller#readme)、 [GCE](https://git.k8s.io/ingress-gce/README.md#readme) 和 [Nginx](https://git.k8s.io/ingress-nginx/README.md#readme) Ingress 控制器。



### Ingress安装

**下载网址**：https://github.com/kubernetes/ingress-nginx

要根据K8S集群版本下载何时的ingress控制器

我的k8s集群是1.28，根据描述我下载nginx控制器版本是v1.11.3

![image-20241216195545337](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216195545337.png)

#### 1.下载部署清单文件

```shell
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.3/deploy/static/provider/baremetal/deploy.yaml
```

#### 2.部署清单文件（ingress-nginx控制器部署）

```shell
kubectl apply -f deploy.yaml 
```

说明：如果部署失败，需要修改镜像下载地址，主要是因为国内无法拉取到 registry.k8s.io

修改deploy.yaml，将image地址更换

镜像仓库地址：https://docker.aityp.com/r/registry.k8s.io/ingress-nginx

nginx-controller镜像：**swr.cn-north-4.myhuaweicloud.com/ddn-k8s/registry.k8s.io/ingress-nginx/controller:v1.11.3-linuxarm64**      

![image-20241216201411463](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216201411463.png)

kube-webhook-certgen镜像：**swr.cn-north-4.myhuaweicloud.com/ddn-k8s/registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.4-linuxarm64 **              #这里要修改两处地方，两处地方一样的

![image-20241216201545758](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216201545758.png)

修改完了之后再次部署

查看ingress-nginx命名空间下的资源状态

```
 kubectl get all -n ingress-nginx
```

![image-20241216202935213](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216202935213.png)

#### 3.部署测试应用

##### 部署nginx应用

```yaml
vim nginx.yaml

apiVersion: apps/v1  # Deployment API 版本
kind: Deployment     # 资源类型：Deployment
metadata:
  labels:
    app: nginx-deploy  # 用于标识 Deployment 的标签
  name: nginx-deploy    # Deployment 的名称
spec:
  replicas: 3  # 副本数，表示运行的 pod 数量
  selector:
    matchLabels:
      app: nginx-deploy  # 匹配具有相同标签的 pods
  template:
    metadata:
      labels:
        app: nginx-deploy  # 给通过 Deployment 创建的 pods 添加标签
    spec:
      containers:
        - image: nginx:1.20  # 使用的容器镜像
          name: nginx        # 容器的名称
          ports:
            - containerPort: 80  # 容器内部暴露的端口号
---
apiVersion: v1  # Service API 版本
kind: Service    # 资源类型：Service
metadata:
  labels:
    app: nginx-deploy  # 用于标识 Service 的标签
  name: nginx-svc    # Service 的名称
spec:
  ports:
    - port: 80        # Service 暴露的端口号
      protocol: TCP   # 使用的协议（TCP）
      targetPort: 80  # 容器内部要转发流量的端口
  selector:
    app: nginx-deploy  # 根据标签匹配 Deployment 创建的 Pods
  type: ClusterIP  # 服务类型：ClusterIP（仅集群内部访问）

```

应用nginx.yaml文件

```shell
[root@k8s-master ~]# kubectl apply -f nginx.yaml 
deployment.apps/nginx-deploy created
service/nginx-svc created
```

查看pod状态

```shell
[root@k8s-master ~]# kubectl get pod -l app=nginx-deploy
NAME                           READY   STATUS    RESTARTS   AGE
nginx-deploy-8d4956749-gpn6w   1/1     Running   0          3m52s
nginx-deploy-8d4956749-hhgz6   1/1     Running   0          3m52s
nginx-deploy-8d4956749-pkdqh   1/1     Running   0          3m52s
```

查看service状态

```shell
[root@k8s-master ~]# kubectl get svc -l app=nginx-deploy
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-svc   ClusterIP   10.105.209.79   <none>        80/TCP    10m
```



##### 部署http应用

```yaml
vim http.yaml

apiVersion: apps/v1  # Deployment API 版本
kind: Deployment     # 资源类型：Deployment
metadata:
  labels:
    app: http-deploy  # 用于标识 Deployment 的标签
  name: http-deploy    # Deployment 的名称
spec:
  replicas: 3  # 副本数，表示运行的 pod 数量
  selector:
    matchLabels:
      app: http-deploy  # 匹配具有相同标签的 pods
  template:
    metadata:
      labels:
        app: http-deploy  # 给通过 Deployment 创建的 pods 添加标签
    spec:
      containers:
        - image: httpd  # 使用的容器镜像
          name: http    # 容器的名称
          ports:
            - containerPort: 80  # 容器内部暴露的端口号
---
apiVersion: v1  # Service API 版本
kind: Service    # 资源类型：Service
metadata:
  labels:
    app: http-deploy  # 用于标识 Service 的标签
  name: http-svc    # Service 的名称
spec:
  ports:
    - port: 80        # Service 暴露的端口号
      protocol: TCP   # 使用的协议（TCP）
      targetPort: 80  # 容器内部要转发流量的端口
  selector:
    app: http-deploy  # 根据标签匹配 Deployment 创建的 Pods
  type: ClusterIP  # 服务类型：ClusterIP（仅集群内部访问）

```

应用http.yaml文件

```shell
 kubectl apply -f http.yaml 
```

查看pod状态

```shell
[root@k8s-master ~]# kubectl get pod -l app=http-deploy
NAME                           READY   STATUS    RESTARTS   AGE
http-deploy-7c9b86d896-9zrh2   1/1     Running   0          83s
http-deploy-7c9b86d896-bzlnf   1/1     Running   0          83s
http-deploy-7c9b86d896-kpt8m   1/1     Running   0          83s
```

查看service状态

```shell
[root@k8s-master ~]# kubectl get svc -l app=http-deploy
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
http-svc   ClusterIP   10.107.150.147   <none>        80/TCP    118s
```

#### 4.暴露ingress controller（创建ingress规则）

**接下来我们只需要暴露ingress controller接口就可以访问后面的pod了（不管有多少个pod，都可以）**

**也就是说，ingress只需要暴露控制器接口，把请求根据相应的规则转发到后端的pod对应的服务上面**

```yaml
#创建ingress 规则
vim ingress.yaml

apiVersion: networking.k8s.io/v1  # Ingress API 版本
kind: Ingress                     # 资源类型：Ingress
metadata:
  name: ingress01  # Ingress 的名称
spec:
  ingressClassName: nginx  # 指定使用的 Ingress 控制器
  rules:
    - host: www.nginx.com  # 第一个主机名
      http:
        paths:
          - backend:
              service:
                name: nginx-svc  # 目标 Service 名称
                port:
                  number: 80  # 目标端口
            path: /  # 请求路径
            pathType: Prefix  # 路径匹配类型（Prefix 表示前缀匹配）
    - host: www.http.com  # 第二个主机名
      http:
        paths:
          - backend:
              service:
                name: http-svc  # 目标 Service 名称
                port:
                  number: 80  # 目标端口
            path: /  # 请求路径
            pathType: Prefix  # 路径匹配类型（Prefix 表示前缀匹配）
```

应用ingress.yaml

```
kubectl apply -f ingress.yaml 
```

查看ingress状态

```shell
[root@k8s-master ~]# kubectl get ingress
NAME        CLASS   HOSTS                        ADDRESS          PORTS   AGE
ingress01   nginx   www.nginx.com,www.http.com   192.168.100.18   80      38s
```

#### 5.测试访问

我在我自己window主机访问就要修改hosts文件

根据ingress去修改

![image-20241216205502813](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216205502813.png)



修改路径：C:\Windows\System32\drivers\etc\hosts

![image-20241216205618539](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216205618539.png)

保存

浏览器访问

访问方式：**域名＋暴露的端口  （暴露端口是部署ingress-nginx的时候随机选择nodePort端口）**

![image-20241216205954426](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216205954426.png)

http://www.nginx.com:30984/

![image-20241216210012567](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216210012567.png)

http://www.http.com:30984/

![image-20241216210028687](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241216210028687.png)



**另外，你也可以只用一个域名，然后ingress.yaml文件下的path: 填写不同的访问路径去进行请求转发**

# 12、使用deployment部署nginx：1.18，副本数为5

```shell
#拉取镜像：nginx:1.18
docker pull docker.1ms.run/nginx:1.18
#打标签
docker tag docker.1ms.run/nginx:1.18 nginx:1.18
```

![image-20241211200334756](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211200334756.png)

![image-20241211201130408](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211201130408.png)

```shell
#编写deployment的yaml文件
vim nginx1_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.18
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        resources:     #添加容器规格，方便接下来的hpq测试！！！
          limits:
            cpu: "50m"  # 设置CPU限制为50m
          requests:
            cpu: "50m"  # 可选：也设置CPU请求为50m，以确保Pod有足够的资源分配
```

```shell
#执行文件
kubectl apply -f nginx1_deployment.yaml 
#查询deployment状态
kubectl get pod -l app=nginx
```

![image-20241211204958045](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211204958045.png)

验证nginx版本，我选择第一个pod去验证

```shell
kubectl exec -it nginx-deployment-584f64656-79sql -- bash -c "nginx -v"
```

![image-20241211205049654](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211205049654.png)



# 13、将nginx的版本升级到nginx:1.20

```shell
#拉取镜像：nginx:1.20
docker pull docker.1ms.run/nginx:1.20
#打标签
docker tag docker.1ms.run/nginx:1.20 nginx:1.20
```

![image-20241211202327002](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211202327002.png)

由于是升级，直接在之前的nginx1.18的yaml文件上修改参数

```shell
vim nginx1_deployment.yaml
#把原先的镜像nginx:1.18替换成nginx:1.20
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        resources:      #添加容器规格，方便接下来的hpq测试！！！
          limits:
            cpu: "50m"  # 设置CPU限制为50m
          requests:
            cpu: "50m"  # 可选：也设置CPU请求为50m，以确保Pod有足够的资源分配
```

```
kubectl apply -f nginx1_deployment.yaml
```

然后查看pod

```shell
 kubectl get pod -l app=nginx
```

![image-20241211205205183](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211205205183.png)

验证版本：

![image-20241211205259376](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211205259376.png)



* 查看deployment的历史版本记录

```shell
kubectl rollout history deployment [deployment_name]
```

![image-20241211205505077](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211205505077.png)

* 查看某个历史记录的详细信息

```shell
kubectl rollout history deployment [deployment_name] --revision=2  #其中--revision表示指定修订版本
```

* 回滚到指定版本

```shell
kubectl rollout undo deployment deployment_name --to-revision=2   #--to-revision表示回滚到指定的修订版本。
```

* 查看rc信息

```shell
kubectl get rs
```

![image-20241211205836181](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211205836181.png)

会发现有旧版本的和新版本之间的交替，旧版本的副本数已经缩容到0，新版本的副本数扩容到5

* 确认更新状态

```shell
kubectl rollout status deployment/nginx-deployment
```

![image-20241211210108217](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211210108217.png)



# 14、对上述deployment配置动态水平扩展HPA

## **HPA工作原理：**

Kubernetes中的某个Metrics Server（Heapster或自定义Metrics Server）持续采集所有的pod副本的指标数据，HPA控制器通过Metrics Server的API（Heapster的API或聚合API）获取这些数据，基于用户定义 的扩缩容规则进行计算，得到目标Pod副本数量

当目标Pod副本数量与当前副本数量不同时，HPA控制器就向Pod的副本控制器 （Deployment、RC或ReplicaSet）发起scale操作，调整Pod的副本数量， 完成扩缩容操作。

**♦** **算法介绍**

从最基本的角度来看，Pod 水平自动扩缩控制器根据当前指标和期望指标来计算扩缩比例。

**期望副本数 = ceil[当前副本数 \* (当前指标 / 期望指标)]**



## 安装

我的k8s集群是v1.28

## 1.先安装Metrics Server

Metrics-Server是集群核心监控数据的聚合器。通俗地说，它存储了集群中各节点的监控数据，并且提供了API以供分析和使用

```shell
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

然后拉到服务器里面

由于网络问题我们先修改components.yaml文件

1.把文件中的镜象地址更换成阿里云的

**把“image: registry.k8s.io/metrics-server/metrics-server:v0.7.2”  换成**

**“ image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.7.2”**

![image-20241211213821503](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211213821503.png)

2.然后再把证书验证禁用，以免后面有问题（生产环境不建议这么做）

```shell
spec:
  containers:
  - args:   
    - --cert-dir=/tmp
    - --secure-port=10250
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --kubelet-use-node-status-port
    - --metric-resolution=15s
    - --kubelet-insecure-tls   #增加
```

![image-20241211213923505](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211213923505.png)

执行应用yaml文件

```
 kubectl apply -f components.yaml 
```

验证是否运行成功

```
kubectl get pods -n kube-system | grep metrics-server
```

![image-20241211214220635](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211214220635.png)

```
kubectl top nodes
```

![image-20241211215115457](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211215115457.png)

```
kubectl top pod
```

![image-20241211223228802](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211223228802.png)

## 2.给之前的nginx deployment创建一个service服务暴露

```yaml
vim nginx-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-service  # 服务名称
  namespace: default  # 指定命名空间，默认是"default"
spec:
  type: NodePort  # 指定服务类型为NodePort
  selector:
    app: nginx  # 确保这个标签与Deployment中Pod的标签匹配
  ports:
    - protocol: TCP
      port: 80  # 服务监听的端口（集群内部）
      targetPort: 80  # Pod上容器暴露的端口
      nodePort: 30080  # （可选）指定节点上的端口号，范围通常在30000-32767之间
```

然后应用

```shell
kubectl apply -f nginx-service.yaml 
kubectl get services -n default
```

![image-20241211221757903](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211221757903.png)



## 3.创建HPA的deployment

```yaml
vim nginx-hpa.yaml

apiVersion: autoscaling/v2  # 指定API版本，这里使用的是v2
kind: HorizontalPodAutoscaler  # 定义资源类型为HorizontalPodAutoscaler，用于自动扩展Pod的数量

metadata:
  name: nginx-hpa  # HPA资源的名称，便于管理和识别
  namespace: default  # 指定该HPA所在的命名空间，默认是"default"，如果你的应用在其他命名空间中，请更改此值

spec:
  scaleTargetRef:  # 引用要自动扩展的目标资源（如Deployment、ReplicaSet等）
    apiVersion: apps/v1  # 目标资源的API版本
    kind: Deployment  # 目标资源的类型，这里是Deployment
    name: nginx-deployment  # 目标资源的具体名称，即你要扩展的Deployment名称

  minReplicas: 3  # 设置HPA管理的最小Pod数量，确保至少有3个Pod运行
  maxReplicas: 10  # 设置HPA管理的最大Pod数量，限制最多有10个Pod运行

  metrics:  # 定义用于触发扩展的度量标准列表
  - type: Resource  # 指定使用的度量类型，这里使用的是Resource类型，通常指的是CPU或内存
    resource:
      name: cpu  # 指定具体的资源名称，这里是CPU
      target:
        type: Utilization  # 指定目标值的类型，Utilization表示基于资源利用率的目标
        averageUtilization: 50  # 设置目标平均CPU利用率百分比，当CPU利用率超过50%时，HPA将增加Pod数量；低于50%时减少Pod数量
```

然后应用

```
kubectl apply -f nginx-hpa.yaml
```

## 3.验证HPA配置

```
kubectl get hpa
```

![image-20241211223023564](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211223023564.png)

现在负载量不够高，就会把pod缩到最小量！

## 4.测试HPA

尝试生成一些负载，使Nginx Pods的CPU使用率升高。你可以使用工具如 stress 或者通过HTTP请求来增加负载

复制到在另一个窗口执行

```shell
# 使用 kubectl run 来创建一个临时Pod并使用 stress 工具生成CPU负载 
kubectl run load-generator --image=busybox:1.30 --restart=Never --command -- sh -c "while true; do wget -q -O- http://nginx-service.default.svc.cluster.local; done"
```

- `kubectl run load-generator`: 创建名为 `load-generator` 的Pod。
- `--image=busybox:1.30`: 指定使用的镜像为 `busybox:1.30`，这是一个轻量级的容器镜像，适合用于执行简单的命令。
- `--restart=Never`: 确保这个Pod不会自动重启，因为我们要它只运行一次任务然后停止。
- `--`: 分隔符，表示后面跟随的是要在容器内执行的命令。
- `wget -q -O- http://nginx-deployment.default.svc.cluster.local`: 这个命令在Pod启动时立即执行一次HTTP请求，以确保服务可用。
- `--command`: 表示后面的参数是容器启动时要执行的命令，而不是默认的入口点或命令。
- `sh -c "while true; do wget -q -O- http://nginx-service.default.svc.cluster.local; sleep 1; done"`: 在容器内启动一个shell，并在这个shell中无限循环地执行HTTP请求，每次请求之间间隔1秒。
- **`nginx-service.default.svc.cluster.local`这个可以使用`kubectl get svc` 命令查询**

**清理**：记得在测试完成后删除这个负载生成Pod，以免不必要的资源消耗：

```shell
kubectl delete pod load-generator
```



**步骤如下：**

窗口一：实时监控hpa的负载量

```
kubectl get hpa --watch
```

窗口二：执行命令(确保有busybox:1.30镜像，我已经拉取到本地了)

```shell
kubectl run load-generator --image=busybox:1.30 --restart=Never --command -- sh -c "while true; do wget -q -O- http://nginx-service.default.svc.cluster.local; done"
```

![image-20241211223127532](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211223127532.png)

然后给点耐心，等负载量超导50%看看有什么变化！

![image-20241211224316505](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211224316505.png)

![image-20241211224325454](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241211224325454.png)

因为新创建了一个pod，所以负载量又下来了！

# 15、k8s连接harbor使用secret

k8s如何配置secret保存harbor仓库账号密码、pod中怎么使用harbor仓库镜像

**环境**：centos 7.9 、k8s-v1.28、harbor-v2.8.4

## **Secret概述**

Secret 解决了密码、token、秘钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者 Pod Spec 中。Secret 可以以 Volume 或者环境变量的方式使用。

**secret 可选参数有三种:**

1.generic: 通用类型，通常用于存储密码数据。

2.tls：此类型仅用于存储私钥和证书。

3.docker-registry: 若要保存 docker 仓库的认证信息的话，就必须使用此种类型来创建。

**Secret 类型：**

1.Service Account：用于被 serviceaccount 引用。serviceaccout 创建时 Kubernetes 会默认创建对应的 secret。Pod 如果使用了serviceaccount，对应的 secret 会自动挂载到 Pod的/run/secrets/kubernetes.io/serviceaccount 目录中。

2.Opaque：base64 编码格式的 Secret，用来存储密码、秘钥等。可以通过 base64 --decode 解码获得原始数据，因此安全性弱。

3.kubernetes.io/dockerconfigjson：用来存储私有 docker registry 的认证信息。

**由于我们目标是连接私有仓库，我们就是使用`kubernetes.io/dockerconfigjson`这种类型的`Secret`存储私有`docker registry`的认证信息**

## 创建私有镜像仓库Secret 的两种方式

1、先在docker上登录harbor镜像仓库地址，得到 /root/.docker/config.json 文件，使用该文件进行创建，可以查看官网： https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/pull-image-private-registry/
        2、使用命令行镜像创建；这里主要讲解使用命令行创建。



## 1.k8s使用命令行创建Secret 配置harbor私有镜像仓库地址

```yaml
# 直接使用命令行创建一个名称叫做harbor-registry的secret 
# Docker 仓库的用户名
# Docker 仓库的密码
# Docker 仓库的地址（包括端口）
kubectl -n default create secret docker-registry harbor-registry  \
  --docker-username=admin \
  --docker-password=Harbor12345 \
  --docker-server=192.168.100.14

#查看secret，已经创建成功
[root@k8s-master ~]# kubectl get secret harbor-registry
NAME              TYPE                             DATA   AGE
harbor-registry   kubernetes.io/dockerconfigjson   1      119s

#查看secret
[root@k8s-master ~]# kubectl describe  secret harbor-registry
Name:         harbor-registry
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  108 bytes


# 对.data.dockerconfigjson 内容进行转储并执行 base64 解码可以发现：
[root@k8s-master ~]# kubectl get secret harbor-registry -o jsonpath='{.data.*}' | base64 -d
{"auths":{"192.168.100.14":{"username":"admin","password":"Harbor12345","auth":"YWRtaW46SGFyYm9yMTIzNDU="}}}

```

## 2.把harbor仓库地址配置到node节点上

注意：在node节点上需要docker能正常请求到私有镜像仓库地址，还需要将harbor镜像仓库地址添加到`/etc/docker/daemon.json` ，如下所示：

```shell
vim /etc/docker/daemon.json 
{
  "registry-mirrors": [
    "https://docker.linkedbus.com",
    "https://docker.xuanyuan.me"
  ],
  "insecure-registries": ["192.168.100.14"]
}


#
systemctl  daemon-reload		#重载
systemctl  restart docker		#重启docker
```



## 3.pod使用imagePullSecrets参数指定镜像拉取秘钥

* 首先登录harbor确保有个nginx镜像用于测试，没有的话要推送上去

![image-20241219195748904](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241219195748904.png)

* 编写一个test.nginx.yaml文件

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-nginx
  labels:
    app: test-nginx
  namespace: default  # 命名空间，要与 secret 的命名空间一致，不要找不到对应的 secret
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-nginx
  template:
    metadata:
      labels:
        app: test-nginx
    spec:
      imagePullSecrets:  # 使用 imagePullSecrets 参数指定镜像拉取秘钥
        - name: harbor-registry  # 使用我们的 secret，即 harbor-registry 
      containers:
        - image: 192.168.100.14/myharbor/nginx:1.18  # 指定拉取的镜像，注意这里的镜像名称只需要写到 IP 地址，端口号，仓库名，镜像名，版本
          name: nginx-container
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80

```

* 应用yaml文件

```shell
kubectl apply -f test-nginx.yaml

#查看pod运行状态
[root@k8s-master ~]# kubectl get pods -l app=test-nginx
NAME                          READY   STATUS    RESTARTS   AGE
test-nginx-6db5f5b986-9f9qv   1/1     Running   0          3s


#查看pod详细信息
[root@k8s-master ~]# kubectl describe  pods test-nginx-6db5f5b986-9f9qv

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  22s   default-scheduler  Successfully assigned default/test-nginx-6db5f5b986-9f9qv to k8s-worker1
  Normal  Pulling    21s   kubelet            Pulling image "192.168.100.14/myharbor/nginx:1.18"
  Normal  Pulled     21s   kubelet            Successfully pulled image "192.168.100.14/myharbor/nginx:1.18" in 86ms (86ms including waiting)
  Normal  Created    21s   kubelet            Created container nginx-container
  Normal  Started    21s   kubelet            Started container nginx-container

#可以看见我成功登录harbor并且拉佢了镜像运行pod

#根据pod运行到哪个节点就去那个节点查看镜像（我这里是worker1）
[root@k8s-worker1 ~]# docker images
```

![image-20241219201204477](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241219201204477.png)





# 16、存储

## k8s存储概述

程序运行在容器中，而容器的生命周期可能极其短暂，容器销毁，数据丢失，因此，为了持久化容器中的数据，Volume出现了，Volume是Pod中能够被多个容器访问的共享目录，它被定义在Pod上，然后被一个Pod里的多个容器挂载到具体的文件目录下。

**kubernetes通过Volume实现：**

* 同一个Pod中不同容器之间的数据共享

* 数据的持久化存储

**kubernetes的Volume支持多种类型：**

**基本存储：**EmptyDir、HostPath、NFS

**高级存储：**PV、PVC

**配置存储：**ConfigMap、Secret







## 最基本的存储EmptyDir（静态）

EmptyDir存储就是主机上的一个空目录。

EmptyDir是在Pod被分配到Node时创建的，它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为kubernetes会自动分配一个目录，当Pod销毁时， EmptyDir中的数据也会被永久删除

举例子:

**解释说明如下：**

* nginx 容器：运行一个 nginx 服务，并将日志存储在 /var/log/nginx 目录，该目录由一个名为 logs-volume 的共享存储卷提供。
* busybox 容器：运行一个命令，实时读取 logs-volume 中的日志文件（/logs/access.log），用于动态监控 nginx 生成的日志。
* 共享存储卷 (logs-volume)：
* 使用 emptyDir 类型的存储卷，它是一个临时目录，两个容器共享这个目录来交换数据（nginx 写入日志，busybox 读取日志）。
* 当 Pod 被删除时，存储在 logs-volume 中的数据也会被清除。

```yaml
vim emptydir.yaml

apiVersion: v1
kind: Pod
metadata:
  name: volume-emptydir
spec:
  containers:
    - name: nginx
      image: nginx:1.18
      ports:
        - containerPort: 80
      volumeMounts:  # 将 logs-volume 挂载到 nginx 容器中，对应的目录为 /var/log/nginx
        - name: logs-volume
          mountPath: /var/log/nginx
    - name: busybox
      image: busybox:1.30
      command: ["/bin/sh", "-c", "tail -f /logs/access.log"]  # 初始命令，动态读取指定文件内容
      volumeMounts:  # 将 logs-volume 挂载到 busybox 容器中，对应的目录为 /logs
        - name: logs-volume
          mountPath: /logs
  volumes:  # 声明 volume，name 为 logs-volume，类型为 emptyDir
    - name: logs-volume
      emptyDir: {}

```

```shell
#创建pod
[root@k8s-master ~]# kubectl create -f emptydir.yaml 
pod/volume-emptydir created

#查看pod
[root@k8s-master ~]# kubectl get pods volume-emptydir -owide
NAME              READY   STATUS    RESTARTS   AGE    IP              NODE          NOMINATED NODE   READINESS GATES
volume-emptydir   2/2     Running   0          109s   10.244.194.79   k8s-worker1   <none>           <none>

#通过pod的ip访问一下nginx
[root@k8s-master ~]# curl 10.244.194.79

#通过kubectl logs命令查看指定容器的标准输出
[root@k8s-master ~]# kubectl logs -f volume-emptydir -c busybox
10.244.235.192 - - [19/Dec/2024:12:33:50 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"

注意：
-c busybox：指定查看 busybox 容器的日志。因为在 Pod 中有多个容器（这里有 nginx 和 busybox 容器），所以需要通过 -c 参数来指定查看哪一个容器的日志。
```

![image-20241219203444386](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241219203444386.png)



********************下面重要介绍NFS、PV、PVC******************



## NFS存储（静态）

Kubernetes的NFS存储**用于将某事先存在的NFS服务器导出export的存储空间挂载到Pod中来供Pod容器使用**。 与emptyDir不同的是，NFS存储在Pod对象终止后仅是被卸载而非删除。 另外，NFS是文件系统及共享服务，它支持同时存在多路挂载请求。

### 安装

```shell
# 安装nfs服务
[root@nfs ~]# yum install nfs-utils -y

# 准备一个共享目录
[root@nfs ~]# mkdir -p /data/nfs

# 将共享目录以读写权限暴露给192.168.100.0/24网段中的所有主机
[root@nfs ~]# vim /etc/exports

[root@nfs ~]# more /etc/exports

/data/nfs 192.168.100.0/24(rw,no_root_squash)

# 启动nfs服务
[root@nfs ~]# systemctl restart nfs

#每个节点安装nfs客户端
yum install -y nfs-utils #无需启动服务
```

编写yaml文件



```yaml
apiVersion: v1  # API 版本，v1 是最基础的版本
kind: Pod  # 定义对象类型为 Pod
metadata:
  name: volume-nfs  # Pod 的名称
  namespace: dev  # Pod 所属的命名空间，这里是 dev
spec:
  containers:
    # 第一个容器，nginx
    - name: nginx  # 容器的名称
      image: nginx:1.17.1  # 使用的镜像版本
      ports:
        - containerPort: 80  # 容器开放的端口，nginx 默认使用端口 80
      volumeMounts:
        - name: logs-volume  # 关联的卷名称
          mountPath: /var/log/nginx  # 将卷挂载到容器内部的路径
          
    # 第二个容器，busybox
    - name: busybox  # 容器名称
      image: busybox:1.30  # 使用的镜像版本
      command: ["/bin/sh", "-c", "tail -f /logs/access.log"]  # 运行命令，持续监控日志文件
      volumeMounts:
        - name: logs-volume  # 关联的卷名称
          mountPath: /logs  # 将卷挂载到容器内部的路径

  volumes:
    # 定义名为 logs-volume 的卷，使用 NFS 存储类型
    - name: logs-volume  # 卷的名称
      nfs:  # NFS 存储配置
        server: 192.168.100.100  # NFS 服务器的 IP 地址
        path: /data/nfs  # NFS 共享目录路径

```

检测

```shell
# 创建pod
[root@k8s-master01 ~]# kubectl create -f volume-nfs.yaml
pod/volume-nfs created
# 查看pod
[root@k8s-master01 ~]# kubectl get pods volume-nfs -n dev
NAME READY STATUS RESTARTS AGE
volume-nfs 2/2 Running 0 2m9s
# 查看nfs服务器上的共享目录，发现已经有文件了
[root@k8s-master01 ~]# ls /data/nfs/
access.log error.log
```



## 什么是PV、PVC

**PV（Persistent Volume）：**是持久化卷的意思，是对底层的共享存储的一种抽象。一般情况下PV由

kubernetes管理员进行创建和配置，它与底层具体的共享存储技术有关，并通过插件完成与共享存储的对

接。

**PVC（Persistent Volume Claim**）：是持久卷声明的意思，是用户对于存储需求的一种声明。换句话

说，PVC其实就是用户向kubernetes系统发出的一种资源需求申请。



### PV介绍

**存储类型：**

底层实际存储的类型，kubernetes支持多种存储类型，每种存储类型的配置都有所差异

**存储能力（capacity）：**

目前只支持存储空间的设置( storage=1Gi )，不过未来可能会加入IOPS、吞吐量等指标的配置

**访问模式（accessModes）：**

用于描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：

ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载

ReadOnlyMany（ROX）： 只读权限，可以被多个节点挂载

ReadWriteMany（RWX）：读写权限，可以被多个节点挂载

需要注意的是，底层不同的存储类型可能支持的访问模式不同

**回收策略（persistentVolumeReclaimPolicy）：**

当PV不再被使用了之后，对其的处理方式。目前支持三种策略：

Retain （保留） 保留数据，需要管理员手工清理数据

Recycle（回收） 清除 PV 中的数据，效果相当于执行 rm -rf /thevolume/*

Delete （删除） 与 PV 相连的后端存储完成 volume 的删除操作，当然这常见于云服务商的存储服务

**存储类别：**

PV可以通过storageClassName参数指定一个存储类别

具有特定类别的PV只能与请求了该类别的PVC进行绑定

未设定类别的PV则只能与不请求任何类别的PVC进行绑定

**状态（status）：**

一个 PV 的生命周期中，可能会处于4中不同的阶段：

Available（可用）： 表示可用状态，还未被任何 PVC 绑定

Bound（已绑定）： 表示 PV 已经被 PVC 绑定

Released（已释放）： 表示 PVC 被删除，但是资源还未被集群重新声明

Failed（失败）： 表示该 PV 的自动回收失败

### PV创建

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2
spec:
  nfs:  # 存储类型，表示使用 NFS 协议
    path: /path/to/nfs/share  # NFS 服务器上的路径
    server: <nfs-server-ip>  # NFS 服务器的 IP 地址或主机名
  capacity:  # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes:  # 访问模式，表示该持久卷的访问权限
    - ReadWriteOnce  # 表示单节点读写
  storageClassName: standard  # 存储类别，标明使用的存储类
  persistentVolumeReclaimPolicy: Retain  # 回收策略
```

### PVC创建

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-logs  # PVC 的名称
  namespace: dev  # PVC 所在的命名空间，这里是 dev
spec:
  accessModes:
    - ReadWriteOnce  # 与 PV 配置的访问模式一致，表示 PVC 只能在一个节点上进行读写
  resources:
    requests:
      storage: 2Gi  # 请求 2Gi 的存储空间
  storageClassName: standard  # 使用与 PV 相同的存储类，确保 PVC 能够与 PV 匹配

```



## 静态pv和动态pv

### 静态 PV（Static PV）

静态 PV 是由管理员预先创建和管理的存储卷，通常通过管理员手动创建一个或多个 PV 来指定存储资源。存储资源（例如 NFS、iSCSI、AWS EBS 等）已经在外部系统中存在，并且这些资源与 Kubernetes 的 PV 绑定。

- **创建方式**：管理员在 Kubernetes 中手动创建 PV，指定底层存储的相关配置（如存储类型、容量、访问模式等）。
- **绑定过程**：当一个 Pod 请求某个 PV 时，Kubernetes 会根据 PV 的属性（如存储容量和访问模式）选择一个匹配的 PV，并将其与该 Pod 绑定。
- **管理方式**：管理员需要手动创建和管理 PV，并确保底层存储的可用性。

#### 优点：

- 适用于需要完全控制存储的情况。
- 管理员可以指定特定的存储资源来满足企业级应用的需求。

#### 缺点：

- 手动管理，容易出错或不够灵活。
- 当存储资源需求发生变化时，需要手动创建和调整 PV。



### 动态 PV（Dynamic PV）

动态 PV 是由 Kubernetes 根据 PVC（Persistent Volume Claim）自动创建的。管理员定义一个 StorageClass（存储类），指定动态创建 PV 的存储类型及策略。当用户创建 PVC 时，Kubernetes 根据 PVC 的请求和指定的 StorageClass 自动创建相应的 PV 并进行绑定。

- **创建方式**：当用户创建 PVC 时，如果没有匹配的 PV，Kubernetes 会根据 PVC 的要求（如存储类型、容量等）自动选择一个 StorageClass，并根据 StorageClass 的定义创建一个新的 PV。
- **绑定过程**：PVC 和 PV 自动绑定，无需管理员手动操作。
- **管理方式**：管理员通过定义 StorageClass，指定不同存储类型的配置和动态创建策略。PVC 用户无需关心底层存储细节。

#### 优点：

- 自动化、灵活，简化了存储资源的管理。
- 根据需要动态创建 PV，减少了手动干预的需要。
- 更适合弹性伸缩和大规模部署场景。

#### 缺点：

- 依赖于 Kubernetes 的存储插件和配置，需要事先配置好 StorageClass。
- 可能会受到存储供应商限制或性能瓶颈影响。



## 实战 使用 nfs 作为动态 storageClass 存储

如果PVC请求成千上万，那么就需要创建成千上万的PV，对于运维人员来说维护成本很高，Kubernetes提供一种自动创建PV的机制，叫StorageClass，它的作用就是创建PV的模板。k8s集群管理员通过创建storageclass可以动态生成一个存储卷pv供k8s pvc使用。

动态PV供给是通过StorageClass实现的，StorageClass定义了如何创 建PV。

**以nfs为例，要想使用NFS，我们需要一个nfs-client的自动装载程序，称之为provisioner，这个程序会使用我们已经配置好的NFS服务器自动创建持久卷，也就是自动帮我们创建PV。**

`nfs-client-provisioner` 是一个Kubernetes的简易NFS的外部provisioner，本身不提供NFS，需要现有的NFS服务器提供存储

![image-20241223202419790](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223202419790.png)



### 环境说明

| NFS服务端   | 192.168.100.14 |
| ----------- | -------------- |
| k8s master  | 192.168.100.13 |
| k8s worker1 | 192.168.100.18 |
| k8s worker2 | 192.168.100.19 |



### 搭建nfs

#在 `192.168.100.14` 上安装 NFS 服务

```
yum install -y nfs-utils
```

创建一个目录，用于共享数据：

```
sudo mkdir -p /mnt/nfs_share
```

编辑 NFS 的导出配置文件，允许你的 Kubernetes 节点（`192.168.100.13`，`192.168.100.18` 和 `192.168.100.19`）访问共享目录：

```
sudo vi /etc/exports
```

添加如下内容，允许这些 IP 地址访问：

```
/mnt/nfs_share 192.168.100.0/24(rw,no_root_squash)
```

重新导出 NFS 目录，并启动 NFS 服务：

```
sudo exportfs -r
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
```

确保 NFS 服务已成功启动：

```
sudo systemctl status nfs-server
```

**检查导出的共享目录：** 你可以使用以下命令查看 NFS 服务器上已经导出的共享目录：

```
sudo exportfs -v
```



#接下来的是在k8s节点执行

 Kubernetes 节点（`192.168.100.13`，`192.168.100.18` 和 `192.168.100.19`）下载nfs

```
yum install -y nfs-utils  #无需启动服务
```

在 NFS 客户端（例如你的 Kubernetes 节点）上，检查是否能够成功连接到 NFS 服务器，并查看共享的目录：

```
showmount -e 192.168.100.14
```

### 创建运行提供者（provisioner）服务账户

```
vim /root/test/nfs-serviceaccount.yaml
```



### 部署deployment（nfs）

```
vim /root/test/deployment.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-provisioner
```

```
kubectl apply -f /root/test/nfs-serviceaccount.yaml
```

```shell
[root@k8s-master ~]# kubectl get sa nfs-provisioner
NAME              SECRETS   AGE
nfs-provisioner   0         85s
```



### 给服务账户授权

（也可以使用yaml文件创建，例如rbac.yaml），这里我们直接用kubectl命令简单授予权限

```yaml
kubectl create clusterrolebinding nfs-provisioner-clusterrolebinding --clusterrole=cluster-admin --serviceaccount=default:nfs-provisioner
```

### 安装nfs-provisioner

```
vim /root/test/nfs-deployment.yaml
```



```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-provisioner  # 部署名称
spec:
  selector:
    matchLabels:
      app: nfs-provisioner  # 匹配标签，用于选择该部署的 Pod
  replicas: 1  # 副本数目，表示部署一个 Pod
  strategy:  # 更新策略
    type: Recreate  # 使用 Recreate 策略，表示每次更新时销毁旧 Pod，再创建新 Pod
  template:
    metadata:
      labels:
        app: nfs-provisioner  # Pod 标签，用于识别
    spec:
      serviceAccount: nfs-provisioner  # 指定 Service Account，以便该 Pod 可以访问 Kubernetes API
      containers:
        - name: nfs-provisioner  # 容器名称
          image: registry.cn-beijing.aliyuncs.com/mydlq/nfs-subdir-external-provisioner:v4.0.0  # 容器镜像地址
          imagePullPolicy: IfNotPresent  # 如果镜像已经存在，跳过拉取操作
          volumeMounts:
            - name: nfs-client-root  # 卷的名称，与下面 volumes 部分的 name 匹配
              mountPath: /persistentvolumes  # 容器内部挂载路径
          env:
            - name: PROVISIONER_NAME
              value: example.com/nfs  # NFS Provisioner 的名称
            - name: NFS_SERVER
              value: 192.168.100.14  # NFS 服务端的 IP 地址
            - name: NFS_PATH
              value: /mnt/nfs_share  # NFS 共享目录的路径
      volumes:
        - name: nfs-client-root  # 卷名称，确保与 volumeMounts 中的一致
          nfs:
            server: 192.168.100.14  # NFS 服务端的 IP 地址
            path: /mnt/nfs_share  # NFS 共享目录的绝对路径

```

部署

```
kubectl apply -f /root/test/nfs-deployment.yaml 
```

**验证部署**： 使用以下命令检查 `nfs-client-provisioner` 是否正常运行：

```
kubectl get deployments
```

![image-20241223215115754](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223215115754.png)

```shell
[root@k8s-master ~]# kubectl get pods -l app=nfs-provisioner
NAME                               READY   STATUS    RESTARTS   AGE
nfs-provisioner-565bb65b98-qvv6v   1/1     Running   0          2m12s
```

![image-20241223215156846](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223215156846.png)

### 创建storageclass（sc）

```
vim /root/test/nfs-storageclass.yaml
```

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: nfs
provisioner: example.com/nfs # 指定NFS供应商名称,和上面对应上
```

```
kubectl apply -f /root/test/nfs-storageclass.yaml 
```

```
kubectl get sc
```

![image-20241223215536384](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223215536384.png)

### 创建PVC

```
vim /root/test/nfs-pvc.yaml
```

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: storageclass-pvc-demo  # PVC 名称
spec:
  accessModes:
    - ReadWriteMany  # 访问模式，ReadWriteMany 表示多个 Pod 可以同时读写
  resources:
    requests:
      storage: 1Gi  # 请求的存储大小
  storageClassName: nfs  # 指定使用 StorageClass 的名称，这里使用了之前创建的 nfs StorageClass

```

将这个配置保存后，可以通过以下命令应用：

```
kubectl apply -f /root/test/nfs-pvc.yaml
```

查看pv，pvc

```
kubectl get pv,pvc
```

![image-20241223215857709](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223215857709.png)

### 创建验证deployment

```
vim /root/test/nfs-test.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-deployment-demo  # Deployment 名称
  labels:
    type: nfs  # 标签，便于识别
spec:
  replicas: 1  # 副本数，设置为1个 Pod，你可以根据需要修改
  selector:
    matchLabels:
      type: nfs  # 用于选择匹配的 Pod
  template:
    metadata:
      labels:
        type: nfs  # Pod 标签
    spec:
      volumes:
        - name: nfs-storage  # 卷名称
          persistentVolumeClaim:
            claimName: storageclass-pvc-demo  # 指定 PVC 的名称
      containers:
        - name: nfs-pod-demo  # 容器名称
          image: nginx：1.18  # 容器镜像
          imagePullPolicy: IfNotPresent  # 镜像拉取策略
          volumeMounts:
            - name: nfs-storage  # 卷名称，确保与 volumes 部分一致
              mountPath: /usr/share/nginx/html  # 容器内的挂载路径
```

将这个配置保存为 YAML 文件后，可以通过以下命令部署：

```
kubectl apply -f /root/test/nfs-test.yaml
```

查看deployment

```
kubectl get deploy -A
```

![image-20241223220827648](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223220827648.png)

查看pod

```yaml
kubectl get pods -l type=nfs
```

![image-20241223220923562](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223220923562.png)



### 测试验证

查看nfs服务器共享路径下的pvc

![image-20241223221031768](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223221031768.png)

接下来直接在上面的pvc修改配置，就等于修改pod容器里面的/usr/share/nginx/html/下的内容

```shell
echo "nfs pv test" > /mnt/nfs_share/default-storageclass-pvc-demo-pvc-3efb9f3f-22f5-4596-9113-0071def415f7/index.html
```

![image-20241223221335091](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223221335091.png)

先查看刚刚创建的pod的ip地址

```shell
[root@k8s-master ~]# kubectl get pods -l type=nfs -owide
NAME                        READY   STATUS    RESTARTS   AGE     IP              NODE          NOMINATED NODE   READINESS GATES
nfs-test-6f896f7b7b-tjmhf   1/1     Running   0          5m40s   10.244.126.12   k8s-worker2   <none>           <none>
```

测试index.html网页内容

![image-20241223221437056](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241223221437056.png)





# 17、调度

## 概述

一个Pod在哪个Node节点上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程是不受人工控制的。但

是在实际使用中，这并不满足的需求，因为很多情况下，我们想控制某些Pod到达某些节点上，那么应该怎么做呢？这就要

求了解kubernetes对Pod的调度规则。

kubernetes提供了四大类调度方式：

* 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
* 定向调度：NodeName、NodeSelector
* 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity
* 污点（容忍）调度：Taints、Toleration



## 定向调度

定向调度，指的是利用在pod上声明nodeName或者nodeSelector，以此将Pod调度到期望的node节点上。注

意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败

而已。

### **NodeName调度**

NodeName用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的

调度逻辑，直接将Pod调度到指定名称的节点

举例子：

1.查看k8s集群节点名称

```bash
[root@k8s-master ~]# kubectl get nodes
NAME          STATUS   ROLES           AGE   VERSION
k8s-master    Ready    control-plane   25d   v1.28.0
k8s-worker1   Ready    <none>          25d   v1.28.0
k8s-worker2   Ready    <none>          25d   v1.28.0
```

2.创建测试pod1.yaml文件

```yaml
vim pod1.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers: 
  - name: nginx
    image: nginx:1.18
  nodeName: k8s-worker1 # 指定调度到node1节点上
```

3.创建测试pod1

```
 kubectl apply -f pod1.yaml 
```

4.查看pod被调度到哪里

```bash
[root@k8s-master ~]# kubectl get pods pod1 -owide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
pod1   1/1     Running   0          31s   10.244.194.94   k8s-worker1   <none>           <none>
```

可以看到被调度到worker1节点并运行成功！



###  **NodeSelector调度**

NodeSelector用于将pod调度到添加了指定标签的node节点上。它是通过kubernetes的label-selector机制实现

的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找出目标node，

然后将pod调度到目标节点，该匹配规则是强制约束。

举例子：

1.给节点添加标签

我下面给k8s-worker2节点添加标签   

```bash
kubectl label nodes k8s-worker2 node2=node2
```

2.查看标签是否成功添加

```bash
kubectl get nodes k8s-worker2 --show-labels    #查看worker2节点的标签
NAME          STATUS   ROLES    AGE   VERSION   LABELS
k8s-worker2   Ready    <none>   25d   v1.28.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker2,kubernetes.io/os=linux,node2=node2
```

3.创建pod2.yaml测试文件

```yaml
vim pod2.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: nginx
    image: nginx:1.18
  nodeSelector:
    node2: node2  #指定调度到具有node2=node2标签节点上
```

4.创建pod2测试

```shell
kubectl apply -f pod2.yaml 
```

5.查看pod调度到哪里

```bash
[root@k8s-master ~]# kubectl get pods pod2 -owide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
pod2   1/1     Running   0          35s   10.244.126.17   k8s-worker2   <none>           <none>
```





## 污点与容忍

### 概述

l **污点（Taints）：**定义在节点上，用于拒绝Pod调度到此节点，除非该Pod具有该节点上的污点容忍度。被标记

有Taints的节点并不是故障节点。

l **容忍（Tolerations）**：定义在Pod上，用于配置Pod可容忍的节点污点，K8S调度器只能将Pod调度到该Pod能

够容忍的污点的节点上。

![image-20241225200808424](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241225200808424.png)

### 污点配置

污点的格式为：**key=value:effect**

key和value是污点的标签，effect描述污点的作用，支持如下三个选项：

* NoSchedule：没有配置此污点容忍度的新Pod不能调度到此节点，节点上现存的Pod不受影响。
* PreferNoSchedule：没有配置此污点容忍度的新Pod尽量不要调度到此节点，如果找不到合适的节点，依然会调度到此节点。

* NoExecute：没有配置此污点容忍度的新Pod对象不能调度到此节点，节点上现存的Pod会被驱逐。

1.首先先查看节点的taint

例如我查看k8s-master的污点，可以看到master默认不会调度pod

```shell
[root@k8s-master ~]# kubectl describe nodes k8s-master | grep Taint   
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

[root@k8s-master ~]# kubectl describe nodes k8s-worker1 | grep Taint
Taints:             <none>

[root@k8s-master ~]# kubectl describe nodes k8s-worker2 | grep Taint
Taints:             <none>

```

2.给k8s-worker1节点打污点

```shell
kubectl taint nodes k8s-worker1 tag=test:NoSchedule
```

3.查看是否打标签成功

```shell
[root@k8s-master ~]# kubectl describe nodes k8s-worker1 | grep Taint
Taints:             tag=test:NoSchedule
```

4.创建pod3测试是否能调度到worker1节点上

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod3
spec:
  containers:
  - name: nginx
    image: nginx:1.18
```

```bash
[root@k8s-master ~]# kubectl get pods pod3 -owide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
pod3   1/1     Running   0          9s    10.244.126.19   k8s-worker2   <none>           <none>
```

pod3将不会调度到worker1节点上



5.如何删除污点

只需要键+effect+`-`

```bash
kubectl taint nodes k8s-worker1 tag:NoSchedule-

或者直接删除所有污点

kubectl taint nodes k8s-worker1 tag-
```

### 容忍配置

在node上添加污点用于拒绝pod调度上来，但是如果就是想将一个pod调度到一个有污点的node上去，这

时候应该怎么做呢？这就要使用到容忍。

**容忍度操作符**

在Pod上定义容忍度时，它支持两种操作符：Equal和Exists。

• **Equal：**容忍度与污点必须在key、value和effect三者完全匹配。

• **Exists：**容忍度与污点必须在key和effect二者完全匹配，容忍度中的value字段要使用空值。

![image-20241225203755677](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241225203755677.png)



![image-20241225203803203](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241225203803203.png)

举例子

1、给节点一、节点二设置污点

```shell
kubectl taint nodes k8s-worker1 tag=test:NoSchedule
kubectl taint nodes k8s-worker2 tag=test:NoSchedule

[root@k8s-master ~]# kubectl describe nodes k8s-worker1 | grep Taint
Taints:             tag=test:NoSchedule
[root@k8s-master ~]# kubectl describe nodes k8s-worker2 | grep Taint
Taints:             tag=test:NoSchedule
```

2.创建一个pod没有添加容忍的

可以发现pod调度失败，因为三个节点都添加了污点，而pod没有添加容忍度，所以没有可用的节点

3.创建pod4添加容忍

```yaml
vim pod4.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod4
spec:
  containers:
  - name: nginx
    image: nginx:1.18
  tolerations:  # 添加容忍
  - key: "tag"  # 要容忍的污点的key
    operator: "Equal"  # 操作符
    value: "test"  # 容忍的污点的value
    effect: "NoSchedule"  # 添加容忍的规则，这里必须和标记的污点规则相同
```

3.创建pod4测试

```shell
kubectl apply -f pod4.yaml 
```

4.查询pod状态

```bash
[root@k8s-master ~]# kubectl get pods pod4 -owide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
pod4   1/1     Running   0          32s   10.244.194.95   k8s-worker1   <none>           <none>
```



# 18、节点亲和性调度

## 概述

为了更灵活更复杂的调度方式，kubernetes还提供了一种亲和性调度（Affinity）。它在NodeSelector的基础之上的进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活。

![image-20241225204619385](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241225204619385.png)

## **亲和性类型**

Affinity主要分为三类：

• **nodeAffinity(node亲和性）:** 以node为目标，解决pod可以调度到哪些node的问题

• **podAffinity(pod亲和性) :** 以pod为目标，解决pod可以和哪些已存在的pod部署在同一个拓扑域中的问题

• **podAntiAffinity(pod反亲和性) :** 以pod为目标，解决pod不能和哪些已存在pod部署在同一个拓扑域中的问题

## **亲和性使用场景**

**亲和性：**如果两个应用频繁交互，那就有必要利用亲和性让两个应用的尽可能的靠近，这样可以减少因网络通信而带来的性能损耗。

**反亲和性：**当应用的采用多副本部署时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性。

## **亲和性表达方式**

不管是Node亲和 还是Pod亲和，他们都有2种亲和性表达方式：

**RequiredDuringSchedulingIgnoredDuringExecution**：是硬亲和的方式，必须满足指定的规则才可以把Pod调度到该Node上。这里注意Required这个词，中文意思必须的。

**PreferredDuringSchedulingIgnoredDuringExecution**：是软亲和的方式，强调优先满足某个规则，然后根据优先的规则，将Pod调度到节点上。这里注意Preferred这个词，中文意思是首选，用来说明选择规则的优先级，确实比较合适。

说明：**IgnoredDuringExecution**：已经在节点上运行的Pod不需要满足定义的规则，即使去除节点上的某个标签，那些需要节点包含该标签的Pod依旧会在该节点上运行。



## NodeAffinity 节点的可配置项：

### 1. `pod.spec.affinity.nodeAffinity`
`nodeAffinity` 用于指定 Pod 的节点亲和性规则。它有两个主要部分：  
- **requiredDuringSchedulingIgnoredDuringExecution**：  
  必须满足指定的所有规则才能调度到节点，相当于硬限制。

- **preferredDuringSchedulingIgnoredDuringExecution**：  
  优先调度到满足指定规则的节点，相当于软限制（倾向）。

---

### 2. `requiredDuringSchedulingIgnoredDuringExecution` (硬限制)
用于定义必须满足的节点选择规则。  
- **nodeSelectorTerms**：节点选择条件的列表。

  - **matchFields**：  
    按节点字段列出的节点选择器要求列表。

  - **matchExpressions**：  
    按节点标签列出的节点选择器要求列表（推荐）。  
    - **key**：节点标签的键。
    - **values**：键对应的值。
    - **operator**：关系符，支持以下选项：
      - `Exists`：节点必须包含此键。
      - `DoesNotExist`：节点必须不包含此键。
      - `In`：节点标签的值必须在指定的值列表中。
      - `NotIn`：节点标签的值不能在指定的值列表中。
      - `Gt`：节点标签的值必须大于指定值。
      - `Lt`：节点标签的值必须小于指定值。

---

### 3. `preferredDuringSchedulingIgnoredDuringExecution` (软限制，倾向)
用于定义优先调度到满足指定规则的节点。  
- **preference**：节点选择器项，通常与相应的权重相关联。  
  - **matchFields**：  
    按节点字段列出的节点选择器要求列表。

  - **matchExpressions**：  
    按节点标签列出的节点选择器要求列表（推荐）。  
    - **key**：节点标签的键。
    - **values**：键对应的值。
    - **operator**：关系符，支持以下选项：
      - `In`：节点标签的值必须在指定的值列表中。
      - `NotIn`：节点标签的值不能在指定的值列表中。
      - `Exists`：节点必须包含此键。
      - `DoesNotExist`：节点必须不包含此键。
      - `Gt`：节点标签的值必须大于指定值。
      - `Lt`：节点标签的值必须小于指定值。

  - **weight**：倾向权重，范围为 1-100。值越高，优先级越高。

## 演示硬限制

1.首先删掉之前配置的污点

```
kubectl taint nodes k8s-worker2 tag-
kubectl taint nodes k8s-worker1 tag-
```

2..首先查看节点的标签

```shell
[root@k8s-master ~]# kubectl get nodes k8s-worker2 --show-labels
NAME          STATUS   ROLES    AGE   VERSION   LABELS
k8s-worker2   Ready    <none>   25d   v1.28.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-worker2,kubernetes.io/os=linux,node2=node2
```

3..创建pod5.yaml测试

```yaml
vim pod5.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod5
spec:
  containers:
  - name: nginx
    image: nginx:1.18
  affinity:  # 亲和性设置
    nodeAffinity:  # 设置 node 亲和性
      requiredDuringSchedulingIgnoredDuringExecution:  # 硬限制
        nodeSelectorTerms:  # 节点选择条件列表
        - matchExpressions:  # 匹配 node2 的值在 ["node2"] 中的标签
          - key: node2
            operator: In
            values: ["node2"]

```

4.创建pod5并查看状态

```shell
kubectl apply -f pod5.yaml 

[root@k8s-master ~]# kubectl get pods pod5 -owide
NAME   READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
pod5   1/1     Running   0          18s   10.244.126.20   k8s-worker2   <none>           <none>


[root@k8s-master ~]# kubectl describe pod pod5
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  100s  default-scheduler  Successfully assigned default/pod5 to k8s-worker2
  Normal  Pulled     99s   kubelet            Container image "nginx:1.18" already present on machine
  Normal  Created    99s   kubelet            Created container nginx
  Normal  Started    99s   kubelet            Started container nginx

```

可以看到已经调度到k8s-master2节点上了

## 演示软限制

创建pod6.yaml测试

```yaml
vim pod6.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod6
spec:
  containers:
  - name: nginx
    image: nginx:1.18
  affinity:  # 亲和性设置
    nodeAffinity:  # 设置 node 亲和性
      preferredDuringSchedulingIgnoredDuringExecution:  # 软限制
        - weight: 1
          preference:
            matchExpressions:  # 匹配 node2 的值在 ["yyy", "zzz"] 中的标签（当前环境没有）
              - key: node2
                operator: In
                values: ["yyy", "zzz"]       #注意这里的标签值可是跟我设置的不一样，我设置的value是node2          
```

查看pod状态

```bash
[root@k8s-master ~]# kubectl apply -f pod6.yaml 
pod/pod6 created
[root@k8s-master ~]# kubectl get pods pod6 -owide
NAME   READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
pod6   1/1     Running   0          9s    10.244.126.8   k8s-worker2   <none>           <none>
```

可见软限制就是选择最佳节点去进行调度（也可以理解为模糊匹配）



# 19、pod亲和性调度

**PodAffinity**主要实现以运行的Pod为参照，实现让新创建的Pod跟参照pod在一个区域的功能。比如Pod1跟随Pod2，Pod2被调度到B节点，那么Pod1也被调度到B节点。

简单来说就是：控制 Pod 如何根据其他 Pod 的位置来选择调度的节点。



举例子：

1.首先我们先创建一个pod参照

```yaml
vim pod7.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-target
  labels:
    podenv: test #设置标签
spec:
  containers: 
  - name: nginx
    image: nginx:1.18
  nodeName: k8s-worker1 # 将目标pod名确指定到k8s-worker1上
```

2.查询参照pod的状态

```bash
[root@k8s-master ~]# kubectl get pods pod-target -owide
NAME         READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
pod-target   1/1     Running   0          67s   10.244.194.97   k8s-worker1   <none>           <none>
```

可以看见pod-target运行在worker1节点上

接下来我们要根据这个pod-target的标签位置，来调度下面的pod

3.创建调度pod（硬限制）



```yaml
vim pod8.yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required
spec:
  containers:
  - name: nginx
    image: nginx:1.18
  affinity:  # 亲和性设置
    podAffinity:  # 设置 Pod 亲和性
      requiredDuringSchedulingIgnoredDuringExecution:  # 硬限制
        - labelSelector:
            matchExpressions:  # 匹配 podenv 的值在 ["test", "yyy"] 中的标签
              - key: podenv           
                operator: In
                values: ["test", "yyy"]     #要匹配参照pod的value
          topologyKey: kubernetes.io/hostname  # 设置拓扑关键字，表示在同一节点上调度
```

4.查看调度pod的运行状态

```shell
[root@k8s-master ~]# kubectl get pods pod-podaffinity-required -owide
NAME                       READY   STATUS    RESTARTS   AGE     IP              NODE          NOMINATED NODE   READINESS GATES
pod-podaffinity-required   1/1     Running   0          4m15s   10.244.194.98   k8s-worker1   <none>           <none>
```

可以看见pod-podaffinity-required是根据pod-target的标签位置去跟随调度的！

接下来，假如我的参照pod去到k8s-worker2节点上的话，我的调度pod是否会跟随过去呢？

* 删除原来的pod7.yaml文件的pod-target

```
kubectl delete -f pod7.yaml 
```

* 修改pod7.yaml文件

```
  nodeName: k8s-worker2 # 将目标pod名确指定到k8s-worker1上
```

* 重新运行pod

```
kubectl apply -f pod7.yaml 
```

* 查看pod状态已经运行节点

```bash
[root@k8s-master ~]# kubectl get pod pod-target -owide
NAME         READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
pod-target   1/1     Running   0          25s   10.244.126.7   k8s-worker2   <none>           <none>
```

* 删除调度pod，重新应用一次调度pod的pod8.yaml文件

  ```bash
  [root@k8s-master ~]# kubectl delete pod pod-podaffinity-required
  pod "pod-podaffinity-required" deleted
  [root@k8s-master ~]# kubectl apply -f pod8.yaml 
  pod/pod-podaffinity-required created
  ```

* 查看调度pod的状态以及节点位置

```bash
[root@k8s-master ~]# kubectl get pods pod-podaffinity-required -owide
NAME                       READY   STATUS    RESTARTS   AGE   IP             NODE          NOMINATED NODE   READINESS GATES
pod-podaffinity-required   1/1     Running   0          38s   10.244.126.9   k8s-worker2   <none>           <none>
```



# 20、访问控制

## 访问控制概述

Kubernetes作为一个分布式集群的管理工具，保证集群的安全性是其一个重要的任务。所谓的安全性其实就是保证对Kubernetes的各种客户端进行认证和鉴权操作。

 **客户端类型**

在Kubernetes集群中，客户端通常有两类：

• **User Account：**一般是独立于kubernetes之外的其他服务管理的用户账号。

• **Service Account：**kubernetes管理的账号，用于为Pod中的服务进程在访问Kubernetes时提供身份标识。

![image-20241226195606544](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226195606544.png)

###  **认证、授权与准入控制**

ApiServer是访问及管理资源对象的唯一入口。任何一个请求访问

ApiServer，都要经过下面三个流程：

1、**Authentication（认证）**：身份鉴别，只有正确的账号才能够通过认证

2、**Authorization（授权）**： 判断用户是否有权限对访问的资源执行特定的动作

3、**Admission Control**（准入控制）：用于补充授权机制以实现更加精细的访问控制功能。

#### Authentication（认证）

* **HTTP Token认证**：通过一个Token来识别合法用户

  这种认证方式是用一个很长的难以被模仿的字符串--Token来表明客户身份的一种方式。每个Token对应一个用户名，当客户端发起API调用请求时，需要在HTTP Header里放入Token，API Server接到Token后会跟服务器中保存的token进行比对，然后进行用户身份认证的过程。

* **HTTPS证书认证**：基于CA根证书签名的双向数字证书认证方式

  这种认证方式是安全性最高的一种方式，但是同时也是操作起来最麻烦的一种方式

![image-20241226200456978](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226200456978.png)

#### Authorization（授权）

授权发生在认证成功之后，通过认证就可以知道请求用户是谁， 然后Kubernetes会根据事先定义的授权策略来决定用户是否有权限访问，这个过程就称为授权。

每个发送到ApiServer的请求都带上了用户和资源的信息：比如发送请求的用户、请求的路径、请求的动作等，授权就是根据这些信息和授权策略进行比较，如果符合策略，则认为授权通过，否则会返回错误。

API Server目前支持以下几种授权策略：

* AlwaysDeny：表示拒绝所有请求，一般用于测试

* AlwaysAllow：允许接收所有请求，相当于集群不需要授权流程（Kubernetes默认的策略）

* ABAC：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制

* Webhook：通过调用外部REST服务对用户进行授权

* Node：是一种专用模式，用于对kubelet发出的请求进行访问控制

* **RBAC：基于角色的访问控制（kubeadm安装方式下的默认选项）**

#### **Admission Control**（准入控制）

通过了前面的认证和授权之后，还需要经过准入控制处理通过之后，apiserver才会处理这个请求。准入控制是一个可

配置的控制器列表，可以通过在Api-Server上通过命令行设置选择执行哪些准入控制器：

**--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,**

**DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds**

只有当所有的准入控制器都检查通过之后，apiserver才执行该请求，否则返回拒绝。

当前可配置的Admission Control准入控制如下：

• AlwaysAdmit：允许所有请求

• AlwaysDeny：禁止所有请求，一般用于测试

• AlwaysPullImages：在启动容器之前总去下载镜像

• DenyExecOnPrivileged：它会拦截所有想在Privileged Container上执行命令的请求

• ImagePolicyWebhook：这个插件将允许后端的一个Webhook程序来完成admission controller的功能。

• Service Account：实现ServiceAccount实现了自动化

• SecurityContextDeny：这个插件将使用SecurityContext的Pod中的定义全部失效

控制器说明：

• ResourceQuota：用于资源配额管理目的，观察所有请求，确保在namespace上的配额不会超标

• LimitRanger：用于资源限制管理，作用于namespace上，确保对Pod进行资源限制

• InitialResources：为未设置资源请求与限制的Pod，根据其镜像的历史资源的使用情况进行设置

• NamespaceLifecycle：如果尝试在一个不存在的namespace中创建资源对象，则该创建请求将被拒绝。当删除一个

namespace时，系统将会删除该namespace中所有对象。

• DefaultStorageClass：为了实现共享存储的动态供应，为未指定StorageClass或PV的PVC尝试匹配默认的

StorageClass，尽可能减少用户在申请PVC时所需了解的后端存储细节

• DefaultTolerationSeconds：这个插件为那些没有设置forgiveness tolerations并具有notready:NoExecute和

unreachable:NoExecute两种taints的Pod设置默认的“容忍”时间，为5min

• PodSecurityPolicy：这个插件用于在创建或修改Pod时决定是否根据Pod的security context和可用的

PodSecurityPolicy对Pod的安全策略进行控制



## RBAC

### RBAC概述

RBAC(Role-Based Access Control) 基于角色的访问控制，主要是在描述一件事情：给哪些对象授予了哪些权限。

其中涉及到了下面三个概念：

• **对象：**User、Groups（主要面向集群外）、ServiceAccount（主要面向pod）

kubernetes 中账户分为：UserAccounts（用户账户） 和 ServiceAccounts（服务账户） 两种：

**UserAccount** 是给 kubernetes 集群外部用户使用的，如 kubectl 访问 k8s 集群要用useraccount 用户，kubeadm 安装的 k8s，默认的 useraccount 用户是 kubernetes-admin。

**ServiceAccount** 是 Pod 使用的账号，一般情况下pod等资源会自动创建对应权限的serviceaccout,使其拥有和apiserver通信的权限，这个权限是最低权限，如果一个Pod没有指定ServiceAccount，那么将自动关联一个默认的ServiceAccount。系统默认的default ServiceAccount包含在命名空间中，它具有最低权限，并且可以与API Server进行交互。

• **角色：**代表着一组定义在资源上的可操作动作(权限)的集合

• **绑定：**将定义好的角色跟用户绑定在一起

注意RoleBinding就是讲对象和角色进行绑定

![image-20241226201939068](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226201939068.png)



### RBAC4个顶级资源对象

RBAC引入了4个顶级资源对象：

• **Role、ClusterRole：**角色，用于指定一组权限

• **RoleBinding、ClusterRoleBinding：**角色绑定，用于将角色（权限）赋予给对象

####  **Role** 

一个角色就是一组权限的集合，这里的权限都是许可形式的（白名单）。

Role只能对命名空间内的资源进行授权，需要指定nameapce。

yaml文件介绍：

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: test       # 一定要指定角色所在的命名空间，要确保命名空间已存在
  name: test-role       # 角色的名称
rules:
  - apiGroups: [""]     # 支持的 API 组列表，空字符串表示核心 API 群
    resources: ["pods"] # 支持的资源对象列表
    verbs: ["get","watch","list"]     # 允许的操作方法          
```

####  **ClusterRole**

ClusterRole可以对集群范围内资源、跨namespaces的范围资源、非资源类型进行授权。

yaml文件介绍：

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1  # 使用 v1 版本
metadata:
  name: test-clusterrole                # ClusterRole 的名称
rules:
  - apiGroups: ["" ]                    # 空字符串表示核心 API 群
    resources: ["pods"]                 # 资源对象：pods
    verbs: ["get", "watch", "list"]     # 允许的操作：get、watch、list
```



------

配置角色项 `rules` 参数说明：

**1. apiGroups: 支持的 API 组列表**

- `""`: 核心 API 群（例如 Pods、Services 等）
- `"apps"`: 包含应用相关的资源，如 Deployments、StatefulSets 等
- `"autoscaling"`: 包含自动扩展相关资源，如 HorizontalPodAutoscalers 等
- `"batch"`: 包含批处理作业相关资源，如 Jobs、CronJobs 等

**2. resources: 支持的资源对象列表**

- `"services"`: 服务资源
- `"endpoints"`: 端点资源
- `"pods"`: Pod 资源
- `"secrets"`: 密钥资源
- `"configmaps"`: 配置映射资源
- `"crontabs"`: 定时任务资源
- `"deployments"`: 部署资源
- `"jobs"`: 作业资源
- `"nodes"`: 节点资源
- `"rolebindings"`: 角色绑定资源
- `"clusterroles"`: 集群角色资源
- `"daemonsets"`: 守护进程集资源
- `"replicasets"`: 副本集资源
- `"statefulsets"`: 有状态集资源
- `"horizontalpodautoscalers"`: 水平 Pod 自动扩展器资源
- `"replicationcontrollers"`: 副本控制器资源
- `"cronjobs"`: 定时作业资源

**3. verbs: 对资源对象的操作方法列表**

- `"get"`: 获取资源
- `"list"`: 列出资源
- `"watch"`: 监听资源变化
- `"create"`: 创建资源
- `"update"`: 更新资源
- `"patch"`: 修补资源
- `"delete"`: 删除资源
- `"exec"`: 执行命令

 **RoleBinding、ClusterRoleBinding**

角色绑定用来把一个角色绑定到一个目标对象上，绑定目标可以是User、Group或者ServiceAccount。

#### **RoleBinding**

RoleBinding可以将同一namespace中的subject绑定到某个Role下，则此subject即具有该Role定义的权限。

```yaml
kind: RoleBinding                # 资源类型：RoleBinding
apiVersion: rbac.authorization.k8s.io/v1  # API 版本，建议使用 v1 版本
metadata:
  name: test-role-binding        # 绑定的名称
  namespace: test                # 绑定的命名空间
subjects: 
  - kind: User                   # 绑定的主体类型，用户（User）
    name: zhangsan               # 绑定的用户名称
    apiGroup: rbac.authorization.k8s.io  # apiGroup，用于指定 RoleBinding 的 API 组
roleRef:
  kind: Role                     # 绑定的角色类型，Role 表示命名空间内的角色
  name: test-role                # 绑定的角色名称
  apiGroup: rbac.authorization.k8s.io  # 指定角色的 API 组
```

#### **ClusterRoleBinding**

ClusterRoleBinding在整个集群级别和所有namespaces将特定的subject与ClusterRole绑定，授予权限。

yaml文件介绍：

```yaml
kind: ClusterRoleBinding                # 资源类型：ClusterRoleBinding，集群范围内的角色绑定
apiVersion: rbac.authorization.k8s.io/v1 # API 版本，建议使用 v1 版本
metadata:
  name: test-clusterrole-binding        # 绑定的名称
subjects: 
  - kind: User                          # 绑定的主体类型，用户（User）
    name: test                          # 绑定的用户名称
    apiGroup: rbac.authorization.k8s.io # apiGroup，用于指定 ClusterRoleBinding 的 API 组
roleRef:
  kind: ClusterRole                    # 绑定的角色类型，ClusterRole 表示集群级别的角色
  name: test-clusterrole               # 绑定的角色名称
  apiGroup: rbac.authorization.k8s.io  # 指定角色所在的 API 组
```



### RBAC配置演示

目的：接下来我将创建用户user01，只能查看test名称空间下的pod资源

1.创建证书

```bash
# cd /etc/kubernetes/pki/
# (umask 077; openssl genrsa -out user01.key 2048)
```

![image-20241226211354155](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226211354155.png)

2.创建证书申请

```bash
# openssl req -new -key user01.key -out user01.csr -subj "/CN=user01/O=test"
```

3.签署证书

```bash
# openssl x509 -req -in user01.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out user01.crt -days 365
```

![image-20241226211430509](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226211430509.png)

4.设置集群、用户、上下文信息

```bash
# kubectl config set-cluster kubernetes --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt --server=https://192.168.100.13:6443
# kubectl config set-credentials user01 --embed-certs=true --client-certificate=/etc/kubernetes/pki/user01.crt --client-key=/etc/kubernetes/pki/user01.key
# kubectl config set-context user01@kubernetes --cluster=kubernetes --user=user01
```

![image-20241226211547203](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226211547203.png)

5.创建命名空间

```
kubectl create ns test
```



6.创建Role和RoleBinding，为user01用户授权

vim user01-role.yaml

```yaml
# 定义 Role 资源，指定在命名空间 'test' 下的角色
kind: Role
apiVersion: rbac.authorization.k8s.io/v1  # 使用 v1 版本
metadata:
  namespace: test  # 角色所属的命名空间
  name: test-role  # 角色的名称
rules:
  - apiGroups: [""]  # 空字符串表示核心 API 组
    resources: ["pods"]  # 资源对象是 pods
    verbs: ["get", "watch", "list"]  # 允许的操作：获取、监听、列出

---
# 定义 RoleBinding 资源，将角色绑定到指定的用户
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1  # 使用 v1 版本
metadata:
  name: test-role-binding  # 绑定名称
  namespace: test  # 角色绑定的命名空间
subjects:
  - kind: User  # 绑定的主体类型，用户（User）
    name: user01  # 绑定的用户名称，已替换为 user01
    apiGroup: rbac.authorization.k8s.io  # API 组指定为 rbac.authorization.k8s.io
roleRef:
  kind: Role  # 绑定的角色类型，Role 表示命名空间级别的角色
  name: test-role  # 绑定的角色名称，指向 'test-role'
  apiGroup: rbac.authorization.k8s.io  # 角色所属的 API 组
```

应用yaml文件

```shell
[root@k8s-master pki]# kubectl apply -f user01-role.yaml 
role.rbac.authorization.k8s.io/test-role created
rolebinding.rbac.authorization.k8s.io/test-role-binding created
```

检查role，rolebiding

![image-20241226212152134](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226212152134.png)

7.创建测试资源对象

```shell
#创建pod
[root@k8s-master ~]# kubectl run nginx01 --image=nginx:1.18 -n test
pod/nginx01 created
#创建deployment
[root@k8s-master ~]# kubectl create deploy nginx02 --image=nginx:1.18 -n test
deployment.apps/nginx02 created
```

8.切换用户测试

切换到用户user01

```shell
[root@k8s-master ~]# kubectl config use-context user01@kubernetes
Switched to context "user01@kubernetes".

#实验完了要切换回来k8s集群管理用户
[root@k8s-master ~]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```

测试

```
kubectl get pod,deploy -n test
```

![image-20241226212440462](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241226212440462.png)

结果：可见我切换到user01用户，只能查看到test命名空间下的pod资源，不能查看deployment资源，因为在上面只授权了Role角色可以查看pods信息资源！！



### NetworkPolicy

#### 概述

Kubernetes网络策略(NetworkPolicy)是一个资源对象，主要用于定义Pod之间的流量控制，其实现了一个基于标签的选择器模型，允许管理员通过网络策略规则限制对Pod的流量访问。（简单来说就是默认pod与pod之间的访问是敞开的，NetworkPolicy的作用就是限制pod与pod之间的访问，可以控制哪个pod可以跟哪个pod交流）

网络策略(NetworkPolicy)是以Pod为单位进行授权的，因此，只有当所有的Pod都通过了网络策略时，才能够接收到其他Pod发送的流量。这种方式极大提高了网络的安全性。该资源对象只配置策略规则，还需要一个策略控制器来实现具体策略规则。第三方网络组件，如 Calico、Weave Net 等，都提供策略控制器来实现 NetworkPolicy。大家熟知的Flannel是不支持网络策略的。

![image-20241228112606409](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241228112606409.png)

Pod 有两种隔离: 出口的隔离和入口的隔离：

• **出站（Egress）**

• **入站（Ingress）**

为了允许两个 Pods 之间的网络数据流，源端 Pod 上的出站（Egress）规则和 目标端 Pod 上的入站（Ingress）规则都需要允许该流量。 如果源端的出站（Egress）规则或目标端的入站（Ingress）规则拒绝该流量， 则流量将被拒绝。

#### yaml配置示例

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy  # 网络策略的名称
spec:
  # podSelector: 定义网络策略适用的 Pod，通过标签选择特定的 Pod
  podSelector:
    matchLabels:
      role: db  # 仅应用于标签为 "role=db" 的 Pod
  
  # policyTypes: 定义启用哪些类型的网络策略（入站规则 Ingress 和出站规则 Egress）
  policyTypes:
    - Ingress  # 启用入站规则
    - Egress   # 启用出站规则
  
  # ingress: 定义入站规则（控制外部流量如何访问 Pod）
  ingress:
    - from:
        # 允许来自特定 IP 范围的流量
        - ipBlock:
            cidr: 172.17.0.0/16  # 允许来自 IP 范围 172.17.0.0/16 的流量
            except:
              - 172.17.1.0/24    # 排除来自 IP 范围 172.17.1.0/24 的流量
        
        # 允许来自特定 Namespace 中符合标签选择的 Pod 的流量
        - namespaceSelector:
            matchLabels:
              project: myproject  # 允许来自标签为 "project=myproject" 的 Namespace
        
        # 允许来自特定标签的 Pod 的流量
        - podSelector:
            matchLabels:
              role: frontend  # 允许来自标签为 "role=frontend" 的 Pod
        
      ports:
        - protocol: TCP    # 允许 TCP 协议的流量
          port: 6379       # 允许访问 6379 端口（如 Redis 服务）
  
  # egress: 定义出站规则（控制 Pod 向外部发起的流量）
  egress:
    - to:
        # 允许发送流量到特定 IP 范围
        - ipBlock:
            cidr: 10.0.0.0/24  # 允许发送到 IP 范围 10.0.0.0/24 的流量
      ports:
        - protocol: TCP    # 允许 TCP 协议的流量
          port: 5978       # 允许访问 5978 端口
```

配置项说明：

• **PodSelector:** 指定被此 NetworkPolicy 影响的 Pod。此处匹配 Label 为 "role=db" 的 Pod。

• **PolicyTypes:** 规定所需的网络策略类型。此处包括 Ingress 和 Egress。

• **Ingress:** 定义允许从指定来源（IP 地址范围、命名空间或 Pod）和端口接收流量的规则。此处仅允许 TCP 流量访问端口 6379。

• **Egress:** 定义允许发送到指定目标（IP 地址范围）和端口的流量的规则。此处仅允许 TCP 流量发送到端口 5978 的目标 IP 地址范围为 10.0.0.0/24。





#### 实践

创建一个名为 `allow-port-from-namespace` 的新 **`NetworkPolicy`**，以允许 **`corp-net`** 命名空间中的 Pods 连接到同一命名空间中其他 Pods 的端口 `9200`

首先我们要知道不设置networkpolicy前，所有pod的互相访问都是畅通的，现在我们添加networkpolicy，限制了只有在同一个命名空间corp-net下的pods可以互相访问另一个pod的9200端口

想法：创建三个pod，分别为pod1、pod2、pod3；而pod1 pod2是在命名空间corp-net下，而pod3是在默认命名空间下创建；我用busybox:1.30镜像创建pod，然后pod2、pod3去访问pod1的9200端口去测试即可，目的是pod2访问pod1的9200端口成功，但是pod3访问pod1失败

##### 1.创建命名空间

```
kubectl create ns corp-net
```

##### 2.创建networkpolicy

vim allow-port-from-namespace.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: corp-net  # 应用此策略的命名空间
spec:
  podSelector: {}  # 选择所有的 Pods（空的选择器表示所有 Pods）
  policyTypes:
    - Ingress  # 规则应用于入站流量
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: corp-net  # 允许来自同一命名空间的 Pods
      ports:
        - protocol: TCP
          port: 9200  # 允许访问端口 9200

```

然后应用

```shell
 kubectl apply -f allow-port-from-namespace.yaml 
```

然后查看创建的networkpolicy

```
kubectl get networkpolicy -n corp-net
```

![image-20241228123139768](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241228123139768.png)

##### 3.为命名空间corp-net添加标签

```shell
kubectl label namespace corp-net name=corp-net

#查看是否添加成功
[root@k8s-master ~]# kubectl get ns --show-labels
NAME              STATUS   AGE     LABELS
corp-net          Active   3h35m   kubernetes.io/metadata.name=corp-net,name=corp-net
```



##### 4.创建相关pod资源

我这里为了方便，我就直接一个yaml文件创建三个pod了

vim pod-ns-networkpolicy1.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: corp-net
  labels:
    app: pod1  # 为 pod1 添加标签
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["sleep", "3600"]

---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: corp-net
  labels:
    app: pod2  # 为 pod2 添加标签
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["sleep", "3600"]

---
apiVersion: v1
kind: Pod
metadata:
  name: pod3
  namespace: default
  labels:
    app: pod3  # 为 pod3 添加标签
spec:
  containers:
  - name: busybox
    image: busybox:1.30
    command: ["sleep", "3600"]

```

然后应用yaml文件

```
kubectl apply -f pod-ns-networkpolicy1.yaml
```

然后查看pod资源

```shell
[root@k8s-master ~]# kubectl get pods -A | grep pod
corp-net      pod1                                       1/1     Running            0              3m
corp-net      pod2                                       1/1     Running            0              3m
default       pod3                                       1/1     Running            0              3m
```

##### 5.创建service暴露服务

* pod1

```bash
kubectl expose pod pod1 --name=pod1svc --namespace=corp-net --port=9200 --target-port=9200
```

* pod2

```bash
kubectl expose pod pod2 --name=pod2svc --namespace=corp-net --port=9200 --target-port=9200
```

* pod3

```bash
kubectl expose pod pod3 --name=pod3svc --namespace=default --port=9200 --target-port=9200
```



验证service是否绑定到podml

```
kubectl get endpoints pod1svc -n corp-net
kubectl get endpoints pod2svc -n corp-net
kubectl get endpoints pod3svc -n default
```

然后查看pod的svc资源

```
kubectl get svc -A
```

![image-20241228160929425](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241228160929425.png)

##### 6.测试验证

我们看看我们的pod相对应的ip地址是什么

```shell
[root@k8s-master ~]# kubectl get pod -A -owide | grep pod
corp-net      pod1                                       1/1     Running            0              15m     10.244.194.120   k8s-worker1   <none>           <none>
corp-net      pod2                                       1/1     Running            0              15m     10.244.126.32    k8s-worker2   <none>           <none>
default       pod3                                       1/1     Running            0              15m     10.244.194.123   k8s-worker1   <none>           <none>
```

我现在目的就是pod1可以访问pod2的9200端口，pod3访问不了pod2的9200端口

* 1.进入pod2，通过nc命令让pod2开启监听端口9200

```
kubectl exec -it pod2 -n corp-net -- nc -lkp 9200
```

* 2.进入pod1，通过nc命令访问pod2的9200端口，注意我这里是通过service的名称访问，这样就可以不用通过pod的ip地址来访问了

```
kubectl exec -it pod1 -n corp-net -- nc -zv pod2svc 9200
```

正确结果如下：

![image-20241228160117324](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241228160117324.png)

相反的我也可以通过pod2访问pod1进行测试

![image-20241228160616598](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241228160616598.png)

* 3.重新再pod2开启再次监听9200端口（刷新作用）

```
kubectl exec -it pod2 -n corp-net -- nc -lkp 9200
```

* 4.在pod3访问pod2

```
 kubectl exec -it pod3 -n default -- nc -zv pod2svc 9200
```

结果显示如下：

![image-20241228160318113](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20241228160318113.png)

# 21、prometheus

## 介绍

Prometheus发展速度很快，12年开发完成，16年加入CNCF，成为继K8s之后第二个CNCF托管的项目。Prometheus提供了从指标（metrics）暴露，到指标抓取、存储和可视化，以及最后的监控告警等一系列组件。

### 核心组件

![image-20250102195714015](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102195714015.png)

### **Prometheus 中的 Metrics**

1. 宿主机的监控数据：

① Node Exporter 工具提供，生产环境中常以 DaemonSet 的方式运行在宿主机上

② 抓取节点的负载（Load）、CPU、内存、磁盘以及网络这样的常规信息

2. K8s组件的数据：

包括了各个组件的核心监控指标

3. K8s 核心监控数据（core metrics）：

包括Pod、Node、容器、Service等主要 K8s 核心概念的 Metrics

### Prometheus的工作流程

1. 定期从配置好的 jobs 或者 exporters 中拉取 metrics，或者接收来自 Pushgateway 发过来的metrics，PushGateway支持 Client 主动推送 metrics 到 PushGateway，而 Prometheus 只是定时去 Gateway 上抓取数据。

2. Prometheus server 在本地存储收集到的 metrics，并运行已定义好的 alert.rules，记录新的时间序号列或者向Alertmanager 推送警报

3. Alertmanager 根据配置文件，对接收到的警报进行处理，发出告警

4. 在图形界面中，可视化采集数据，例如 Grafana、自带的 Promdash 以及自身提供的模版引擎等等。

## 安装

prometheus下载地址：https://github.com/prometheus-operator/kube-prometheus

按照官网下载对应版本相对稳定一些

部署的Kubernetes集群版本为Kubernetes 1.28.3，系统使用的CentOS7.9，部署工具使用的sealos，部署方法请查看官文文档：[Sealos Official Documents](https://sealos.run/docs/self-hosting/lifecycle-management/) ，话不多说，来看下Kube-Prometheus的部署：

### 1、下载安装prometheus安装包并解压

```bash
#下载tar包
wget https://github.com/prometheus-operator/kube-prometheus/archive/refs/tags/v0.13.0.tar.gz
#解压
tar -zxvf v0.13.0.tar.gz 
```

### 2、修改镜像地址

 国内无法访问registry.k8s.io，需替换资源清单内带使用仓库镜像的地址

```shell
#首先查看镜像，根据镜像版本去自行寻找国内仓库，然后修改仓库地址
grep "image:" ./*
```

![image-20250102221107997](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102221107997.png)

```bash
cd kube-prometheus-0.13.0/manifests
#修改两个yaml文件，把镜像变成下面这样
```

![image-20250102222756763](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102222756763.png)

### 3、修改service端口类型

service默认使用端口类型为ClusterIP，为方便访问所以修改为NodePort类型，如果打算用Ingress访问，可以不修改。

我这里修改为nodeport类型

#### （1）prometheus-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.46.0
  name: prometheus-k8s
  namespace: monitoring
spec:
  type: NodePort             #新增
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 32501           #新增
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
  selector:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
  sessionAffinity: ClientIP
```

#### （2）grafana-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 9.5.3
  name: grafana
  namespace: monitoring
spec:
  type: NodePort      #新增
  ports:
  - name: http
    port: 3000
    targetPort: http
    nodePort: 32500     #新增
  selector:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
```

#### (3) alertmanager-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/instance: main
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 0.26.0
  name: alertmanager-main
  namespace: monitoring
spec:
  type: NodePort     #新增
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 32503     #新增
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
  selector:
    app.kubernetes.io/component: alert-router
    app.kubernetes.io/instance: main
    app.kubernetes.io/name: alertmanager
    app.kubernetes.io/part-of: kube-prometheus
  sessionAffinity: ClientIP
```

### 4、安装

```shell
cd /root/kube-prometheus-0.13.0

kubectl apply --server-side -f manifests/setup
kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
kubectl apply -f manifests/

#查看资源
kubectl get pod -n monitoring

#删除资源
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

![image-20250102224654396](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102224654396.png)

镜像拉取失败解决方法：

1.尝试重弹pod（delete）

2.尝试手动拉取镜像到本地，然后要修改对应yaml文件，修改镜像地址和修改拉取镜像类型为`IfNotPresent`

### 5、访问

查看svc

```
kubectl get svc -n monitoring
```

![image-20250102224950621](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102224950621.png)

```shell
#prometheus
http://IP:32501
#alertmanager
http://IP:32503
#grafana
http://IP:32500
```

因为prometheus的网络策略有问题，因为只接受来自[编辑prometheus](https://search.bilibili.com/all?from_source=webcommentline_search&keyword=prometheus&seid=9888839542819627357)自己的流量

```
kubectl -n monitoring delete networkpolicy --all
```

访问prometheus

![image-20250102231809347](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102231809347.png)

![image-20250102231829282](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102231829282.png)

访问grafana

默认用户名密码 admin

![image-20250102231913381](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102231913381.png)

然后要确认自己的密码，然后登录

![image-20250102231954408](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102231954408.png)

构建dashboard可以去官网下载模版，也可以使用默认的模版

![image-20250102232644510](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102232644510.png)

![image-20250102232650155](C:\Users\86134\AppData\Roaming\Typora\typora-user-images\image-20250102232650155.png)

