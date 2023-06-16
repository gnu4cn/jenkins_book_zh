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


