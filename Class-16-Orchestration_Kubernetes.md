# Orchestration & Kubernetes

## 目录

- [Orchestration & Kubernetes](#orchestration--kubernetes)
  - [什么是 Kubernetes (K8s)？](#什么是-kubernetes-k8s)
  - [历史](#历史)
  - [为什么需要 Kubernetes 以及它能做什么？](#为什么需要-kubernetes-以及它能做什么)
  - [了解组件](#了解组件)
    - [Pod](#pod)
    - [Node](#node)
      - [kubelet](#kubelet)
      - [kube-proxy](#kube-proxy)
      - [容器运行时](#容器运行时)
    - [Cluster](#cluster)
    - [控制平面](#控制平面)
      - [kube-apiserver](#kube-apiserver)
      - [etcd](#etcd)
      - [kube-scheduler](#kube-scheduler)
      - [kube-controller-manager](#kube-controller-manager)
      - [cloud-controller-manager](#cloud-controller-manager)
    - [Deployment](#deployment)
  - [Swarm](#swarm)
  - [区别](#区别)
  - [进一步阅读](#进一步阅读)
  - [Steps](#steps)
    - [什么是 Minikube？](#什么是-minikube)
    - [前提条件](#前提条件)
    - [安装](#安装)
    - [目标](#目标)
    - [简单操作](#简单操作)
      - [Cluster vs Service](#cluster-vs-service)
      - [部署 vs 服务](#部署-vs-服务)
    - [清理](#清理)
    - [NodePort](#nodeport)
    - [LoadBalancer](#loadbalancer)
    - [Ingress](#ingress)
    - [使用 YAML 文件部署应用](#使用-yaml-文件部署应用)
    - [CronJob](#cronjob)
    - [配置 ConfigMap](#配置-configmap)
  - [在 GKE 中创建和部署集群](#在-gke-中创建和部署集群)
    - [步骤](#步骤)
      - [任务 1：创建集群并部署示例应用](#任务1创建集群并部署示例应用)
      - [任务 2：从市场创建应用](#任务2从市场创建应用)
      - [任务 3：在集群控制台运行 kubectl 命令](#任务3在集群控制台运行-kubectl-命令)
      - [绑定子域名](#绑定子域名)
      - [删除 GKE 项目](#删除-gke-项目)
  - [Docker 和 Orchestration 相关任务](#docker-和-orchestration-相关任务)
  - [简历优化](#简历优化)

## 什么是 Kubernetes (K8s)？

Kubernetes 是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，支持声明式配置和自动化。它拥有一个快速增长的大型生态系统。Kubernetes 的服务、支持和工具广泛可用。

## 历史

- **传统部署时代**: 早期，组织在物理服务器上运行应用程序，没有办法在物理服务器中定义应用程序的资源边界，导致资源分配问题。
- **虚拟化部署时代**: 为了解决这个问题，引入了虚拟化技术。它允许在单个物理服务器的 CPU 上运行多个虚拟机 (VM)。虚拟化允许应用程序在虚拟机之间隔离，并提供一定程度的安全性，因为一个应用程序的信息不能自由访问另一个应用程序。
- **容器部署时代**: 容器类似于虚拟机，但它们具有放松的隔离属性，以便在应用程序之间共享操作系统 (OS)。因此，容器被认为是轻量级的。

## 为什么需要 Kubernetes 以及它能做什么？

容器是捆绑和运行应用程序的好方法。在生产环境中，需要管理运行应用程序的容器并确保没有停机。例如，如果一个容器宕机，另一个容器需要启动。如果由一个系统来处理这种行为，岂不是更方便？

Kubernetes 为您提供了一个框架来可靠地运行分布式系统。

Kubernetes 提供：

- 服务发现和负载均衡
  - Kubernetes 可以使用 DNS 名称或 IP 地址公开一个容器。如果流量高，Kubernetes 能够负载均衡并分配网络流量，使部署保持稳定。
- 存储编排
  - Kubernetes 允许您自动挂载所选存储系统，如本地存储、公共云提供商等。
- 自动化的滚动更新和回滚
  - 使用 Kubernetes 可以描述已部署容器的期望状态，并以受控速度将实际状态更改为期望状态。例如，可以自动化 Kubernetes 创建新容器、移除现有容器并将所有资源移至新容器。
- 自动打包
  - 提供一个 Kubernetes 集群节点用于运行容器化任务。告知 Kubernetes 每个容器需要多少 CPU 和内存 (RAM)。Kubernetes 可以将容器适配到节点上以充分利用资源。
- 自我修复
  - Kubernetes 会重启失败的容器、替换容器、终止未响应健康检查的容器，直到它们准备好提供服务。
- 秘密和配置管理
  - Kubernetes 允许存储和管理敏感信息，如密码、OAuth 令牌和 SSH 密钥。可以部署和更新秘密和应用配置，而无需重建容器镜像，也无需在堆栈配置中暴露秘密。

## 了解组件

### Pod

Pod 是 Kubernetes 抽象，代表一个或多个应用程序容器（例如 Docker）以及这些容器共享的资源。

这些资源包括：

- 共享存储，作为卷
- 网络，作为唯一的集群 IP 地址
- 关于如何运行每个容器的信息，如容器镜像版本或特定端口

一个 Pod 模拟特定应用程序的 "逻辑主机"，可以包含相对紧密耦合的不同应用程序容器。

### Node

Pod 总是运行在 Node 上。Node 是 Kubernetes 中的工作机器，可以是虚拟机也可以是物理机，取决于集群。

每个 Node 由控制平面管理。Node 可以有多个 Pod，Kubernetes 控制平面自动调度 Pod 到集群中的 Node 上。控制平面的自动调度考虑了每个 Node 上的可用资源。

每个 Kubernetes Node 至少运行：

- Kubelet，一个负责 Kubernetes 控制平面和 Node 之间通信的进程；它管理机器上运行的 Pod 和容器。
- 一个容器运行时（例如 Docker），负责从注册表中拉取容器镜像、解压容器和运行应用程序。

#### kubelet

在集群中的每个节点上运行的代理，确保 Pod 中的容器正在运行。

#### kube-proxy

kube-proxy 是在集群中每个节点上运行的网络代理，维护节点上的网络规则。

#### 容器运行时

容器运行时是负责运行容器的软件。Kubernetes 支持多个容器运行时：Docker、containerd、CRI-O 和任何实现 Kubernetes CRI（容器运行时接口）的实现。

### Cluster

Kubernetes 集群由一组工作机器组成，称为节点，运行容器化应用程序。每个集群至少有一个工作节点。

工作节点托管应用程序工作负载的 Pod。

控制平面管理集群中的工作节点和 Pod。

- 在生产环境中，控制平面通常跨多台计算机和一个集群运行，通常运行多个节点，提供容错性和高可用性。

### 控制平面

控制平面的组件对集群做出全局决策（例如调度），以及检测和响应集群事件（例如在部署的副本字段未满足时启动新 Pod）。

控制平面组件可以在集群中的任何机器上运行。但是，为了简化，设置脚本通常在同一台机器上启动所有控制平面组件，并且不在此机器上运行用户容器。

#### kube-apiserver

API 服务器是 Kubernetes 控制平面的一个组件，公开 Kubernetes API。API 服务器是 Kubernetes 控制平面的前端。

#### etcd

一致且高可用的键值存储，用作 Kubernetes 的后台存储所有集群数据。

参考：https://etcd.io/docs/v3.5/demo/

#### kube-scheduler

控制平面组件，监视新创建的没有分配节点的 Pod，并为它们选择一个节点运行。

#### kube-controller-manager

运行控制器进程的控制平面组件。

#### cloud-controller-manager

嵌入云特定控制逻辑的 Kubernetes 控制平面组件。cloud-controller-manager 允许将集群连接到云提供商的 API，并将与云平台交互的组件与仅与集群交互的组件分开。

cloud-controller

-manager 仅运行特定于云提供商的控制器。如果在本地运行 Kubernetes 或在自己的 PC 内的学习环境中运行集群，集群没有 cloud-controller-manager。

### Deployment

一旦拥有一个运行中的 Kubernetes 集群，就可以在其上部署容器化应用程序。为此，需要创建 Kubernetes 部署配置。部署指示 Kubernetes 如何创建和更新应用程序的实例。一旦创建了部署，Kubernetes 控制平面会调度部署中的应用程序实例在集群中的各个节点上运行。

注意：注意 Kubernetes 集群的价格，使用价格计算器进行估算。

## Swarm

当前版本的 Docker 包括用于本地管理 Docker 引擎集群的 Swarm 模式。使用 Docker CLI 创建 Swarm，部署应用程序服务到 Swarm，并管理 Swarm 行为。

## 区别

参考：https://www.youtube.com/watch?v=9_s3h_GVzZc

## 进一步阅读

- [Docker Swarm vs Kubernetes: A Comparison](https://www.ibm.com/cloud/blog/docker-swarm-vs-kubernetes-a-comparison)
- [Swarm](https://docs.docker.com/engine/swarm/)
- [Swarm Key Concepts](https://docs.docker.com/engine/swarm/key-concepts/)
- [K8s Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [K8s Nodes](https://kubernetes.io/docs/concepts/architecture/nodes/)
- [K8s Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [K8s Autoscaling](https://kubernetes.io/blog/2016/07/autoscaling-in-kubernetes/#benefits-of-autoscaling)

## Steps

### 什么是 Minikube？

Minikube 是本地 Kubernetes，专注于使学习和开发 Kubernetes 变得容易。

所需的仅是 Docker（或类似兼容的）容器或虚拟机环境，并且 Kubernetes 只需一条命令即可启动：`minikube start`

### 前提条件

- 2 个或更多 CPU
- 2GB 的可用内存
- 20GB 的可用磁盘空间
- 互联网连接
- 容器或虚拟机管理器，例如：Docker、QEMU、Hyperkit、Hyper-V、KVM、Parallels、Podman、VirtualBox 或 VMware Fusion/Workstation

### 安装

https://minikube.sigs.k8s.io/docs/start/

### 目标

- 了解集群、节点、部署概念
- 使用 Kubectl 与集群交互

### 简单操作

#### Step 1. 启动本地集群

```sh
minikube start
```

#### Step 2. 启动集群仪表板

```sh
minikube dashboard
```

#### Step 3. 创建部署

```sh
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
```

#### Step 4. 查看部署

```sh
kubectl get deployments
```

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           1m
```

#### Step 5. 查看 Pod

```sh
kubectl get pods
```

```
NAME                          READY     STATUS    RESTARTS   AGE
hello-node-5f76cf6ccf-br9b5   1/1       Running   0
```

#### Step 6. 查看集群事件

```sh
kubectl get events
```

#### Step 7. 查看配置

```sh
kubectl config view
```

#### Step 8. 查看日志

```sh
kubectl logs hello-node-<blah>
```

#### Step 9. 使节点在 Kubernetes 外部可访问

```sh
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

`--type=LoadBalancer` 标志表示希望将服务暴露到集群外部。

#### Step 10. 查看服务

```sh
kubectl get services
```

#### Cluster vs Service

### 集群

集群由一组节点组成，运行容器化应用程序。

- 节点：运行容器的工作机器。
- 控制平面：管理节点和 Pod 的组件。

### 服务

服务是 Kubernetes 中的抽象，定义了一组 Pod 及其访问策略。

- ClusterIP: 仅内部 IP。默认服务类型，仅在集群内可访问。
- NodePort: 在每个节点的静态端口上暴露服务。
- LoadBalancer: 使用云提供商的负载均衡器暴露服务。
- ExternalName: 将服务映射到 DNS 名称。

#### 部署 vs 服务

### 部署

- 目的：主要负责保持指定数量的 Pod 副本运行。提供 Pod 和 ReplicaSet 的声明式更新。
- 组件：
  - Pod: Kubernetes 中最小的可部署单位，托管应用程序容器。
  - ReplicaSet: 确保维持指定数量的 Pod 副本。如果 Pod 失败，ReplicaSet 会创建新的 Pod 来替换它。
- 功能：
  - 滚动更新：允许更新 Pod 模板，并逐步推出更新，确保零停机。
  - 扩展：可以向上或向下扩展 Pod 数量。
  - 版本和回滚：维护版本历史，允许在需要时回滚到以前的版本。
  - Pod 健康：替换变得不健康或无响应的 Pod。
- 使用场景：当需要部署无状态应用程序、管理其期望状态，并在必要时轻松更新或回滚时使用。

### 服务

- 目的：定义一组 Pod 及其访问策略，提供稳定的 IP 地址、DNS 名称和端口，通过这些 Pod 可以访问，即使它们被扩展或替换。
- 服务类型：
  - ClusterIP: 仅内部 IP。默认服务类型，仅在集群内可访问。
  - NodePort: 在每个节点的静态端口上暴露服务。
  - LoadBalancer: 使用云提供商的负载均衡器暴露服务。
  - ExternalName: 将服务映射到 DNS 名称。
- 功能：
  - 稳定端点：提供一致的 IP 地址和 DNS 名称，即使底层 Pod 被替换。
  - 负载分配：将网络流量分配给匹配服务选择器的所有 Pod。
  - 服务发现：Kubernetes 支持两种主要的查找服务模式 - 环境变量和 DNS。
- 使用场景：需要一致的方式访问一组 Pod 或需要外部暴露应用程序时使用。

#### Step 11. 在浏览器中运行应用

```sh
minikube service hello-node
```

#### Step 12. 查看插件列表

```sh
minikube addons list
```

#### 清理

```sh
kubectl delete service hello-node
kubectl delete deployment hello-node
```

### NodePort

#### Step 1. 创建部署并暴露

```sh
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

#### Step 2. 检查状态并转发端口

检查服务是否在运行

```sh
kubectl get services hello-minikube
```

使用 kubectl 转发端口

```sh
kubectl port-forward service/hello-minikube 7080:8080
```

应用现在可在 http://localhost:7080/ 访问。

#### Step 3. 删除

```sh
kubectl delete service hello-minikube
kubectl delete deployment hello-minikube
```

### LoadBalancer

#### Step 1. 创建部署并暴露

```sh
kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080
```

#### Step 2. 打开隧道

启动隧道以为 'balanced' 部署创建可路由的 IP

```sh
minikube tunnel
```

打开 http://localhost:8080/

#### Step 3. 扩展

```sh
kubectl scale --replicas=3 deployment/balanced
kubectl get deployments
```

现在刷新链接几次，查看负载均衡是否生效。

#### Step 4. 清理

```sh
kubectl delete service balanced
kubectl delete deployment balanced
```

### Ingress

#### Step 1. 启用 Ingress 插件

```sh
minikube addons enable ingress
```

#### Step 2. 应用内容

```sh
kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
```

#### Step 3. 检查可用地址

```sh
kubectl get ingress
```

#### Step 4. 打开隧道

```sh
minikube tunnel
```

#### Step 5. 打开另一个标签并访问端点

```sh
curl 127.0

.0.1/foo
```

```sh
Request served by foo-app
...
```

```sh
curl 127.0.0.1/bar
```

```sh
Request served by bar-app
...
```

#### Step 6. 清理

```sh
kubectl delete -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
```

### 使用 YAML 文件部署应用

#### Step 1. 理解 demoapp.yaml

打开 `minikube/config/demoapp.yaml`

```yaml
apiVersion: apps/v1
```

apiVersion: 定义用于创建对象的 Kubernetes API 版本。对于部署，版本通常是 apps/v1。

```yaml
kind: Deployment
```

kind: 指定要管理的资源类型，在这种情况下是部署。部署是允许对 Pod 和 ReplicaSet 进行声明性更新的高级对象。

```yaml
metadata:
  name: demoapp
  namespace: jiangren
```

metadata: 唯一标识部署的信息。

- name: 分配给此部署的名称。
- namespace: 创建此部署的 Kubernetes 命名空间。这里是 `jiangren`。

```yaml
spec:
  replicas: 12
```

spec: 指定部署的期望特性。

- replicas: 要维持的 Pod 副本数量。这里是 12。

```yaml
selector:
  matchLabels:
    app: demoapp
```

selector: 指定部署识别应管理的 Pod 的方式。

- matchLabels: 一个 {key, value} 对的映射。Pod 必须满足所有标签条件才能被视为匹配。这里选择带有标签 `app=demoapp` 的 Pod。

```yaml
template:
  metadata:
    labels:
      app: demoapp
```

template: 定义 Pod 的蓝图。

- metadata.labels: 分配给每个 Pod 的标签。这是至关重要的，因为部署的选择器将查找此标签。

```yaml
spec:
  containers:
    - name: echo
      image: stevesloka/echo-server
      command: ["echo-server"]
      args:
        - --echotext=This is the jiangren test site!
      imagePullPolicy: IfNotPresent
      ports:
        - name: http
          containerPort: 8080
```

spec: 要在每个 Pod 中创建的容器的详细信息。

- containers: 要在 Pod 内运行的容器列表。
- name: 容器的名称。
- image: 使用的 Docker 镜像。
- command: 容器启动后运行的命令。覆盖 Docker 镜像提供的默认命令。
- args: 传递给命令的参数。
- imagePullPolicy: 确定何时拉取镜像。`IfNotPresent` 表示仅在本地不存在时才拉取镜像。
- ports: 指定容器的端口信息。
- name: 端口的名称（供参考）。
- containerPort: 容器监听的端口。

```yaml
kind: Service
```

kind: 指定正在定义的服务，这是一个用于暴露在一组 Pod 上运行的应用程序的构造。

```yaml
type: LoadBalancer
```

type: 指定服务的类型。LoadBalancer 表示它将尝试获取一个外部 IP 来暴露服务。

#### Step 2. 应用内容

这将创建服务并部署应用

```sh
cd minikube/config
kubectl apply -f namespace.yaml
kubectl apply -f demoapp.yaml
```

#### Step 3. 暴露服务

```sh
minikube service demoapp --namespace=jiangren
```

或

```sh
minikube tunnel
```

### CronJob

#### Step 1. 运行示例任务

```sh
kubectl apply -f demojob.yaml
```

#### Step 2. 检查 Pod 状态和日志

```sh
kubectl get pods -n jiangren
kubectl logs hello-28283126-x5tzg -n jiangren
```

#### Step 3. 阅读 demojob.yaml

#### Step 4. 查看 cheatsheet 以与 Pod 交互

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

### 配置 ConfigMap

许多应用程序依赖于初始化或运行时使用的配置。大多数情况下，需要调整分配给配置参数的值。ConfigMaps 是 Kubernetes 的机制，允许将配置数据注入应用程序 Pod 中。

ConfigMap 概念允许将配置工件与镜像内容解耦，以保持容器化应用程序的可移植性。例如，可以下载并运行相同的容器镜像，以便出于本地开发、系统测试或运行实时终端用户工作负载的目的启动容器。

#### 基础

##### Step 1. 创建配置

从命令行创建键值对。

```sh
kubectl create configmap special-config --from-literal=special.how=very
```

也可以从 yaml 文件、env 文件和其他本地文件中创建键值对。

##### Step 2. 创建 Pod 并检查

阅读以下文件以了解如何配置 ConfigMap

```sh
kubectl create -f https://kubernetes.io/examples/pods/pod-single-configmap-env-variable.yaml
```

运行此命令检查注入的配置

```sh
kubectl logs dapi-test-pod
```

##### Step 3. 清理

```sh
kubectl delete configmap special-config
kubectl delete pod dapi-test-pod
```

#### 配置 Redis

##### Step 1. 创建 ConfigMap

```sh
cd minikube/config
kubectl apply -f example-redis-config.yaml
```

检查配置是否存在

```sh
kubectl describe configmap/example-redis-config
```

##### Step 2. 创建 Redis Pod

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/config/redis-pod.yaml
```

然后检查值

```sh
kubectl exec -it redis -- redis-cli
```

检查 `maxmemory`：

```sh
127.0.0.1:6379> CONFIG GET maxmemory
```

检查 `maxmemory-policy`：

```sh
127.0.0.1:6379> CONFIG GET maxmemory-policy
```

## 在 GKE 中创建和部署集群

### 步骤

#### 任务 1：创建集群并部署示例应用

##### Step 1. 创建新项目

https://console.cloud.google.com/projectcreate

##### Step 2. 打开 GKE 集群仪表板

##### Step 3.（第一次用户）可能需要填写信用卡信息才能继续

点击 ENABLE 后，会弹出以下窗口。请填写账单信息并返回。
Google 为首次用户提供 $300 的信用额度。

##### Step 4. 否则，应该看到创建集群

点击 CREATE 并选择 Standard 进行配置

##### Step 4. 配置集群。可以选择 AU 区域以降低集群的延迟。

我们将暂时使用静态版本。也可以尝试 Release Channel 以保持集群版本最新。

##### Step 5. 集群准备好后，点击部署以部署应用程序。

##### Step 6. 部署 Easy CRM: davisjiangren01/easycrm:latest

注意：可以使用自己的镜像。有关带数据库的部署，请参阅 `5.GKE2.md`。

##### Step 7. 继续部署

##### Step 8. 当 Pod 部署后，可以在“Workloads”选项卡中检查 Pod。请点击“Expose”继续创建服务。

##### Step 9. 使用默认 8090 端口并暴露。

##### Step 10. 当服务准备好时，会获得外部端点。

##### Step 11. 在 web 浏览器中访问公共 IP 地址。

#### 任务 2：从市场创建应用

##### Step 1. 在市场中搜索 "wordpress" 或其他内容。

##### Step 2. 点击配置继续

##### Step 3. 输入配置并点击 "Deploy"

##### Step 4. Wordpress 需要一些时间部署

##### Step 5. 准备好后，可以在详细信息页面找到凭据。

##### Step 6. 示例凭据如下。

##### Step 7. 可以在 Wordpress 中找到默认 "Hello world" 页面。

##### Step 8. 使用 "Step 5" 中的凭据登录

##### Step 9. 应该成功登录页面。

##### Step 10. 可以在“外观”选项卡中更改主题。

#### 任务 3：在集群控制台运行 kubectl 命令

转到集群选项卡并打开控制台。尝试运行幻灯片中的 kubectl 命令。

### 绑定子域名

#### Step 1. 申请免费账号

打开 http://freedns.afraid.org/signup/?plan=starter 并申请。

#### Step 2. 注册域名。建议使用 org 域名，因为它的解析和更新速度更快。

http://freedns.afraid.org/domain/registry/

#### Step 3. 选择 "A" 类型，在 "Subdomain" 字段中输入你喜欢的内容，并将目标设置为你网站的 IP 地址。在本例中，使用你的 Wordpress 网站。

http://freedns.afraid.org/subdomain/

#### Step 4. 你应该会看到域名已准备好。

#### Step 5. 在几秒钟内应该能够通过公共 DNS 查看你的网站。

#### Step 6. 将域名提供给你的团队成员，看看他们是否可以成功访问。

### 删除 GKE 项目

应该可以在培训期间保留集群在账户中。可以删除项目以从头开始练习。

#### Step 1. 点击门户右侧的三点图标并选择“项目设置”

#### Step 2. 点击“关闭”

#### Step 3. 输入项目 ID 并点击“关闭”。

## Docker 和 Orchestration 相关任务

- 任务 1. Docker 化任何 web 项目（例如 EasyCRM、P3）并将镜像推送到 dockerhub
- 任务 2. 使用 docker-compose 启动一个 nginx 和 webapp 容器。
- 任务 3. 能否使用 docker swarm 创建几个 webapp 副本并平衡它们之间的负载？
- （可选）如果使用 EasyCRM，任务 3 中会遇到什么问题？（提示：有状态 vs 无状态服务器）

- 延展任务：将应用部署到 Kubernetes
- 问题 1：能否扩展和缩减 webapp？
- 问题 2：能否推出新版本（例如不同标签）的 webapp？步骤是什么？
- 延展问题：能否将 nginx 配置为 webapp 的反向代理和负载均衡器？

  （提示：

  - I. 可能需要学习 nginx 知识并重写 nginx 配置文件以适应集群
  - II. 如何配置 nginx 以检测并负载均衡任何新 Pod 的请求？）

## 简历优化

1. 从 [Sheryl Sandberg 的简历](https://enhancv.com/resume-examples/famous/sheryl-sandberg/) 中学到了什么？列出一些可以引入到自己简历中的要点。
2. 开始重写简历，并在认为有进展时让他人审阅。（提示：使用 Grammarly 修正语法错误；使用项目符号；思考自己的成就而非日常任务）
