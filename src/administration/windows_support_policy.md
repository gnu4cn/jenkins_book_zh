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





（End）


