# Kubernetes 安装

Kubernetes（K8s）是一个容器化应用部署、扩展和管理自动化的开源系统。

Kubernetes 集群为 Jenkins 增加了一个新的自动化层。Kubernetes 确保资源得到有效利用，确保咱们的服务器和底层基础设施不会过载。Kubernetes 编排容器部署的能力确保 Jenkins 总是有适量的资源可用。

在 Kubernetes 集群上托管 Jenkins 有利于基于 Kubernetes 的部署和基于动态容器的可扩展 Jenkins 代理。在这里，我们看到在 Kubernetes 集群上设置 Jenkins 的分步过程。


## 在 Kubernetes 上设置 Jenkins

要在 Kubernetes 上设置 Jenkins 集群，我们将执行以下操作：

1. [创建一个命名空间](#创建一个命名空间)；

2. [创建一个服务账号](#创建一个服务账号)，具有 Kubernetes 管理权限；

3. 为 Pod 重启时的持久性 Jenkins 数据 [创建本地持久性卷](#创建本地持久性卷)；

4. [创建一个部署 YAML](#创建一个部署-YAML) 并进行部署；

5. [创建一个服务 YAML](#创建一个服务-YAML) 并进行部署；


> 本指南不使用本地持久卷，因为这是一个通用指南。要为 Jenkins 数据使用持久卷，咱们需要创建相关云或本地数据中心的卷并进行配置。

### Jenkins Kubernetes Manifest 文件

这里使用的所有Jenkins Kubernetes Manifest 清单文件都托管在 GitHub 上。如果咱们在复制文件中的 manifest 清单时遇到困难，请克隆该仓库。

```bash
git clone https://github.com/scriptcamp/kubernetes-jenkins
```

请使用这些 GitHub 文件作为参考，并按照接下来的步骤进行。


### Kubernetes 方式 Jenkins 的部署

我们来开始在 Kubernetes 上部署 Jenkins。

**第 1 步**：为 Jenkins 创建一个命名空间。把所有的 DevOps 工具归类为一个独立的命名空间，与其他应用程序分开是很好的；

```bash
kubectl create namespace devops-tools
```

**第 2 步**：创建 `serviceAccount.yaml` 文件并复制以下管理服务帐户 manifest 清单；

```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: devops-tools
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: devops-tools
```

`serviceAccount.yaml` 创建一个 `jenkins-admin` 的集群角色，`ClusterRole`，一个 `jenkins-admin` 服务账号，`ServiceAccount`，并将集群角色绑定到服务账号。

集群角色 `jenkins-admin` 拥有管理集群组件的所有权限。咱们还可以通过指定个别资源的活动来限制访问。

现在使用 `kubectl` 创建服务账户。

```bash
kubectl apply -f serviceAccount.yaml
```

**第 3 步**：创建 `volume.yaml` 并复制以下持久性卷清单 manifest；

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv-volume
  labels:
    type: local
spec:
  storageClassName: local-storage
  claimRef:
    name: jenkins-pv-claim
    namespace: devops-tools
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  local:
    path: /mnt
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker-node01
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: devops-tools
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

**重要提示**：将 `worker-node01` 替换为咱们集群工作节点的任何一个主机名。

你可以使用 `kubectl` 获得工作节点的主机名。

```bash
kubectl get nodes
```

对于卷，出于演示目的，我们使用 `local` 存储类。意思是，他会在特定节点中的 `/mnt` 下创建一个 `PersistentVolume` 卷。

由于 `local` 存储类需要节点选择器，咱们需要正确指定工作节点名称，以便 Jenkins pod 在特定的节点上得到调度。

如果 pod 被删除（？）或重启，数据将被持久保存在节点卷中。然而，如果节点被删除，你将失去所有的数据。

理想情况下，咱们应该使用云提供商提供的可用存储类的持久卷，或者集群管理员提供的持久卷，以在节点出现故障时保留数据。

我们来用 `kubectl` 创建卷。

```bash
kubectl create -f volume.yaml
```

**第 4 步**：创建一个名为 `deployment.yaml` 的部署文件并复制以下部署清单 manifest。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: devops-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-server
  template:
    metadata:
      labels:
        app: jenkins-server
    spec:
      securityContext:
            fsGroup: 1000
            runAsUser: 1000
      serviceAccountName: jenkins-admin
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          resources:
            limits:
              memory: "2Gi"
              cpu: "1000m"
            requests:
              memory: "500Mi"
              cpu: "500m"
          ports:
            - name: httpport
              containerPort: 8080
            - name: jnlpport
              containerPort: 50000
          livenessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 90
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
          readinessProbe:
            httpGet:
              path: "/login"
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          volumeMounts:
            - name: jenkins-data
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-data
          persistentVolumeClaim:
              claimName: jenkins-pv-claim
```

在此 Jenkins Kubernetes 部署中，我们使用了以下内容：

1. 为 Jenkins pod 能够写入本地持久卷的 `securityContext`；

2. 用于监测 Jenkins pod 的健康状况的活跃度及准备度探针，liveness and readiness probe；

3. 基于本地存储类的本地持久卷，持有 Jenkins 数据路径 `/var/jenkins_home`。


> 该部署文件使用本地存储类持久性卷存储 Jenkins 数据。对于生产用例，咱们应该为咱们的 Jenkins 数据添加一个云特定的存储类持久性卷。


如果咱们不想要本地存储的持久卷，咱们可以用主机目录替换部署中的卷定义，如下所示。

```yaml
volumes:
- name: jenkins-data
emptyDir: \{}
```

使用 kubectl 创建部署。

```bash
kubectl apply -f deployment.yaml
```

检查部署状态。

```bash
kubectl get deployments -n devops-tools
```

现在，咱们可以使用以下命令获取部署详细信息。

```bash
kubectl describe deployments --namespace=devops-tools
```

### 使用 Kubernetes 服务访问 Jenkins

我们现在已经创建了一个部署。然而，他不能被外部世界访问。为了从外部世界访问 Jenkins 部署，我们需要创建一个服务并将其映射到部署上。

创建 `service.yaml` 并复制以下服务清单 manifest：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: devops-tools
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/path:   /
      prometheus.io/port:   '8080'
spec:
  selector:
    app: jenkins-server
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 32000
```

> 在这里，我们使用的类型是 `NodePort`，他将在所有 kubernetes 节点 IP 上的端口 `32000` 暴露 Jenkins。如果咱们有一个入站设置，咱们就可以创建一个入站规则来访问 Jenkins。此外，如果咱们在 AWS、Google 或 Azure 云上运行集群，咱们可以将 Jenkins 服务作为一个负载均衡器。

使用 `kubectl` 创建 Jenkins 服务。

```bash
kubectl apply -f service.yaml
```

现在，当浏览到任何一个节点 IP 上的 `32000` 时，咱们将能够访问 Jenkins dashboard。

```url
http://<node-ip>:32000
```

当咱们第一次访问控制面板时，Jenkins 会要求咱们提供初始管理密码。

你可以从 Kubernetes dashboard 或 CLI 的 pod 日志中获得初始管理员密码。咱们可以使用下面的 CLI 命令来获得 pod 的详细信息。

```bash
kubectl get pods --namespace=devops-tools
```

通过 pod 名称，咱们可以像下面所给出的那样得到日志。用咱们的 pod 名称替换那个 pod 名称。

```bash
kubectl logs jenkins-deployment-2539456353-j00w5 --namespace=devops-tools
```

可以在日志的末尾找到那个初始管理密码。

或者，你可以运行 `exec` 命令，直接从该位置获取密码，如下所示。

```bash
kubectl exec -it jenkins-559d8cd85c-cfcgk cat /var/jenkins_home/secrets/initialAdminPassword -n devops-tools
```

输入密码后，继续安装建议的插件并创建管理员用户。所有这些步骤在 Jenkins dashboard 中都是不言自明的。


## 使用 Helm v3 安装 Jenkins

典型的 Jenkins 部署由一个控制器节点和一个或多个代理（可选）组成。为了简化 Jenkins 的部署，我们将使用 [Helm](https://helm.sh/) 来部署 Jenkins。 Helm 是 Kubernetes 的一个包管理器，其包格式称为图，a chart。 [GitHub](https://github.com/helm/charts) 上提供了许多社区开发的图。


Helm Charts 提供了 "按钮式 "的应用部署和删除，使那些没有容器或微服务经验的人更容易采用和开发 Kubernetes 应用。

### 先决条件

#### Helm 命令行界面

如果你没有在本地安装和配置 Helm 命令行界面，请参阅下面的 [安装 Helm](#安装-Helm]) 和 [配置 Helm](#配置-Helm]) 部分。


### 安装 Helm

要安装 Helm CLI，请按照 [安装 Helm](https://helm.sh/docs/intro/install/) 页面的说明进行。


### 配置 Helm


安装并正确设置 Helm 后，按如下方式添加 Jenkins 源 repo：

```bash
$ helm repo add jenkinsci https://charts.jenkins.io
$ helm repo update
```

可以使用以下命令列出 Jenkins repo 源中的 helm 图表：

```bash
$ helm search repo jenkinsci
```

### 创建一个持久卷

我们打算为我们的 Jenkins 控制器 pod 创建一个持久卷。这将防止我们在重启 minikube 时丢失 Jenkins 控制器的整个配置和咱们的作业。[这个官方 minikube 文档](https://minikube.sigs.k8s.io/docs/handbook/persistent_volumes/) 解释了我们可以使用哪些目录来挂载咱们的数据。在多节点 Kubernetes 集群中，咱们需要一些类似 NFS 的解决方案，来使挂载目录在整个集群中可用。但是因为我们使用的是单节点集群 minikube，所以我们不必为此操心。

我们选择使用 `/data` 目录。这个目录将包含我们的 Jenkins 控制器配置。


**我们将创建一个名为 `jenkins-pv` 的卷**：

- 将 `https://raw.githubusercontent.com/installing-jenkins-on-kubernetes/jenkins-volume.yaml` 中的内容粘贴到名为 `jenkins-volume.yaml` 的 YAML 格式文件中；

- 运行以下命令来应用该规范：


```bash
$ kubectl apply -f jenkins-volume.yaml
```

> 值得注意的是，在上述规范中，`hostPath` 使用咱们节点的 `/data/jenkins-volume/` 来模拟网络连接的存储。这种方法只适合于开发和测试目的。对于生产来说，咱们应该提供一个网络资源，比如 Google Compute Engine 的持久化磁盘，或者 Amazon Elastic Block Store 卷。

> 为 `hostPath` 配置的 minikube 会只将 `/data` 的权限设置为 `root` 帐户。创建卷后，咱们需要手动更改权限以允许 `jenkins` 帐户写入其数据。


```bash
minikube ssh
sudo chown -R 1000:1000 /data/jenkins-volume
```

### 创建一个服务账号

在 Kubernetes 中，服务帐户用于为 pod 提供身份。想要与 API 服务器交互的 pod 将使用特定的服务帐户进行身份验证。默认情况下，应用程序将以他们所运行的命名空间中的默认服务账户进行认证。这意味着，例如，运行在 `test` 命名空间的应用程序将使用 `test` 命名空间的默认服务账户。

我们将创建一个名为 `jenkins` 的服务帐户：

`ClusterRole` 是一组权限，可以分配给给定集群内的资源。Kubernetes 的 API 根据其相关的 API 对象被归类为 API 组。在创建 `ClusterRole` 时，咱们可以指定 `ClusterRole` 可以对一个或多个 API 组中的一个或多个 API 对象进行的操作，就像我们上面做的那样。`ClusterRole` 有几种用途。咱们可以使用 `ClusterRole` 来：

- 定义对命名空间资源的权限并在单个命名空间内授予权限；
- 定义命名空间资源的权限，并在所有命名空间中授予权限；
- 定义集群范围内资源的权限。

如果咱们打算在整个集群内定义一个角色，请使用 `ClusterRole`；如果咱们打算在一个命名空间内定义一个角色，请使用 `Role`。

角色绑定将一个角色中定义的权限授予一个用户或一组用户。他持有一个主体（用户、组或服务账户）的列表，以及一个对被授予的角色的引用。

`RoleBinding` 可以引用同一命名空间中的任何角色。或者，某个 `RoleBinding` 可以引用一个 `ClusterRole`，并将该 `ClusterRole` 绑定到 `RoleBinding` 的命名空间。为了将一个 `ClusterRole` 绑定到我们集群中的所有命名空间，我们使用 `ClusterRoleBinding`。

1. 将 [`https://raw.githubusercontent.com/installing-jenkins-on-kubernetes/jenkins-sa.yaml`](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-sa.yaml) 中的内容粘贴到名为 `jenkins-sa.yaml` 的 YAML 格式文件中；

2. 运行以下命令来应用该规范：

```bash
$ kubectl apply -f jenkins-sa.yaml
```

### 安装 Jenkins

我们将部署 Jenkins，包括 Jenkins Kubernetes 插件。更多细节见 [官方 chart](https://github.com/jenkinsci/helm-charts/tree/main/charts/jenkins)。

1. 为了启用持久性，我们将创建一个覆盖文件，an override file，并将其作为一个参数传递给 Helm CLI。请将 [raw.githubusercontent.com/jenkinsci/helm-charts/main/charts/jenkins/values.yaml](https://raw.githubusercontent.com/jenkinsci/helm-charts/main/charts/jenkins/values.yaml) 中的内容粘贴到一个名为 `jenkins-values.yaml` 的 YAML 格式的文件中；

2. 在你偏好的文本编辑器中打开 `jenkins-values.yaml` 文件，并修改以下内容：

- `nodePort`: 因为我们正在使用 minikube，我们需要使用 `NodePort` 作为服务类型。只有云供应商才会提供负载均衡器。我们将端口 `32000` 定义为端口；

- `storageClass`：

```yaml
storageClass: jenkins-pv
```

- `serviceAccount`：`jenkins-values.yaml` 文件的 `serviceAccount` 部分应该是这样的：

```yaml
serviceAccount:
  create: false
# Service account name is autogenerated by default
name: jenkins
annotations: {}
```

其中 `name: jenkins` 是指为 `jenkins` 创建的 `serviceAccount`。

- 我们还可以定义我们打算在我们的 Jenkins 上安装哪些插件。我们使用一些默认的插件，如 git 和管道 pipeline 插件。

3. 现在咱们可以通过运行 `helm install` 命令并传递给他以下参数来安装 Jenkins：

- 发布的名称 `jenkins`；

- 覆盖 `jenkins-values.yaml` 的带有 YAML 文件的 `-f` 命令行开关；

- Helm chart 名称 `jenkinsci/jenkins`；

- 带有咱们命名空间名称 `jenkins` 的 `-n` 命令行开关。

```bash
$ chart=jenkinsci/jenkins
$ helm install jenkins -n jenkins -f jenkins-values.yaml $chart
```

这会输出类似于以下的内容：


```bash
NAME: jenkins
LAST DEPLOYED: Wed Sep 16 11:13:10 2020
NAMESPACE: jenkins
STATUS: deployed
REVISION: 1
```

### 安装后配置

1. 通过运行以下命令，获得咱们的 `admin` 用户密码：

```bash
$ jsonpath="{.data.jenkins-admin-password}"
$ secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
$ echo $(echo $secret | base64 --decode)
```

2. 通过在同一个 shell 中运行下面这些命令，获得要访问的 Jenkins URL：

```bash
$ jsonpath="{.spec.ports[0].nodePort}"
$ NODE_PORT=$(kubectl get -n jenkins -o jsonpath=$jsonpath services jenkins)
$ jsonpath="{.items[0].status.addresses[0].address}"
$ NODE_IP=$(kubectl get nodes -n jenkins -o jsonpath=$jsonpath)
$ echo http://$NODE_IP:$NODE_PORT/login
```

3. 用第 1 步中的密码和用户名：`admin` 登录；

4. 通过在 `values.yaml` 文件中指定 `configScripts` 来将 Jenkins 配置, Configuration，用作代码 Code。请参阅 [配置作为代码的文档](https://plugins.jenkins.io/configuration-as-code) 和 [示例](https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos)；

请访问 [Jenkins on Kubernetes解决方案](https://cloud.google.com/solutions/jenkins-on-container-engine) 页面，了解在 Kubernetes 上运行 Jenkins 的更多信息。访问 [作为为代码项目的 Jenkins 配置](https://www.jenkins.io/projects/jcasc/)，了解更多关于作为代码的配置方面的信息。根据咱们的环境，Jenkins 的启动可能需要一点时间。输入以下命令来检查咱们 Pod 的状态：

```bash
$ kubectl get pods -n jenkins
```

安装 Jenkins 后，状态应设置为 `Running`，如以下输出所示：

```bash
$ kubectl get pods -n jenkins
NAME                       READY   STATUS    RESTARTS   AGE
jenkins-645fbf58d6-6xfvj   1/1     Running   0          2m
```

### 访问 Kubernetes 中的 Jenkins


1. 要访问咱们的 Jenkins 服务器，咱们必须找回密码。咱们可以使用以下两个选项之一找回密码。

**选项 1**

请运行以下命令：

```bash
$ jsonpath="{.data.jenkins-admin-password}"
$ secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
$ echo $(echo $secret | base64 --decode)
```

输出应如下所示：

```bash
Um1kJLOWQY
```

> 注意：咱们的密码将有所不同。


**选项 2**

请运行以下命令：


```bash
$ jsonpath="{.data.jenkins-admin-password}"
$ kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath
```

输出应该是一个 **base64 编码的字符串**，就像这样：


```bash
WkIwRkdnbDZYZg==
```

解码这个 base64 字符串，咱们就会得到咱们的密码。咱们可以使用 [这个网站](https://www.base64decode.org/) 来解码咱们的输出。


2. 使用以下命令获得正在运行 Jenkins 的 Pod 名称：


```bash
$ kubectl get pods -n jenkins
```


3. 使用 `kubectl` 命令设置端口转发：


```bash
$ kubectl -n jenkins port-forward <pod_name> 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

访问 `127.0.0.1:8080/`，用 `admin` 作为用户名和咱们之前获取的密码登录。


## 使用 YAML 文件安装 Jenkins

本节介绍了如何使用一组 YAML（Yet Another Markup Language）文件在 Kubernetes 集群上安装 Jenkins。YAML 文件很容易被追踪、编辑，并且可以无限次重复使用。


### 创建 Jenkins 部署文件


把 [这里](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-deployment.yaml) 的内容复制到你喜欢的文本编辑器中，并在我们在上面 [这一节](#kubernetes-方式-jenkins-的部署) 创建的 `jenkins` 命名空间中创建一个 `jenkins-deployment.yaml` 文件。

- 这个 [部署文件](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-deployment.yaml) 正在定义一个部署，Deployment，正如 `kind` 字段所表明的那样；

- 部署，Deployment，指定了一个单一的副本。这确保在发生故障时，复制控制器，the Replication Controller，将维护一个且只有一个实例；

- 容器镜像名称为 `jenkins`，版本为 `2.32.2`；

+ 规范，the spec，中指定的端口列表是一个从 Pods IP 地址上的容器暴露出来的端口列表；

    - Jenkins 运行在（`http`）`8080` 端口；
    - Pod 暴露了 `jenkins` 容器的 `8080` 端口。

- 该文件的 `volumeMounts` 部分创建了一个持久卷。这个卷被挂载在容器中的 `/var/jenkins_home` 路径下，因此对 `/var/jenkins_home` 中的数据的修改会被写入这个卷中。持久卷的作用是存储 Jenkins 的基本数据，并在 pod 的生命周期内保存这些数据。

咱们把内容添加到Jenkins部署文件中后，要退出并保存更改。


### 部署 Jenkins


要创建部署，请执行：

```bash
$ kubectl create -f jenkins-deployment.yaml -n jenkins
```

该命令还指示系统在 `jenkins` 命名空间内安装Jenkins。


要验证创建部署是否成功，咱们可以运行：

```bash
$ kubectl get deployments -n jenkins
```

### 授予对 Jenkins 服务的访问权限


咱们已经部署了一个 Jenkins 实例，但他仍然无法访问。Jenkins Pod 已分配到一个属于 Kubernetes 集群内部的地址。可以登录 Kubernetes 节点，并从那里访问 Jenkins，但这并不是一个非常有用的访问服务的方式。

为了使 Jenkins 在 Kubernetes 集群之外也能被访问，Pod 需要作为一项服务被公开。服务是一个抽象的概念，他将 Jenkins 暴露在更广泛的网络中。他允许我们保持与 Pod 的持久连接，而不管集群中的变化如何。在本地部署中，这意味着创建一个 `NodePort` 服务类型。`NodePort` 服务类型在集群中的每个节点的一个端口上公开一项服务。该服务通过节点的 IP 地址和服务 `nodePort` 被访问。[这里](https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-service.yaml) 定义了一个简单的服务：

- 这个服务文件正在定义一项服务，Service，正如 `kind` 字段所表明的那样；

- 该服务的类型是 `NodePort`。其他选项是 `ClusterIP`（只能在集群内访问）和 `LoadBalancer`（由云提供商分配的 IP 地址，例如 AWS Elastic IP）；

+ 规范，the spec，中指定的端口列表是此服务公开的端口列表；

    - 端口，port, 是将由服务暴露的端口；

    - 目标端口，target port, 是访问该服务所针对的 Pod 的端口。也可以指定一个端口名称。

- 选择器，selector 指定了该服务所针对的 Pod 的选择标准。


要创建服务，请执行：

```bash
$ kubectl create -f jenkins-service.yaml -n jenkins
```

为了验证创建服务是否成功，咱们可以运行：


```bash
$ kubectl get services -n jenkins
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP    PORT(S)           AGE
jenkins    NodePort    10.103.31.217    <none>         8080:32664/TCP    59s
```


### 访问 Jenkins 控制面板


所以现在我们已经创建了一个部署和服务，我们如何访问 Jenkins 呢？

从上面的输出中，我们可以看到该服务已经在 `32664` 端口暴露。我们还知道，由于该服务是 `NodeType` 类型，该服务将把对该端口上任何节点的请求路由到 Jenkins pod。我们剩下的就是要确定 `minikube` 虚拟机的 IP 地址。Minikube 通过包括一个特定的命令，输出运行集群的 IP 地址，使这一点变得非常简单：

```bash
$ minikube ip
192.168.99.100
```

现在我们可以在 [`192.168.99.100:32664/`](http://192.168.99.100:32664/) 访问 Jenkins 实例。

要访问 Jenkins，咱们首先需要输入咱们的凭证。新安装的默认用户名是 `admin`。密码可以通过几种方式获得。这个例子使用 Jenkins 部署的 pod 名称。

要找到 pod 的名称，请输入以下命令：

```bash
$ kubectl get pods -n jenkins
```

一旦咱们找到了 pod 的名字，就用它来访问 pod 的日志。


```bash
$ kubectl logs <pod_name> -n jenkins
```

密码在日志的最后，格式为一个长的字母数字字符串：

```text
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required.
An admin user has been created and a password generated.
Please use the following password to proceed to installation:

94b73ef6578c4b4692a157f768b2cfef

This may also be found at:
/var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

咱们已经成功地在咱们的 Kubernetes 集群上安装了 Jenkins，可以用它来创建新的、高效的开发管道，development pipelines。


## 使用 Jenkins Operator 安装 Jenkins

Jenkins Operator 是一个 Kubernetes 的原生 Operator，他管理 Jenkins 在 Kubernetes 上的操作。

他在构建时考虑到了不变性和作为代码的声明性配置，以使在 Kubernetes 上部署和运行 Jenkins 所需的许多手动任务自动化。

Jenkins Operator 很容易安装，只需应用几个 `yaml` 清单或使用 Helm。

关于在你的 Kubernetes 集群上安装 Jenkins Operator 以及在那里部署和配置 Jenkins 的说明，请参见 [Jenkins Operator 的官方文档](https://jenkinsci.github.io/kubernetes-operator/docs/getting-started/latest/)。


{{#include ./docker.md:292:}}


## 总结


当咱们在 Kubernetes 上为生产性工作负载托管 Jenkins 时，咱们需要考虑设置一个高可用的持久化卷，以避免在删除 pod 或节点时的数据丢失。

在 Kubernetes 环境中，pod 或节点的删除可能随时发生。他可能是一个打补丁的活动，也可能是一个缩减的活动，a downscaling activity。

希望这个分步指南能帮助咱们学习和理解在 Kubernetes 集群上设置 Jenkins 服务器所涉及的组件。


（End）


