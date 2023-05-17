## Java 要求

Jenkins 安装分别有单独的运行和作业执行两方面的要求。

### 运行 Jenkins 系统

从 Jenkins 2.357 和 LTS 2.361.1 开始，Jenkins 需要 Java 11 或 17。请 [在公告博文中阅读更多相关内容](https://www.jenkins.io/blog/2022/06/28/require-java-11/)。

> *将 Java 版本升级到 11*
>
> 将现有的 Jenkins 设置从 Java 8 升级到 Java 11？请参考 [升级指南](#把-jenkins-的-java-版本从-8-升级到-11)。


### 在 Docker 中的 Java 11 上运行 Jenkins

Java 11 Docker 安装说明包含在 [“下载和运行 Docker 中的 Jenkins”](Ch01_Installing_Jenkins.md#下载和运行-docker-中的-jenkins) 中。或者，`jenkins/jenkins:jdk17` Docker 镜像允许咱们在 Java 17 上运行 Jenkins 控制器。

所有其他的Java版本都不被支持。

Jenkins项目使用以下JDK/JREs执行完整的测试流程：

- OpenJDK JDK/JRE 11 - 64 位；
- OpenJDK JDK/JRE 17 - 64 位。

其他厂商的 JRE/JDK 是被支持的，可以使用。关于已知的 Java 兼容性问题，请参考 [我们的 issue tracker](https://issues.jenkins.io/issues/?jql=labels%3Djdk)。Jenkins 维护者们积极测试 [基于 HotSpot 的 Java 虚拟机](https://en.wikipedia.org/wiki/HotSpot_(virtual_machine)，如来自 OpenJDK、Eclipse Temurin 和 Amazon Corretto 等。Jenkins 维护者不测试基于 Eclipse OpenJ9 的 Java 虚拟机。[Platform SIG](https://www.jenkins.io/sigs/platform/) 在 2021 年 8 月停止了对基于 OpenJ9 的 Java 虚拟机的工作。

### 在 Jenkins 上运行基于 Java 的工具和构建

用于构建基于 Java 项目的 JDK 版本，或用于运行基于 Java 工具的 Java 版本独立于用于运行 Jenkins 控制器和代理进程的 Java 版本。在构建过程中，可以启动任何版本的 JRE 或 JDK，只要他与运行构建的系统兼容即可。这包括：

- 在 shell 构建步骤中执行 `java` 或 `javac`，以及类似的情况;
- 使用由 JDK 工具安装程序管理的 JDK 执行 Maven/Ant/... 等构建步骤。

少数插件有更严格的要求，一般要求构建的 Java 版本与运行 Jenkins 控制器和代理所用的版本相同。一个明显的例子是 Maven 集成插件，他要求用于 Maven 构建的 JDK 版本必须与用于 Jenkins 控制器的版本至少相同。这些情况一般在插件的文档中都有记载。

### 监控 Java 版本

现代版本的 Jenkins 控制器和 Jenkins 代理会验证 Java 要求，并在启动时通知用户不支持的版本。

[Versions Node Monitors 插件](https://plugins.jenkins.io/versioncolumn) 提供了详细的 Java 版本监控。


### 所使用的 Java 开发套件 JDKs

Jenkins 项目使用 [Eclipse Temurin](https://projects.eclipse.org/projects/adoptium.temurin) 作为其主要的 JDK 来构建和测试基于 Java 的应用程序。这包括：

- 容器镜像；
- Jenkins 核心发布版本的构建；
- [自动化的插件版本发布](https://www.jenkins.io/doc/developer/publishing/releasing-cd/)；
- [持续集成构建和测试](https://ci.jenkins.io/)；
- 测试基础设施。


选择 Temurin 的一些理由是：

- 在多种平台上的可用性，包括不同的操作系统和体系结构，以及许多不同的 Java SE 版本；
- 由 Eclipse 基金会提供定期维护和长期支持。
