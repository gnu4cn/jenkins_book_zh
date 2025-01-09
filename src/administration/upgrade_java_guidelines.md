## 把 Jenkins 的 Java 版本从 8 升级到 11

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


（End）


