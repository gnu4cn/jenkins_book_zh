# 使用 `Jenkinsfile`

本小节以 [Pipeline 入门](./get_started.md) 中的信息为基础，介绍了更多有用的步骤、常见的模式，并演示了一些非简单的 `Jenkinsfile` 例子。

{{#include ../pipeline.md:35:43}}

Pipeline 支持 [两种语法](./syntax.md)，声明式（Pipeline 2.5 中引入）与脚本化 Pipeline。两种语法都支持构建持续交付的 Pipeline。两者都可用于在 Web UI 或 `Jenkinsfile` 中定义 Pipeline，尽管通常认为创建 `Jenkinsfile` 并将该文件签入到源代码控制库中是最佳做法。


## 创建 `Jenkinsfile`

正如 [在 SCM 中定义 Pipeline](./get_started.md#在-scm-中) 所讨论的，`Jenkinsfile` 是一个包含 Jenkins 流水线定义的文本文件，并被签入到源代码控制系统中。请考虑下面这个 Pipeline，他实现了一个基本三阶段的持续交付 Pipeline。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```

<details>
    <summary>切换到脚本化 Pipeline</summary>

```groovy
// Jenkinsfile (脚本化 Pipeline)
node {
    stage('Build') {
        echo 'Building....'
    }
    stage('Test') {
        echo 'Testing....'
    }
    stage('Deploy') {
        echo 'Deploying....'
    }
}
```
</details>


并非所有管道都会有这同样的三个阶段，但对于大多数项目来说，定义出他们是一个很好的起点。下面的小节将演示在 Jenkins 的测试安装中创建和执行一个简单的 Pipeline。

> 假设已经为项目设置了一个源代码控制仓库，并按照 [这些说明](./get_started.md#在-scm-中) 在 Jenkins 中定义了一个 Pipeline。

使用文本编辑器，最好是支持 [Groovy](http://groovy-lang.org/) 语法高亮的，在项目的根目录下创建一个新的 `Jenkinsfile`。

上面的声明式 Pipeline 示例包含了实现持续交付 Pipeline 的最小必要结构。[`agent` 指令](./syntax.md#agent) 是必需的，他指示 Jenkins 为管道分配一个执行器，executor，与工作空间，workspace。如果没有 `agent` 指令，声明式 Pipeline 不仅无效，而且不能做任何工作! 默认情况下，·`agent` 指令确保源码库被签出，并提供给后续阶段的步骤使用。

`stages` 指令和 `steps` 指令也是一个有效的声明式 Pipeline 所必需的，因为他们指示 Jenkins 执行什么，在哪个阶段执行。

> 对于脚本化流水线的更高级用法，上面的示例 `node` 是关键的第一步，因为他为流水线分配了执行器和工作空间。本质上，若没有 `node`，Pipeline 就无法完成任何工作！在 `node` 中，首要任务是签出该项目的源代码。由于 `Jenkinsfile` 是直接从源代码控制系统中提取的，Pipeline 提供了一种快速简便的方法来访问源代码的正确修订。
>
```groovy
// Jenkinsfile (脚本化 Pipeline)
node {
    checkout scm // 1
    /* .. snip .. */
}
```
> 1. `checkout` 步骤将从源代码控制系统中签出代码；`scm` 是一个特殊的变量，他指示签出步骤克隆触发该管道运行的特定修订。

### 构建


