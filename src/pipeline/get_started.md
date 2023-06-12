# 管道入门

**Getting started with Pipeline**

[如前](../pipeline.md) 所述，Jenkins Pipeline 是一套插件，支持在 Jenkins 中实现和集成持续交付管道。Pipeline 提供了一套可扩展的工具，通过 Pipeline DSL 将简单到复杂的交付管道 “作为代码” 建模。

本节描述了如何开始在 Jenkins 中创建咱们的管道项目，并介绍了创建和存储 `Jenkinsfile` 的各种方式。


## 前提条件

要使用 Jenkins 管道，咱们将需要：

- Jenkins 2.x 或更新版本；（1.642.3 之后的旧版本可以工作，但不建议使用。）

- 管道插件，其作为 “建议的插件” 的一部分被安装（在 [安装 Jenkins](../Ch01_Installing_Jenkins.md) 后通过运行 [安装后设置向导](../installation/docker.md#安装后设置向导) 时指定）。

请在在 [管理插件](../managing/plugins.md) 中阅读有关如何安装和管理插件的更多信息。
