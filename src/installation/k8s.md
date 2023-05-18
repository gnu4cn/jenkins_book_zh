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

当咱们第一次访问仪表板时，Jenkins 会要求咱们提供初始管理密码。

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



