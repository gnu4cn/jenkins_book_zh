# 在流水线中使用 Docker

**Using Dokcer with Pipeline**


许多组织都使用 [Docker](https://www.docker.com/) 来统一其跨机器的构建和测试环境，并为部署应用程序提供一种有效的机制。从 Pipeline 2.5 及以上版本开始，Pipeline 内建了对从 `Jenkinsfile` 中与 Docker 进行交互的支持。

本节将介绍在 `Jenkinsfile` 中使用 Docker 的基本知识，但不包括 Docker 的基本原理，那可以在 [Docker入门指南](https://docs.docker.com/get-started/) 中阅读。


## 执行环境的定制

**Customizing the execution environment**


Pipeline 旨在轻松地使用 [Docker](https://docs.docker.com/) 镜像作为单个 [阶段](../glossary.md#阶段) 或整个 Pipeline 的执行环境。这意味着用户可以为他们流水线定义所需的工具，而不必手动配置代理。实际上，任何可以 [打包在 Docker 容器中的工具](https://hub.docker.com/)，只需对 `Jenkinsfile` 进行小的编辑，就可以轻松使用。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent {
        docker { image 'node:18.16.0-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```



<details>
<summary>切换至脚本化 Pipeline</summary>

```groovy
// Jenkinsfile (脚本化 Pipeline)
node {
    /* Requires the Docker Pipeline plugin to be installed */
    docker.image('node:18.16.0-alpine').inside {
        stage('Test') {
            sh 'node --version'
        }
    }
}
```
</details>



当这个流水线执行时，Jenkins 将自动启动指定的容器并在其中执行所定义的步骤：


```console
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] sh
[guided-tour] Running shell script
+ node --version
v16.13.1
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
```


### 工作区同步

**Workspace synchronization**


简而言之：如果保持工作区与其他阶段的同步是很重要的，那么请使用 `reuseNode true`。否则，docker 化的阶段则可以在任何其他代理或同一代理上运行，不过是在临时工作区。


默认情况下，对于容器化阶段，Jenkins 会这样做：


- 挑选出任何一个代理；

- 创建新的空工作区；

- 克隆流水线代码到其中；

- 将这个新的工作区挂载到容器中。


如果咱们有多个 Jenkins 代理，那么咱们的容器化阶段可在其中任何一个上启动。

当 `reuseNode` 被设置为 `true` 时：将不会创建新的工作区，而当前代理的当前工作区将被挂载到容器中，并且容器将在同一节点启动，因此整个数据将被同步。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'gradle:6.7-jdk11'
                    // Run the container on the node specified at the
                    // top-level of the Pipeline, in the same workspace,
                    // rather than on a new node entirely:
                    reuseNode true
                }
            }
            steps {
                sh 'gradle --version'
            }
        }
    }
}
```


<details>
<summary>切换至脚本化 Pipeline</summary>

```groovy
// Jenkinsfile (脚本化 Pipeline)
// 选项 “reuseNode true” 目前在脚本化流水线中不支持。
```
</details>
