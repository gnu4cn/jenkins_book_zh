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

对于许多项目来说，Pipeline 中 “工作” 的开始是 “构建” 阶段。通常情况下，流水线的这一阶段将是源代码组装、编译或打包的地方。`Jenkinsfile` **不** 是既有构建工具，如 GNU/Make、Maven、Gradle 等的替代品，而是可以被视为一个胶水层，a glue layer，将项目开发生命周期的多个阶段（构建、测试、部署等）结合在一起。

Jenkins 有可以调用几乎所有通用构建工具的许多插件，但下面这个例子将简单地从 shell 步骤（`sh`）中调用 `make`。`sh` 步骤假定系统是基于 Unix/Linux 的，对于基于 Windows 的系统，可以使用 `bat` 来代替。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make' // 1
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true // 2
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
        sh 'make' // 1
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true // 2
    }
}
```
</details>

1. `sh` 步骤调用 `make` 命令，且只有当该命令返回的退出代码为零时才会继续。任何非零的退出代码都将导致 Pipeline 失败；

2. `archiveArtifacts` 会捕获所构建出的匹配包含模式 ( `**/target/*.jar` ) 的文件，并将他们保存到 Jenkins 控制器以供后面的索取。

> 归档成品（`archiveArtifacts`）不能替代使用外部成品库，如 Artifactory 或 Nexus，只应考虑用于基本报告和文件存档。


### 测试

运行自动化测试是任何成功的持续交付过程的一个关键组成部分。因此，Jenkins 有着一些由一些插件所提供得测试记录、报告和可视化设施。从根本上说，当出现测试失败时，让 Jenkins 记录失败的情况，以便在 Web UI 中进行报告和可视化，是非常有用的。下面的例子使用了由 JUnit 插件提供的 `junit` 步骤。

在下面的例子中，如果测试失败，这个 Pipeline 就被标记为 "不稳定，unstable"，在 Web UI 中用一个黄色的球表示。基于所记录的测试报告，Jenkins 还可以提供历史趋势分析与可视化。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` returns non-zero on test failures,
                * using `true` to allow the Pipeline to continue nonetheless
                */
                sh 'make check || true' // 1
                junit '**/target/*.xml' // 2
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
    /* .. snip .. */
    stage('Test') {
        /* `make check` returns non-zero on test failures,
         * using `true` to allow the Pipeline to continue nonetheless
         */
        sh 'make check || true' // 1
        junit '**/target/*.xml' // 2
    }
    /* .. snip .. */
}
```
</details>


1. 使用一个内联的 shell 条件（ `sh 'make check || true'` ）确保 `sh` 步骤总是看到一个零的退出代码，给 `junit` 步骤以机会来捕获和处理测试报告。这方面的替代方法在下面的 [处理失败](#处理失败) 部分有更详细的介绍；

2. `junit` 会捕获并关联符合包含模式（matching the inclusion pattern, `**/target/*.xml`）的 JUnit XML 文件。


### 部署

部署可以意味着各种步骤，这取决于项目或组织的要求，可能是从发布构建的成品到 Artifactory 服务器，到将代码推送到生产系统等的任何东西。

在示例 Pipeline 的这个阶段，“构建 Build” 和 “测试 Test” 阶段都已成功执行。实质上，“部署 Deploy” 阶段将只在假设前几个阶段成功完成的情况下执行，否则这个 Pipeline 将提前退出。


```groovy
// Jenkinsfile (声明式 Pipeline)

pipeline {
    agent any

    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' // 1
              }
            }
            steps {
                sh 'make publish'
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
    /* .. snip .. */
    stage('Deploy') {
        if (currentBuild.result == null || currentBuild.result == 'SUCCESS') { // 1
            sh 'make publish'
        }
    }
    /* .. snip .. */
}
```
</details>

1. 访问 `currentBuild.result` 变量允许 Pipeline 确定是否有任何测试失败。在这种情况下，该值将是 `UNSTABLE`。

假设示例 Jenkins Pipeline 中的一切都已成功执行，那么每个成功的 Pipeline 运行都将在 Jenkins 中存档相关的构建成品、报告测试结果以及完整的控制台输出。

> 脚本管道可以包括条件测试（如上所示）、循环、`try`/`catch`/`finally` 代码块并甚至是函数。下一节将更详细地介绍这种高级的脚本管道语法。


## 使用咱们的 `Jenkinsfile`

**Working with your `Jenkinsfile`**

接下来的小节提供了处理以下内容的详细信息：

- 咱们 `Jenkinsfile` 中专门的流水线语法，以及

- 构建咱们的应用程序或 Pipelline 项目所必需的一些流水线语法特性及功能。


### 使用环境变量

**Using environmental variables**

Jenkins Pipeline 通过全局变量 `env` 暴露出环境变量，该变量可在 `Jenkinsfile` 的任何地方使用。在 Jenkins 流水线内可访问的环境变量的完整列表记录在 `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env` 中，包括：


#### `BUILD_ID`

当前构建的 ID，与在 Jenkins 1.597 以上版本中创建的构建的 `BUILD_NUMBER` 相同。


#### `BUILD_NUMBER`

当前构建的编号，如 "153"。


#### `BUILD_TAG`

`jenkins-${JOB_NAME}-${BUILD_NUMBER}` 的字符串。便于放在资源文件、`.jar` 文件等中，以易于识别。


#### `BUILD_URL`

可以找到这次构建结果的 URL（例如 `http://buildserver/jenkins/job/MyJobName/17/`）。


