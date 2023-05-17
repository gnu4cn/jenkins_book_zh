## Docker 安装

[Docker](https://docs.docker.com/get-started/overview/) 是一个在称为 "容器"（或 Docker 容器）的隔离环境中运行应用程序的平台。像Jenkins 这样的应用程序可以作为只读 "镜像"（或 Docker 镜像）被下载，每个镜像都作为一个容器在 Docker 中运行。一个 Docker 容器实际上是一个 Docker 镜像的 "运行实例"。从这个角度来看，镜像或多或少是永久存储的（就镜像更新发布而言），而容器则是临时存储的。请在 Docker 文档的 [Get Started, Part 1: Orientation and Setup](https://docs.docker.com/get-started/) 中阅读更多关于这些概念。

Docker的基本平台和容器设计意味着单个的Docker镜像（对于任何特定的应用程序，如 Jenkins）可以在任何支持的操作系统（macOS、Linux 和 Windows）或同样运行着 Docker 的云服务（AWS 和 Azure）。

### 安装 Docker

要在你的操作系统上安装Docker，请遵循 [Guided Tour page](Guided_Tour.md#prerequisites) 的 "先决条件 "部分。

作为替代解决方案，你可以访问 [Dockerhub](https://hub.docker.com/search?type=edition&offering=community) 并选择适合咱们操作系统或云服务的 **Docker 社区版**。按照他们网站上的安装说明进行操作。

> 如果你在基于 Linux 的操作系统上安装 Docker，请确保你对 Docker 进行配置，使其可以作为非 root 用户进行管理。请在 Docker 文档的 [Linux 安装后步骤](https://docs.docker.com/engine/installation/linux/linux-postinstall/) 页面阅读更多相关信息。该页面还包含了关于如何配置 Docker 在系统引导时启动的信息。

### 前提条件

最低硬件要求：

- 256MB 运行内存 RAM；
- 1GB 的磁盘空间（但如果作为Docker容器运行Jenkins，建议最小为10GB）。

为小团队推荐的硬件配置：

- 4GB 以上的运行内存 RAM;
- 50GB 以上的磁盘空间。

软件要求：

- Java：请参阅 [Java 要求](Administration.md#java-要求) 页面；
- Web 浏览器：请参阅 [Web 浏览器兼容性](Administration.md#浏览器兼容性) 页面;
- 对于 Windows 操作系统：[Windows 支持政策](Administration.md#windows-support-policy);
- 对于 Linux 操作系统：[Linux 支持政策](Administration.md#linux-support-policy);
- 对于 Servlet 容器： [Servlet 容器支持政策](Administration.md#servlet-container-support-policy)。

### 下载和运行 Docker 中的 Jenkins

有好几个 Jenkins 的 Docker 镜像可以使用。

推荐使用的 Docker 镜像是官方的 [`jenkins/jenkins` 镜像](https://hub.docker.com/r/jenkins/jenkins/)（来自 [Docker Hub 资源库](https://hub.docker.com/)）。这个镜像包含 Jenkins 当前的长期支持（LTS）版本（可用于生产）。但是这个镜像里面没有 docker CLI，也没有捆绑常用的 Blue Ocean 插件和功能。这意味着，如果你想使用 Jenkins 和 Docker 的全部功能，你或许要通过下面描述的安装过程。

> 每次发布新的 Jenkins Docker 版本时，都会发布一个新的 `jenkins/jenkins` 镜像。你可以在 [标签页](https://hub.docker.com/r/jenkins/jenkins/tags/) 上看到以前发布的 `jenkins/jenkins` 镜像的版本列表。

### 在 macOS 和 Linux 上

1. 打开一个终端窗口;
2. 使用以下的 [`docker network create`](https://docs.docker.com/engine/reference/commandline/network_create/) 命令在 Docker 中创建一个 [桥接网络](https://docs.docker.com/network/bridge/)：

```bash
docker network create jenkins
```

3. 为了在 Jenkins 节点内执行 Docker 命令，使用以下的 `docker run` 命令下载并运行 [`docker:dind`](https://docs.docker.com/engine/reference/run/) Docker 镜像：

```bash
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```

    1. （可选的）`--name jenkins-docer` 命令行标志，指定运行镜像时要使用的 Docker 容器名称。默认情况下，Docker 将为该容器生成一个唯一的名称；
    2. （可选的）`--rm` 命令行开关，关闭时自动删除 Docker 容器（Docker 镜像的实例）；
    3. （可选的） `--detach` 命令行开关，在后台运行 Docker 容器。稍后可以通过运行 `docker stop jenkins-docker` 停止此实例；
    4. `--privileged` 命令行开关，在 Docker 中运行 Docker 目前需要特权访问才能正常运行。较新的 Linux 内核版本可能会放宽此要求；
    5. `--network jenkins` 命令行参数，这与前面步骤中创建的网络相对应；
    6. `--network alias docker` 命令行参数，使 Docker 容器中的 Docker 在 `jenkins` 网络中作为主机名 `docker` 可用；
    7. `--env DOCKER_TLS_CERTDIR=/certs` 命令行参数，在 Docker 服务器中启用 TLS。由于使用了特权容器，因此建议这样做，尽管他需要使用下面会描述的共享卷。此环境变量控制管理 Docker TLS 证书的根目录；
    8. `--volume jenkins-docker-certs:/certs/client` 命令行参数，将容器内的 `/certs/client` 目录映射到上面创建的名为 `jenkins-docker-certs` 的 Docker 卷；
    9. `--volume jenkins-data:/var/jenkins_home` 命令行参数，将容器内的 `/var/jenkins_home` 目录映射到名为 `jenkins-data` 的 Docker 卷。这将允许由该 Docker 容器的 Docker 守护进程控制的其他 Docker 容器从 Jenkins 挂载数据；
    10. （可选的）`--publish 2376:2376` 命令行参数，在主机上暴露 Docker 守护程序端口。这对于在主机上执行 docker 命令以控制此内部 Docker 守护进程很有用；
    11. `docker:dind` 命令行参数，`docker:dind` 镜像本身。这个镜像可以在运行前通过以下命令下载：`docker image pull docker:dind`；
    12. Docker 卷的存储驱动。参见 ["Docker存储驱动"](https://docs.docker.com/storage/storagedriver/select-storage-driver)，了解支持的选项。
