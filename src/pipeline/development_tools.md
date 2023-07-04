# 流水线开发工具

**Pipeline Development Tools**

Jenkins 的流水线包含了 [内置文档](#内建文档) 和 [代码片段生成器](#代码片段生成器)，他们是开发时的关键资源。他们提供了根据当前安装的 Jenkins 与相关插件的版本进行定制的详细帮助和信息。在本节中，我们将讨论可能有助于开发 Jenkins 流水线的其他工具和资源。


## 命令行的流水线静态分析程序

**Command-line Pipeline Linter**

Jenkins 可以在实际运行某个声明式流水线之前从命令行中验证，或者说 “[静态分析，lint](https://en.wikipedia.org/wiki/Lint_(software))”。这可以通过使用 Jenkins CLI 命令或带适当参数的 HTTP POST 请求来完成。我们建议使用 [SSH 接口](../managing/jenkins_cli.md#通过-ssh-使用命令行界面) 来运行静态分析程序。关于如何正确配置 Jenkins 以实现安全命令行访问的细节，请参见 [Jenkins CLI 文档](../managing/jenkins_cli.md)。


