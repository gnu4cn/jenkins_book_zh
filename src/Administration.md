# 系统管理


## Java 要求

Jenkins 安装分别有单独的运行和作业执行两方面的要求。

### 运行 Jenkins 系统

从 Jenkins 2.357 和 LTS 2.361.1 开始，Jenkins 需要 Java 11 或 17。请 [在公告博文中阅读更多相关内容](https://www.jenkins.io/blog/2022/06/28/require-java-11/)。

> *将 Java 版本升级到 11*
>
> 将现有的 Jenkins 设置从 Java 8 升级到 Java 11？请参考 [升级指南](#upgrade-java-guidelines)。


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

### 把 Jenkins 的 Java 版本从 8 升级到 11

有一些细节和步骤来升级用于运行 Jenkins 的 JVM，更具体地说是从 Java 8 到 Java 11。

如果你要升级用于运行 Jenkins 的 JVM，特别是如果你要从 Java 8 升级到 Java 11，有一些细节你应该知道，也应该采取预防措施。


### 备份

与任何升级一样，我们建议备份 `JENKINS_HOME`，并在生产实例上，执行升级之前，用备份测试升级。


### 升级 Jenkins

如果你需要升级 Jenkins 以及 JVM，我们建议你：

1. 备份 `JENKINS_HOME`；
2. 停止 Jenkins 实例；
3. 升级运行 Jenkins 的 JVM；
    - 使用软件包管理器来安装新的 JVM；
    - 确保默认 JVM 是新安装的版本。如果不是，运行 `systemctl edit jenkins` 并设置 `JAVA_HOME` 环境变量或 `JENKINS_JAVA_CMD` 环境变量。
4. 将 Jenkins 升级到最新版本；
    - 如何升级 Jenkins 取决于咱们原来的 Jenkins 安装方法；
    - 我们（Jenkins 开发团队）建议你使用你系统的软件包管理器（如 `apt` 或 `yum`）。
5. 验证升级，以确认所有的插件和作业都已加载；
6. 升级所需的插件（见 [升级插件](#升级插件)）。

从 Jenkins 2.357 每周发布版和长期支持版 2.361.1 开始，Java 11 或 Java 17 是必需的。


### 升级插件

不仅要升级 Jenkins 和 JVM，而且要升级支持 Java 11 的插件，这一点很重要。插件的升级保证了与最新的 Jenkins 版本的兼容性。

> 如果你发现一个以前没有报告的问题，请让我们知道：请阅读 [how to report an issue](https://www.jenkins.io/participate/report-issue/#issue-reporting) 以获得指导。


### Jakarta XML 绑定

有些插件使用 JDK 提供的 JAXB 库。然而，`java.xml.bind` 和 `javax.activation` 模块在 OpenJDK 11 中不再包含，如果没有提供替换，插件可能会失效。

为了解决这个问题，我们把这些库捆绑在一个新的分离式插件中：[JAXB 插件](https://plugins.jenkins.io/jaxb)。当任何比 `2.163` 更新的 Jenkins 核心在 Java 11 上运行时，这个插件会自动安装。然而，如果你在 Jenkins 之外管理你的插件，例如，如果你在 Docker 镜像中使用 `plugins.txt`，你可能需要明确安装该插件。


### 代理上的 JVM 版本

由于控制器和代理的通信方式，所有代理必须运行在与控制器相同的 JVM 版本上。如果你要升级你的 Jenkins 控制器以运行在 Java 11 上，你也需要升级你的代理上的 JVM。

你可以用 [Versions Node Monitors](https://plugins.jenkins.io/versioncolumn) 插件来验证每个代理的版本。这个插件在你的 Jenkins 实例的节点管理屏幕上，提供关于每个代理的 JVM 版本信息。你也可以配置这个插件来自动断开任何 JVM 版本不正确的代理。


### Java Web Start

Java Web Start 已从 Java 11 中移除。当 Jenkins 控制器在 Java 11 上运行时，Java Web Start 按钮将不再出现在 Web UI 中。你不能从下载到 Web 浏览器的 `*.jnlp` 文件，为 Java 11 Jenkins 服务器启动代理。

没有替换此功能的计划。请使用 [SSH Build Agents](https://plugins.jenkins.io/ssh-slaves) 等插件，使用操作系统命令行调用 `java -jar agent.jar`，或使用容器将代理连接到 Java 11 上的 Jenkins。


### JDK 工具自动安装程序

Oracle JDK 11 的授权使 Jenkins 社区无法列出 Oracle JDK。由于这一许可限制，Oracle JDK 11 不能被 Jenkins 自动安装。这个问题在 [issue JENKINS-54305](https://issues.jenkins.io/browse/JENKINS-54305) 中进行了追踪。

作为替代方案，我们鼓励你使用其中包含了构建所需的所有工具的基于镜像的容器。



## 浏览器兼容性

> 本页记录了 Jenkins 自动化服务器中的浏览器支持政策。他不适用于 Jenkins 网站或 Jenkins 项目托管的其他服务。


### 支持模型

Jenkins 的 web 浏览器支持分为三类：

1. 级别 1：旨在主动修复这些浏览器，并在所有浏览器上提供平等的用户体验;
2. 级别 2：接受补丁来解决问题，并尽最大努力确保至少有一种方法可以执行任何操作；
3. 级别 3：没有保证。我们将接受补丁，但前提是他们的风险较低。**对于下面未列出的浏览器/版本，这是默认浏览器支持的默认级别**。

我们（Jenkins 开发团队）不声明与浏览器的预发布（例如，alpha、beta 或 canary）版本的任何兼容性或接受错误报告和补丁。

### 浏览器兼容性汇总表

| 浏览器 | 级别 1 | 级别 2 | 级别 3 |
| :-- | :-- | :-- | :-- |
| Google Chrome | 最新的定期发布/补丁 | N-1 版本，最新补丁 | 别的版本 |
| Mozilla Firefox | 最新的定期发布/补丁；最新的 [ESR](https://www.mozilla.org/en-US/firefox/organizations/) 发布版本 | N-1 版本，最新补丁 | 别的版本 |
| Microsoft Edge | 最新的定期发布/补丁 | N-1 版本，最新补丁 | 别的版本 |
| Apple Safari | 最新的定期发布/补丁 | N-1 版本，最新补丁 | 别的版本 |

对移动浏览器（例如 iOS Safari）的支持尚未确定。

### 更改历史

- 2022-02-01 - 移除对 Internet Explorer 的支持，增加 Edge（ [开发者邮件列表中的讨论](https://groups.google.com/g/jenkinsci-dev/c/piANoeohdik) ）；
- 2019-11-19 - 政策更新（[开发者邮件列表中的讨论](https://groups.google.com/forum/#!topic/jenkinsci-dev/TV_pLEah9B4) ）；
- 2014-09-03 - Jenkins 1.579 的最初政策（ [治理会议记录](http://meetings.jenkins-ci.org/jenkins/2014/jenkins.2014-09-03-18.01.html) ）。
