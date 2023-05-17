# Kubernetes 安装

Kubernetes（K8s）是一个容器化应用部署、扩展和管理自动化的开源系统。

Kubernetes 集群为 Jenkins 增加了一个新的自动化层。Kubernetes 确保资源得到有效利用，确保咱们的服务器和底层基础设施不会过载。Kubernetes 编排容器部署的能力确保 Jenkins 总是有适量的资源可用。

在 Kubernetes 集群上托管 Jenkins 有利于基于 Kubernetes 的部署和基于动态容器的可扩展 Jenkins 代理。在这里，我们看到在 Kubernetes 集群上设置 Jenkins 的分步过程。


## 在 Kubernetes 上设置 Jenkins

要在 Kubernetes 上设置 Jenkins 集群，我们将执行以下操作：

1. [创建一个命名空间](#创建一个命名空间)；

2. [创建一个服务账号](#创建一个服务账号)，具有 Kubernetes 管理权限；

3. 为 Pod 重启时的持久性 Jenkins 数据 [创建本地持久性卷](#创建本地持久性卷)；

4. [创建一个部署 YAML](#创建一个部署 YAML) 并进行部署；

5. [创建一个服务 YAML](#创建一个服务 YAML) 并进行部署；


> 本指南不使用本地持久卷，因为这是一个通用指南。要为 Jenkins 数据使用持久卷，咱们需要创建相关云或本地数据中心的卷并进行配置。

### Jenkins Kubernetes Manifest 文件

这里使用的所有Jenkins Kubernetes Manifest 清单文件都托管在 GitHub 上。如果咱们在复制文件中的 manifest 清单时遇到困难，请克隆该仓库。

```bash
git clone https://github.com/scriptcamp/kubernetes-jenkins
```

请使用这些 GitHub 文件作为参考，并按照接下来的步骤进行。



