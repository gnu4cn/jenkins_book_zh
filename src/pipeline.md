# 管道

**Pipeline**

本章涵盖了 Jenkins Pipeline 功能的所有推荐方面，包括如何：

- [Pipeline 入门](./pipeline/get_started.md) -- 涵盖了如何通过 Blue Ocean、经典 UI 或 SCM 来定义 Jenkins 管道（即咱们的 `Pipeline`）；

- [创建和使用 `Jenkinsfile`](./pipeline/jenkinsfile.md) - 涵盖了关于如何制作和构建咱们的 `Jenkinsfile` 的用例情景；

- 使用 [分支和拉取请求](./pipeline/branches_and_prs.md)；

- [在 Pipeline 中使用 Docker]() -- 涵盖 Jenkins 如何在代理/节点上调用 Docker 容器（从 `Jenkinsfile`）来构建咱们的 Pipeline 项目；

- [使用共享库扩展管道](./pipeline/shared-libraries.md)

- 使用不同的 [开发工具](./pipeline/development.md) 来促进咱们管道的创建，以及

- 使用 [Pipeline 语法](./pipeline/syntax.md) - 这一页是全部声明式 Pipeline 语法的综合参考。


关于Jenkins用户手册的内容概述，请参见 [用户手册概述](./Ch00_Overview.md)。


## 什么是 Jenkins 管道？

Jenkins Pipeline（或者简单的有着大写字母 “P” 的 “Pipeline”）是一套插件，支持实施与集成 *持续交付管道，continuous delivery pipelines* 到 Jenkins 中。

而 *持续交付（continous delivery, CD）管道*，则是咱们从版本控制系统，将软件一直交付到咱们用户与客户过程的一种自动化表达。咱们软件的每一个变化（在源码控制中所提交的）都要经过一个复杂的过程，才能被发布。这个过程包括以可靠和可重复的方式构建软件，以及通过多个阶段的测试和部署来推进构建的软件（称为一次 "构建"）。



