# 系统管理


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


## Windows 支持政策

此页面记录了 Jenkins 服务器和代理的 Windows 支持政策。


### 范围

Jenkins 插件，例如 [WMI Windows 代理](https://plugins.jenkins.io/windows-slaves)，可能对控制器和/或代理上的 Windows 版本设置额外的要求。本页没有记录这些要求。请参考插件文档。


### 缘由

理论上，Jenkins 可以在任何可以运行所支持 Java 版本的地方运行，但在实践中存在一些限制。Jenkins core 和一些插件包含了原生代码，或依赖于 Windows API 和子系统，因此他们依赖于特定的 Windows 平台和版本。在 Windows 服务中，我们也使用 [Windows 服务封装器（WinSW）](https://github.com/winsw/winsw)，他需要 .NET 框架。


### 支持级别

我们为 Windows 平台定义了多个支持级别。


| 支持级别 | 描述 | 平台 |
| :-- | :-- | :-- |
| **级别 1** - 完全支持 | 我们对这些平台进行了自动化测试，我们打算及时修复报告的问题。 | <ul><li>64 位 (amd-64) Windows Server 版本，带有最新的 GA 更新包;</li><li>官方 Docker 镜像中使用的 Windows 版本。</li></ul> |
| **级别 2** - 受支持 | 我们不积极测试这些平台，但我们打算保持兼容性。我们很乐意接受补丁。 | <ul><li>Microsoft 普遍支持的 64 位 (amd-64) Windows Server 版本；</li><li>Microsoft 普遍支持的 64 位 (amd-64) Windows 10 版本。</li></ul> |
| **级别 3** - 补丁会被重视 | 支持可能有限制和额外要求。我们不测试兼容性，如果有需要，我们可能会放弃支持。如果补丁不会将 1/2 级支持置于风险之中并且不会产生维护开销，我们将考虑补丁。 | <ul><li>微软不再支持的 64 位 Windows 版本；</li><li>x86 和其他非 amd64 架构</li><li>非主流版本，如 Windows Embedded；</li><li>预览版；</li><li>Windows API 模拟引擎，例如 Wine 或 ReactOS。</li></ul> |
| **级别 4** - 不受支持 | 已知这些版本不兼容或有严重局限。我们不支持列出的平台，我们不会接受补丁。 | <ul><li>早于 SP3 的 Windows XP；</li><li>Windows Phone；</li><li>2008 年之前发布的其他 Windows 平台。</li></ul>


### `.NET` 的要求

- 从 `Jenkins 2.238` 开始，所有的 Windows 服务安装和内建的 Windows 服务管理逻辑都需要 .NET 框架 4.0 或以上；
- 在 `Jenkins 2.238` 前，支持 .NET Framework 2.0；
- 对于不支持这些版本的平台，请考虑使用 [Windows Service Wrapper](https://github.com/winsw/winsw) 项目提供的原生可执行文件。

### 参考

- [微软生命周期政策](https://docs.microsoft.com/en-us/lifecycle/)
- [微软产品生命周期检索](https://support.microsoft.com/en-us/lifecycle/search)

### 贡献

如果你想增加对更多 Windows 平台的支持或分享反馈，我们将感谢你的贡献！Jenkins 中的 Windows 支持是 [平台特别兴趣小组，Platform Special Interest Group](http://www.jenkins.io/sigs/platform/)，他有一个聊天室、一个邮件列表和定期会议。欢迎你加入这些渠道。


### 版本历史

- 2020 年 6 月 3 日 - 第一版（[邮件列表中的讨论](https://groups.google.com/forum/#!msg/jenkinsci-dev/oK8pBCzPPpo/1Ue1DI4TAQAJ)，[治理会议记录](https://docs.google.com/document/d/11Nr8QpqYgBiZjORplL_3Zkwys2qK1vEvK-NYyYa4rzg/edit#heading=h.ele42cjexh55)）


## Linux 支持政策

本页记录了 Jenkins 控制器和代理的 Linux 支持政策。

### 范围

个别 Jenkins 插件可能对控制器和/或代理上的 Linux 版本设置额外的要求。本页没有记录这些要求。请参考 [插件文档](https://plugins.jenkins.io/) 以了解额外的要求。


### 缘由

理论上，Jenkins 可以在任何可以运行所支持 Java 版本的地方运行，但在实践中存在一些限制。Jenkins core 和一些插件包括了原生代码，或依赖于 Linux API 和子系统，因此他们依赖于特定的 Linux 版本。Jenkins 平台特定的安装包依赖于特定的 Linux 版本。


### 支持级别


我们为 Linux 平台定义了多个支持级别。


| 支持级别 | 描述 | 平台 |
| :-- | :-- | :-- |
| **级别 1** - 受支持 | 我们为这些平台运行自动化的软件包管理器安装测试，我们打算及时修复报告的问题。我们推荐 Linux 下基于包管理器的安装或基于容器的安装。安装也可以使用 `jenkins.war` 而不使用包管理器，尽管我们的自动化测试集中在包管理器和容器安装上。 | <ul><li>[在 ci.jenkins.io 上测试过的](https://ci.jenkins.io/job/Packaging/job/packaging/job/master/) 使用 Debian 打包格式的 64位（amd64）Linux 版本；</li><li>[在 ci.jenkins.io 上测试过的](https://ci.jenkins.io/job/Packaging/job/packaging/job/master/) 使用 Red Hat rpm 打包格式的 64 位 (amd64) Linux 版本；</li><li>[在 ci.jenkins.io 上测试过的](https://ci.jenkins.io/job/Packaging/job/packaging/job/master/) 使用 OpenSUSE rpm 打包格式的 64 位 (amd64) Linux 版本；</li><li>[在 ci.jenkins.io 上测试过的](https://ci.jenkins.io/job/Infra/job/acceptance-tests/) 使用 Debian 打包格式的 64 位（arm64、s390x）Linux 版本；</li><li>[在 ci.jenkins.io 上测试过的](https://ci.jenkins.io/job/Infra/job/acceptance-tests/) 使用 rpm 打包格式的 64 位（arm64，s390x）Linux 版本；</li><li>为 [控制器](https://hub.docker.com/r/jenkins/jenkins) 和各种代理发布的 Linux 容器镜像（amd64、arm64、s390x）。</li></ul> |
| **级别 2** - 补丁会被重视 | 支持可能有限制和额外要求。我们不测试兼容性，而且我们可能在任何时候放弃支持。我们考虑那些不会使 1 级支持处于风险之中并且不会产生维护开销的补丁。 | <ul><li>32位（x86，arm）的 Linux 版本；</li><li>RISC-V 和其他不包含在 1 级支持中的架构；</li><li>预览版。</li></ul> |
| **级别 3** - 不受支持 | 已知这些版本不兼容或有严重局限。我们不支持列出的平台，我们不接受补丁。 | 操作系统供应商不再支持的 Linux 版本。 |

### 参考

- [Debian 的长期支持](https://wiki.debian.org/LTS)；
- [红帽企业级Linux的生命周期](https://access.redhat.com/support/policy/updates/errata)；
- [SUSE 产品支持生命周期](https://www.suse.com/lifecycle/)；
- [Ubuntu 生命周期和发布节奏](https://ubuntu.com/about/release-cycle)。

### 贡献

我们欢迎你提出增加对其他 Linux 平台支持的 PR，或者分享反馈；我们将感谢你的贡献！ Jenkins 中的 Linux 支持是 [平台特别兴趣小组](https://www.jenkins.io/sigs/platform/)，他有 [聊天室](https://app.gitter.im/#/room/#jenkinsci_platform-sig:gitter.im)、[论坛](https://community.jenkins.io/)，以及 [定期会议](https://www.jenkins.io/sigs/platform/#meetings)。欢迎你加入这些频道。

### 版本历史

- 2022 年三月 - 首个版本（[邮件列表中的讨论](https://groups.google.com/g/jenkinsci-dev/c/cYi4GyG7Il8/m/oQ2m0C3UAgAJ)、[治理会议记录和录音](https://community.jenkins.io/t/governance-meeting-jan-26-2022/1348)）。
