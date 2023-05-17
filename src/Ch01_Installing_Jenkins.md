# 安装 Jenkins

本章中的过程适用于 Jenkins 的新安装。

Jenkins 通常在其自己的进程中作为独立应用程序运行。 Jenkins WAR 文件捆绑了 [Winstone](https://github.com/jenkinsci/winstone)，一个 [Jetty](https://www.eclipse.org/jetty/) servlet 容器包装器，并且可以在任何操作系统或平台上启动，只要有 Jenkins 支持的 Java 版本。

理论上，Jenkins 也可以在 [Apache Tomcat](https://tomcat.apache.org/) 或 [WildFly](https://www.wildfly.org/) 这样的传统 servlet 容器中作为 servlet 运行，但在实践中，这基本上没有经过测试，而且有很多注意事项。特别是，对 WebSocket 代理的支持只在 Jetty servlet 容器中实现。详见 [Servlet 容器支持政策](https://www.jenkins.io/doc/administration/requirements/servlet-containers) 页面。

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


