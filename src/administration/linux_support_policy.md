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