#### `EXECUTOR_NUMBER`

识别当前执行器，the current executor（在同一台机器的执行器中）执行此次构建的唯一编号。这就是咱们在 “构建执行器状态，build executor status”（`${YOUR_JENKINS_URL}/computer/`）中看到的数字，只不过这个数字是从 `0` 开始的，而不是 `1`。


#### `JAVA_HOME`

如果咱们的作业被配置为使用特定的 JDK，这个变量将被设置为指定的 JDK 的 `JAVA_HOME`。当这个变量被设置时，`PATH` 也会被更新，以包括 `JAVA_HOME` 的 `bin` 子目录。


#### `JENKINS_URL`

Jenkins 的完整 URL，如 `https://example.com:port/jenkins/`（注意：只有在 “系统配置” 中设置了 Jenkins 的 URL 时才可用）。


#### `JOB_NAME`

本次构建的项目名称，如 `foo` 或 `foo/bar`。


#### `NODE_NAME`

当前构建所运行的节点名称。对于 Jenkins 控制器，设置为 `master`。


#### `WORKSPACE`

工作区的绝对路径。

引用或使用这些环境变量可以像访问某个 Groovy [Map](http://groovy-lang.org/syntax.html#_maps) 中的任何键一样完成，比如说：


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
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
    echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
}
```
</details>


#### 设置环境变量

根据所使用的是声明式还是脚本化管道，Jenkins Pipeline 中设置环境变量的方式有所不同。


声明式管道支持 `environment` 指令，而脚本化管道的用户则必须使用 `withEnv` 步骤。


```groovy
// Jenkinsfile (声明式 Pipeline)

pipeline {
    agent any

    environment { // 1
        CC = 'clang'
    }

    stages {
        stage('Example') {

            environment { // 2
                DEBUG_FLAGS = '-g'
            }

            steps {
                sh 'printenv'
            }
        }
    }
}
```

<details>
<summary>切换至脚本化 Pipeline</summary>

```groovy
node {
    /* .. snip .. */
    withEnv(["PATH+MAVEN=${tool 'M3'}/bin"]) {
        sh 'mvn -B verify'
    }
}
```
</details>


1. 在顶层 `pipeline` 代码块中使用的 `environment` 指令将适用于该流水线内的所有步骤；

2. 在莫个 `stage` 内定义的 `environment` 指令将只把所给定的环境变量应用于该 `stage` 里的步骤。


#### 动态地设置环境变量

环境变量可以在运行时设置，可以被 shell 脚本（`sh`）、Windows 批处理脚本（`bat`）和 PowerShell 脚本（ `powerhell`）使用。每个脚本都可以 `returnStatus` 或 `returnStdout`。[关于脚本的更多信息](./nodes_and_processes.md)。
