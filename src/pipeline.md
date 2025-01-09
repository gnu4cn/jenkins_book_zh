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

Pipeline 提供了一套可扩展的工具，用于通过 [Pipeline 领域特定语言（DSL）语法](./pipeline/syntax.md)，将简单到复杂的交付管道 “作为代码, as code” 建模。

Jenkins 管道的定义会被写入一个文本文件（称为 `Jenkinsfile`），而这个文件又可被提交到项目的源码控制仓库。这即是 “管道即代码，Pipeline-as-code” 的基础；将 CD 管道作为应用程序的一部分，像其他代码一样进行版本管理和审阅。

创建一个 `Jenkinsfile` 并将其提交到源代码控制，可以带来以下数种直接好处：

- 为所有分支和拉取请求自动创建管道 Pipeline 构建过程；

- 对管道 Pipeline 进行代码审阅/迭代（连同剩余的源代码）；

- 管道的审计跟踪，audit trail for the Pipeline；

- 管道的 [单一真相来源](https://en.wikipedia.org/wiki/Single_source_of_truth)，可由项目的多名成员查看和编辑。


虽然在 Web UI 中或使用 `Jenkinsfile` 定义管道的语法是相同的，但通常认为最佳做法是在 `Jenkinsfile` 中定义管道并将其签入源码管理系统。


### 声明式与脚本式的管道语法

**Declarative versus Scripted Pipeline syntax**

`Jenkinsfile` 可以两种语法写就 -- 声明式与脚本式语法。

声明式管道和脚本式管道的构建从根本上讲是不同的。声明式管道是 Jenkins 管道的一个较新的功能，他：

- 提供了比脚本管道语法更丰富的语法特性，并且

- 旨在使编写和阅读流水线代码更加容易。

然而，写进 `Jenkinsfile` 的许多单独的语法组件（或 "步骤"）对声明式和脚本式管道都是通用的。在下面的 [Pipeline 概念](#pipeline-concepts) 和 [Pipeline 语法概述](#管道语法概述) 中阅读更多关于这两类语法的不同。


## 为何要用管道？

从根本上说，Jenkins 是一个自动化引擎，支持许多自动化模式。Pipeline 在 Jenkins 上增加了一套强大的自动化工具，支持从简单的持续集成到全面的 CD 管道的用例。通过建立一系列相关任务的模型，用户可以利用 Pipeline 的许多功能：

- **代码，code**：管道是用代码实现的，通常被检入到源代码控制系统，使团队有能力编辑、审查和迭代他们的交付管道；

- **持久性，durable**：持久性： 管道可以在 Jenkins 控制器的计划和非计划重启中存活；

- **可暂停，pausable**：管道可选择性停止并等待人类的输入或批准，然后继续其运行；

- **通用性，versatile**：管道支持复杂的现实世界的 CD 要求，包括分叉/连接（fork/join）、循环，及并行完成工作的能力；

- **可扩展性，extensible**： Pipeline 插件支持对其 DSL 的定制扩展，以及与其他插件集成的多种选择。

虽然 Jenkins 一直允许以初级形式将 Freestyle 的作业串联在一起，以执行顺序任务，但 Pipeline 使这个概念成为 Jenkins 的头等公民。

> 一些额外插件，如复制工件 Copy Artifact、参数化触发器 Parameterized Trigger 与提升构建 Promoted Builds 等插件，被用于利用 Freestyle 的作业实现复杂行为。

*Jenkins 中自由风格与管道的区别*


[![Freestyle 与 Pipeline 构建的区别](https://img.youtube.com/vi/IOUm1lw7F58/0.jpg)](https://www.youtube.com/watch?v=IOUm1lw7F58)


**总结**：

1. 自由风格的构建可以通过 “构建触发器” 中的 “其他工程构建后触发”，配置为类似 Pipeline 方式的构建；

2. 这样配置后的多个自由风格构建，会受 Jenkins 主控制器重启等的影响，但 Pipeline 构建不会；

3. 多个自由风格的构建与 Pipleline 的构建，不是苹果与苹果的比较，而是苹果与橙子的比较，有着本质上的不同。


基于可扩展性这一 Jenkins 核心价值，在 [Pipeline 共享库](./pipeline/shared-libraries.md) 下，对于其用户和插件开发者来说，Pipeline 也同时是可扩展的。

下面的流程图是在 Jenkins Pipeline 中轻松建模的一个 CD 场景的示例：

![真实世界的管道流程](./images/realworld-pipeline-flow.png)


## 管道的一些概念

以下概念是 Jenkins Pipeline 的关键方面，他们与 Pipeline 语法密切相关（见下文 [概述](#管道语法概述)）。


### 管道

**Pipeline**

Pipeline 是用户定义的持续交付 CD 流水线的一个模型。管道的代码定义了咱们整个构建过程，通常包括构建应用程序、测试和交付等阶段。


同样，`pipeline` 的代码块则是 [声明式管道语法的关键部分](#声明式管道基础)。


### 节点

**Node**

节点是属于 Jenkins 环境一部分的机器，他能够执行一个管道。

同样，`node` 代码块是 [脚本化管道语法的关键部分](#脚本化管道基础)。


### 阶段

**Stage**


`stage`代码块定义了贯穿整个管道执行过程的概念上不同的任务子集（例如 “构建 Build”、“测试 Test” 与 “部署 Deploy” 等阶段），许多插件使用他来可视化或呈现 Jenkins 管道的状态/进度。

> 用到 `stage` 代码块来显示状态/进度的插件有：[Blue Ocean](./blue_ocean.md)、[Pipeline: Stage View 插件](https://plugins.jenkins.io/pipeline-stage-view) 等。


### 步骤

**Step**


单项的任务。基本上，一个 `step` 会告诉 Jenkins 在某个特定时间点（或进程中的 "step"）做 *什么*。例如，要执行 shell 命令 `make`，就要使用 `sh` 步骤：`sh 'make'`。当某个插件扩展了 Pipeline DSL，通常意味着该插件实现了一个新 `step`。



## 管道语法概述

以下流水线代码框架说明了 [声明式管道语法](#声明式管道基础) 和 [脚本化管道语法](#脚本化管道基础) 之间的根本区别。

请注意，[阶段](#阶段) 和 [步骤](#步骤)（上文）均为声明式和脚本化管道语法的共同要素。


### 声明式管道基础


在声明式管道语法中，`pipeline` 代码块定义了整个管线所完成的全部工作。


```groovy
// Jenkinsfile（声明式管道，Declarative Pipeline）

pipeline {
    agent any   // 1
    stages {
        stage('Build') { //2
            steps {
                // 3
            }
        }
        stage('Test') { // 4
            steps {
                // 5
            }
        }
        stage('Deploy') { // 6
            steps {
                // 7
            }
        }
    }
}
```

1. `agent any` 在任何可用的代理上执行该管道或其任何阶段；

2. `stage('Build')` 定义了 “构建 Build” 阶段；

3. 执行与 “构建 Build” 阶段相关的一些步骤；

4. 定义了 “测试 Test” 阶段；

5. 执行与 “测试” 阶段相关的一些步骤；

6. 定义了 “部署 Deploy” 阶段；

7. 执行与 “部署” 阶段相关的一些步骤。


### 脚本化管道基础

**Scripted Pipeline fundamentals**


在脚本化管道语法中，一个或多个 `node` 代码块在整个管线中完成核心工作。虽然这不是脚本化管道语法的强制性要求，但将咱们管道的工作限制在 `node` 代码块内有两方面的作用：

- 通过向 Jenkins 队列添加一个项目来调度 `node` 代码块中包含步骤的运行。只要某个节点上的执行器有空，这些步骤就会运行；

- 创建一个工作区（特定于该管道的目录），在那里可以对从源代码控制中签出的文件进行工作。

    **注意**：根据咱们的 Jenkins 配置，某些工作空间在一段时间不活动后可能不会被自动清理。请参阅 [JENKINS-2111](https://issues.jenkins.io/browse/JENKINS-2111) 中链接的那些工单与讨论了解更多信息。


```groovy
// Jenkinsfile （脚本化管道）
node { // 1
    stage('Build') { // 2
        // 3
    }
    stage('Test') { // 4
        // 5
    }
    stage('Deploy') { // 6
        // 7
    }
}
```

1. 在任何可用的代理上执行该管道或其任何阶段；

2. 定义了 “构建 Build” 阶段。`stage` 代码块在脚本化管道语法中是可选的。然而，在脚本化管道中实现 `stage` 块，可以在 Jenkins 用户界面中更清楚地呈现每个阶段的任务/步骤子集；

3. 执行一些与 “构建 Build” 阶段相关的步骤；

4. 定义了 “测试 Test” 阶段；

5. 执行一些与 “测试 Test” 相关的步骤；

6. 定义了 “部署 Deploy” 阶段；

7. 执行一些与 “部署” 阶段相关的步骤。



## 管道示例

下面是一个使用声明式管道语法的 `Jenkinsfile` 的例子 -- 点击下面的 **切换脚本化管道** 链接，就可以访问其脚本化语法的等体：


```groovy
// Jenkinsfile（声明式管道）
pipeline { // 1
    agent any // 2
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') { // 3
            steps { // 4
                sh 'make' // 5
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml' // 6
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```

<details>
    <summary>切换脚本化 Pipeline</summary>

```groovy
node { // 5
    stage('Build') { // 3
        sh 'make' // 5
    }
    stage('Test') {
        sh 'make check'
        junit 'reports/**/*.xml' // 6
    }
    if (currentBuild.currentResult == 'SUCCESS') {
        stage('Deploy') {
            sh 'make publish' // 5
        }
    }
}
```
</details>

1. `pipeline` 是声明式管道特有的语法，他定义了一个 "块"，包含执行整个 Pipeline 的所有内容和指令；

2. `agent` 是声明式管道特有的语法，给 Jenkins 指明了为整个管道分配执行器（an executor，在某个节点上）和工作空间；

3. `stage` 是一个语法块，用于描述 [该管道的某个阶段](#阶段)。请在 [Pipeline 语法](./pipeline/syntax.md) 页面上阅读更多关于声明式 Pipeline 语法 `stage` 块的内容。如上所述，在脚本化管道语法中，阶段块是可选的。

4. `steps` 是声明式管道特有的语法，描述了在这个 `stage` 要运行的步骤；

5. `sh` 是一个 Pipeline [步骤](#步骤)（由 [Pipeline: 节点与过程插件](https://plugins.jenkins.io/workflow-durable-task-step) 提供），执行给定的 shell 命令；

6. `junit` 是另一个 Pipeline [步骤](#步骤)（由 [JUnit 插件](https://plugins.jenkins.io/junit) 提供），用于汇总测试报告；


请在 [管道语法](./pipeline/syntax.md) 页面阅读更多有个管道语法的内容。


（End）


