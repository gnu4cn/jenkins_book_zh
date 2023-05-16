# 安装 Jenkins

本章中的过程适用于 Jenkins 的新安装。

Jenkins 通常在其自己的进程中作为独立应用程序运行。 Jenkins WAR 文件捆绑了 [Winstone](https://github.com/jenkinsci/winstone)，一个 [Jetty](https://www.eclipse.org/jetty/) servlet 容器包装器，并且可以在任何操作系统或平台上启动，只要有 Jenkins 支持的 Java 版本。

理论上，Jenkins 也可以在 [Apache Tomcat](https://tomcat.apache.org/) 或 [WildFly](https://www.wildfly.org/) 这样的传统 servlet 容器中作为 servlet 运行，但在实践中，这基本上没有经过测试，而且有很多注意事项。特别是，对 WebSocket 代理的支持只在 Jetty servlet 容器中实现。详见 [Servlet 容器支持政策](https://www.jenkins.io/doc/administration/requirements/servlet-containers) 页面。

## Docker 安装

[Docker](https://docs.docker.com/get-started/overview/) 是一个在称为 "容器"（或 Docker 容器）的隔离环境中运行应用程序的平台。像Jenkins 这样的应用程序可以作为只读 "镜像"（或 Docker 镜像）被下载，每个镜像都作为一个容器在 Docker 中运行。一个 Docker 容器实际上是一个 Docker 镜像的 "运行实例"。从这个角度来看，镜像或多或少是永久存储的（就镜像更新发布而言），而容器则是临时存储的。请在 Docker 文档的 [Get Started, Part 1: Orientation and Setup](https://docs.docker.com/get-started/) 中阅读更多关于这些概念。

Docker的基本平台和容器设计意味着单个的Docker镜像（对于任何特定的应用程序，如 Jenkins）可以在任何支持的操作系统（macOS、Linux 和 Windows）或同样运行着 Docker 的云服务（AWS 和 Azure）。

### 安装 Docker
