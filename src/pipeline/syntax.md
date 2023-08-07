# 流水线语法

**Pipeline Syntax**

这一小节是建立在 [流水线入门](./get_started.md) 中介绍的信息之上的，应仅作为参考。关于如何在实际示例中使用 Pipeline 语法的更多信息，请参考本章的 [使用 `Jenkinsfile`](./jenkinsfile.md) 部分。从 Pipeline 插件的 2.5 版本开始，Pipeline 就支持了两种不同的语法，详见下文。关于两种语法各自的优点和缺点，请参阅 [语法比较](#两种语法比较)。

正如本章开头所讨论的，流水线的最基本部分是 “步骤, step”。基本上，步骤会告诉 Jenkins 要做 *什么*，并作为声明式和脚本化流水线语法的基础构建块，the basic building block for both Declarative and Scripted Pipeline syntax。

关于可用步骤的概述，请参考 [Pipeline 步骤参考](../pipeline_steps_ref.md)，其中包含了 Pipeline 内置步骤与由插件所提供步骤的全面清单。


## 声明式流水线

**Declarative Pipeline**

声明式流水线是 Jenkins 流水线的一个相对较新的补充，他在流水线子系统的基础上提出了一个更简化和有独立见解的语法。

> `2.5` 版本的 “管道插件” 引入了对声明式流水线语法的支持。

所有有效声明式流水线都必须被包含在一个 `pipeline` 代码块中，例如：


```groovy
pipeline {
    /* insert Declarative Pipeline here */
}
```

在声明式流水线中有效的基本语句和表达式遵循了与 Groovy 语法相同的规则，但有以下例外：

- 流水线的顶层必须是个代码块，具体为：`pipeline { }`；

- 没有作为语句分隔符的分号。每条语句都必须在自己的行上；

- 代码块必须只由小节、指令、步骤或赋值语句组成；

- 属性引用语句，a property reference statement，会被当作无参数的方法调用来处理。因此，举例来说，`input` 就会被当作 `input()` 处理。


咱们可以使用 [声明式指令生成器](./get_started.md#声明式指令生成器) 来帮助咱们开始配置声明式流水线中的指令和小节。


### 局限性

**Limitations**

目前存在一个 [未解决的问题](https://issues.jenkins.io/browse/JENKINS-37984)，该问题限制了 `pipeline{}` 代码块内代码的最大数量。此限制不适用于脚本化流水线。


### 小节

**Sections**


声明式流水线中的小节通常会包含一个或多个的 [指令](#指令) 或 [步骤](#步骤)。


#### `agent`

根据 `agent` 小节的放置位置，其会指定出整个流水线或某个特定阶段在 Jenkins 环境中何处执行。该小节必须在 `pipeline` 代码块内的顶层定义，但阶段级别的使用则是可选的。


<table>
  <tr>
    <th>是否必需</th>
    <td>是</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>下面会讲到</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在 <code>pipeline</code> 代码块的顶层及各个 <code>stage</code> 代码块中</td>
  </tr>
</table>


**顶级代理和阶段性代理之间的区别**

**differences between top level agents and stage level agents**


当应用了 `options` 指令时，在顶层或阶段级别添加某个代理时，会有一些细微的差别。请查看 [选项](#选项) 部分了解更多信息。



**顶层的代理**

**Top Level Agents**


在流水线顶层声明的代理中，某个代理会被分配，然后 `timeout` 选项就会被应用。用于分配代理的时间 **不会包括** 在 `timeout` 选项所设定的限制中。


```groovy
pipeline {
    agent any
    options {
        // Timeout counter starts AFTER agent is allocated
        timeout(time: 1, unit: 'SECONDS')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**阶段的代理**

**Stage Agents**

在阶段内声明的代理中，选项在分配 `agent` 和检查任何 `when` 条件 **之前** 就会被调用。在这种情况下，当使用 `timeout` 时，其是在 `agent` 分配之前就应用的。用于分配代理的时间会 **包含在** `timeout` 选项所设定的限制中。


```groovy
pipeline {
    agent none
    stages {
        stage('Example') {
            agent any
            options {
                // Timeout counter starts BEFORE agent is allocated
                timeout(time: 1, unit: 'SECONDS')
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

此超时将包括代理配置时间。而由于超时包括了代理配置时间，因此在代理分配延迟的情况下，流水线就可能会失败。


**参数**

**Parameters**


为支持流水线作者们可能拥有的各种用例，`agent` 小节支持几种不同类型的参数。这些参数可以在 `pipeline` 代码块的顶层应用，也可以在每个 `stage` 指令中应用。

- `any`

在任何可用代理上执行流水线或阶段。例如：`agent any`。

- `none`

在 `pipeline` 代码块的顶层应用 `agent none` 时，就不会为整个流水线的运行分配全局代理，而此时每个 `stage` 小节都需要包含自己的 `agent` 小节。例如：`agent none`。

- `label`

在 Jenkins 环境中带有所提供标签的可用代理上执行流水线或阶段。例如： `agent { label 'my-defined-label' }`。

也可以使用标签条件，label conditions： 例如：`agent { label 'my-label1 && my-label2' }` 或 `agent { label 'my-label1 || my-label2' }`。

- `node`

`agent { node { label 'labelName' } }` 的行为与 `agent { label 'labelName' } }` 相同，但 `node` 允许使用其他选项（如 `customWorkspace` 等）。

- `docker`

使用给定的容器执行流水线或阶段，容器将动态配置，be dynamically provisioned, 到预先配置好接受基于 Docker 的流水线的节点上，或者配置到与可选定义 `label` 参数相匹配的节点上。`docker` 还可选地接受 `args` 参数，其中可能包含直接传递给 `docker run` 调用的参数，以及 `alwaysPull` 选项，这时即使镜像名称已经存在，也会强制执行 `docker pull`。例如： `agent { docker 'maven:3.9.3-eclipse-temurin-17' }` 或

```groovy
agent {
    docker {
        image 'maven:3.9.3-eclipse-temurin-17'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
}
```

`docker` 还可选地接受 `registryUrl` 和 `registryCredentialsId` 参数，这有助于指定要使用的 Docker 注册表及其凭据。参数 `registryCredentialsId` 可单独用于 docker 中心的私有存储库。例如：


```groovy
agent {
    docker {
        image 'myregistry.com/node'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
```

- `dockerfile`

使用源代码库中包含的 `Dockerfile` 构建出的容器执行流水线或阶段。要使用此选项，`Jenkinsfile` 必须从 **多分支流水线** 或 **SCM 流水线** 中加载。通常情况下，这是源代码库根目录下的 `Dockerfile`： `agent { dockerfile true }`。如果在其他目录下构建 `Dockerfile`，请使用 `dir` 选项： `agent { dockerfile { dir 'someSubDir' } }`。如果咱们的 `Dockerfile` 有其他名字，可以使用 `filename` 选项指定文件名。咱们可以使用 `additionalBuildArgs` 选项为 `docker build ...` 命令传递额外参数，比如 `agent { dockerfile { additionalBuildArgs '--build-arg foo=bar' } }`。例如，对于有着包含 `build/Dockerfile.build` 文件的某个版本库，其构建参数的 `version` 为：

```groovy
agent {
    // Equivalent to "docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        additionalBuildArgs  '--build-arg version=1.0.2'
        args '-v /tmp:/tmp'
    }
}
```

像参数 `docker` 一样，`dockerfile` 也可选择接受 `registryUrl` 和 `registryCredentialsId` 参数，这有助于指定要使用的 Docker 注册表及其凭据。例如：


```groovy
agent {
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}
```

- `kubernetes`

在 Kubernetes 集群上部署的 pod 中执行流水线或阶段。要使用该选项，`Jenkinsfile` 必须从 **多分支流水线** 或 **SCM 流水线** 中加载。Pod 模板是在 `kubernetes { }` 代码块内定义的。例如，如果咱们想要某个其中内含了 Kaniko 容器的 pod，就可以按如下方式定义：


```groovy
agent {
    kubernetes {
        defaultContainer 'kaniko'
        yaml '''
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
      - name: aws-secret
        mountPath: /root/.aws/
      - name: docker-registry-config
        mountPath: /kaniko/.docker
  volumes:
    - name: aws-secret
      secret:
        secretName: aws-secret
    - name: docker-registry-config
      configMap:
        name: docker-registry-config
'''
   }
```

咱们需要为 Kaniko 创建一个 `aws-secret` 密钥，以便能够对 ECR 进行身份验证。该秘密应包含 `~/.aws/credentials` 的内容。另一个卷则是个 `ConfigMap`，其中应包含 ECR 注册表的端点，the endpoint of your ECR registry。例如：

```groovy
{
      "credHelpers": {
        "<your-aws-account-id>.dkr.ecr.eu-central-1.amazonaws.com": "ecr-login"
      }
}
```

请参考以下示例：[https://github.com/jenkinsci/kubernetes-plugin/blob/master/examples/kaniko.groovy](https://github.com/jenkinsci/kubernetes-plugin/blob/master/examples/kaniko.groovy)


**常用选项**

下面是可用于两种或更多的代理实现的一些选项。除非明确说明，否则这些选项不是必需的。


- `label`

一个字符串。运行流水线或单个 `stage` 所依据的标签或标签条件。

该选项适用于 `node`、`docker` 和 `dockerfile`，对于 `node` 则是必需的。

- `customWorkspace`

一个字符串。在此定制工作区中，而非默认的工作区运行此代理所应用到的流水线或单个阶段。可以是相对路径（在这种情况下，定制工作区将位于节点上工作区根目录之下），也可以是绝对路径。例如：

```groovy
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```

此选项对 `node`、`docker` 及 `dockerfile` 有效。

- `reuseNode`

一个布尔值，默认为 `false`。如果为 `true`，那么将在流水线顶层指定的节点上，在同一工作区中运行容器，而不是在全新的节点上运行。

此选项对 `docker` 和 `dockerfile` 有效，且仅在某个 `agent` 上对单个 `stage` 使用时有效。

- `args`

一个字符串。传递给 `docker run` 的运行时参数。


该选项适用于 `docker` 和 `dockerfile`。


> **注意**：下面两个示例是 `agent` 小节的总体示例，并非 “常用选项” 或 `args` 的示例。


*示例 1：Docker 构建代理，声明式流水线*


```groovy
pipeline {
    agent { docker 'maven:3.9.3-eclipse-temurin-17' } // 1

    stages {
        stage('Example Build') {
            steps {
                sh 'mvn -B clean verify'
            }
        }
    }
}
```

1. 在以给定名称及标签（`maven:3.9.3-eclipse-temurin-17`）新创建的容器中执行此流水线中定义的所有步骤。


*示例 2：阶段级别的代理小节*

```groovy
pipeline {
    agent none // 1

    stages {
        stage('Example Build') {
            agent { docker 'maven:3.9.3-eclipse-temurin-17' }  // 2

            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }

        stage('Example Test') {
            agent { docker 'openjdk:17-jre' } // 3

            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

1. 在流水线顶层定义 `agent none` 可确保不会不必要地分配 [执行器，an Executor](../glossary.md#exector)。使用 `agent none` 还能强制各个 `stage` 小节都要包含自己的 `agent` 小节；

2. 在使用这个 Docker 镜像新建出的容器中执行本阶段的步骤；

3. 使用与上一阶段不同的 Docker 镜像而新创建出的容器中执行本阶段的步骤。


#### `post`

`post` 小节定义了流水线或阶段运行完成后要运行的一个或多个附加 [步骤](#步骤)（取决于 `post` 在流水线中的位置）。`post` 小节可支持以下任何 [后置条件，post-condition](#后置条件) 代码块：`always`、`changed`、`fixed`、`regression`、`aborted`、`failure`、`success`、`unstable`、`unsuccessful` 及 `cleanup`。这些条件代码块允许根据流水线或阶段的完成状态执行各个条件内的步骤。条件代码块的执行顺序如下所示。

<table>
  <tr>
    <th>是否必需</th>
    <td>不是</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>无</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在 <code>pipeline</code> 代码块的顶层及各个 <code>stage</code> 代码块中</td>
  </tr>
</table>


**后置条件**

**Conditions**


- `always`

无论流水线或阶段运行的完成状态如何，都要运行 `post` 小节中的步骤。

- `changed`

只有在当前的流水线运行的完成状态与上次运行的完成状态不同时，才运行 `post` 小节中的步骤。

- `fixed`

只有在当前的流水线运行成功，而上一次运行失败或不稳定的情况下，才运行 `post` 小节中步骤。

- `regression`

只有当当前流水线运行的或状态为失败、不稳定或中止，且上一次运行成功时，才运行 `post` 小节中步骤。

- `aborted`

只有在当前的流水线运行处于 “中止，aborted” 状态（通常是由于流水线被手动中止）时，才运行 `post` 小节中的步骤。这在网页用户界面中通常以灰色表示。

- `failure`

只有在当前的流水线或阶段运行处于 “失败，failed” 状态（通常在 Web UI 中用红色表示）时，才运行 `post` 小节中步骤。

- `success`

只有在当前的管道或阶段运行处于 “成功，success” 状态（通常在 Web UI 中以蓝色或绿色表示）时，才运行 `post` 小节中的步骤。

- `unstable`

只有在当前的流水线运行处于 “不稳定，unstable” 状态（通常由测试失败、代码违规等引起）时，才运行 `post` 小节中步骤。这种情况在网页用户界面上通常用黄色表示。

- `unsuccessful`

只有在当前的流水线或阶段运行未达到 “成功” 状态时，才运行 `post` 小节中的步骤。这通常会根据前面提到的状态在 Web UI 中显示（对于阶段，如果构建本身不稳定，则可能会触发）。

- `cleanup`

无论流水线或阶段的状态如何，都要在评估完所有其他 `post` 条件后运行此 `post` 条件中的步骤。


*示例 3：后置小节，声明式流水线*


```groovy
pipeline {
    agent any

    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }

    post { // 1
        always { // 2
            echo 'I will always say Hello again!'
        }
    }
}
```

1. 按照惯例，`post` 小节应放在流水线的末尾；

2. [后置条件，post-condition](#后置条件) 代码块包含了与 [步骤](#步骤) 小节相同的[步骤](#步骤)。


#### `stages`

`stages` 小节包含一个或多个的 [`stage`](#stage) 指令序列，是流水线描述的大部分 “工作” 所在。建议 `stages` 为持续交付流程的每个具体部分（如构建、测试和部署等）至少包含一个 [`stage`](#stage) 指令。


<table>
  <tr>
    <th>是否必需</th>
    <td>是</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>无</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在 <code>pipeline</code> 代码块内部，或某个 <code>stage</code> 内部</td>
  </tr>
</table>


*示例 4：阶段，声明式流水线*


```groovy
pipeline {
    agent any

    stages { // 1
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

1. `stages` 小节通常紧随 `agent`、`options` 等指令之后。


#### `steps`


`steps` 小节定义了在给定的 `stage` 指令中要执行的一系列的一个或多个 [步骤](#声明式步骤)。

<table>
  <tr>
    <th>是否必需</th>
    <td>是</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>无</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在各个 <code>stage</code> 代码块内部</td>
  </tr>
</table>


*示例 5：单个步骤，声明式流水线*


```groovy
pipeline {
    agent any

    stages {
        stage('Example') {
            steps { // 1
                echo 'Hello World'
            }
        }
    }
}
```

1. `steps` 小节必须包含一个或多个步骤。


### 指令

**Directives**


#### `environment`

`environment` 指令指定了一系列键值对，根据 `environment` 指令在流水线中的位置，这些键值对将被定义为所有步骤或特定阶段步骤的环境变量。

该指令支持一种特殊的辅助方法 `credentials()`，可用于通过 Jenkins 环境中的标识符访问预定义的凭据。


<table>
  <tr>
    <th>是否必需</th>
    <td>非必需</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>无</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在 <code>pipeline</code> 代码块内，或在 <code>stage</code> 指令里面</td>
  </tr>
</table>


**支持的凭据类型**

- **秘密文本，Secret Text**

所指定的环境变量将被设置为秘密文本，Secret Text，内容。

- **秘密文件，Secret File**

所指定的环境变量将被设置为临时创建的 File 文件的位置。

- **用户名与口令，username and password**

所指定的环境变量将被设置为 `username:password`，另外两个环境变量将被自动定义出来： 分别为 `MYVARNAME_USR` 和 `MYVARNAME_PSW`。

- **带有私钥的 SSH**

所指定的环境变量将被设置为临时创建的 SSH 密钥文件的位置，另外两个环境变量将被自动定义出来： `MYVARNAME_USR` 和 `MYVARNAME_PSW`（保存口令）。

> 不支持的凭据类型会导致流水线失败，并显示以下消息：


```console
org.jenkinsci.plugins.credentialsbinding.impl.CredentialNotFoundException: No suitable binding handler could be found for type <unsupportedType>.
```


**示例 6：秘密文本的凭据，声明式流水线**


```groovy
pipeline {
    agent any
    environment { // 1
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { // 2
                AN_ACCESS_KEY = credentials('my-predefined-secret-text') // 3
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```


1. 在顶层 `pipeline` 代码块中使用的 `environment` 指令将适用于流水线中的所有步骤；

2. 在某个 `stage` 中定义的 `environment` 指令只会将给定的环境变量应用于这个 `stage` 中的步骤；

3. 这个 `environment` 代码块定义了一个辅助方法 `credentials()`，可用于通过 Jenkins 环境中的标识符访问预定义的凭据。


*示例 7：用户名与密码凭据*


```groovy
pipeline {
    agent any
    stages {
        stage('Example Username/Password') {
            environment {
                SERVICE_CREDS = credentials('my-predefined-username-password')
            }
            steps {
                sh 'echo "Service user is $SERVICE_CREDS_USR"'
                sh 'echo "Service password is $SERVICE_CREDS_PSW"'
                sh 'curl -u $SERVICE_CREDS https://myservice.example.com'
            }
        }
        stage('Example SSH Username with private key') {
            environment {
                SSH_CREDS = credentials('my-predefined-ssh-creds')
            }
            steps {
                sh 'echo "SSH private key is located at $SSH_CREDS"'
                sh 'echo "SSH user is $SSH_CREDS_USR"'
                sh 'echo "SSH passphrase is $SSH_CREDS_PSW"'
            }
        }
    }
}
```


#### `options`


`options` 指令允许在 Pipeline 本身中配置特定于 Pipeline 的选项。Pipeline 提供了许多此类选项，如 `buildDiscarder` 等，但他们也可能是由插件所提供，如 `timestamps` 等。


<table>
  <tr>
    <th>是否必需</th>
    <td>非必需</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>无</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在 <code>pipeline</code> 代码块内，或（附带条件地）在 <code>stage</code> 指令里</td>
  </tr>
</table>


**可用选项**

- `buildDiscarder`

为最近特定次数的流水线运行保留构建物与控制台输出。例如： `options { buildDiscarder(logRotator(numToKeepStr: '1')) }`。

- `checkoutToSubdirectory`

在工作区的子目录中执行自动源代码控制检出。例如： `options { checkoutToSubdirectory('foo') }`。

- `disableConcurrentBuilds`

禁止流水线的并发执行。这对于防止同时访问共用的资源等方面非常有用。例如：`options { disableConcurrentBuilds() }` 可在流水线已有构建正在执行时排队等待构建；`options { disableConcurrentBuilds(abortPrevious: true) }` 可中止正在运行的构建并启动新的构建。

- `disableResume`

如果控制器重新启动，则不允许流水线恢复。例如：`options { disableResume() }`。

- `newContainerPerStage`

与 `docker` 或 `dockerfile` 的顶级代理一起使用。指定后，每个阶段都将在同一节点上的新容器实例中运行，而不是所有阶段都在同一容器实例中运行。

- `overrideIndexTriggers`

允许覆盖分支索引触发器的默认处理方式，default treatment of branch indexing triggers。如果在多分支或组织标签中禁用了分支索引触发器，那么 `options { overrideIndexTriggers(true) }` 将仅对此任务启用分支索引触发器。否则，`options { overrideIndexTriggers(false) }` 则将仅对此任务禁用分支索引触发器。

- `preserveStashes`

保留已完成构建的存储，stashes from completed builds, 以便在阶段重启时使用。例如：`options { preserveStashes() }` 会保留最近完成构建中的存储，或 `options { preserveStashes(buildCount: 5) }` 则会保留最近完成的五个构建中的存储。

- `quietPeriod`

设置流水线的静默期（以秒为单位），会覆盖全局的默认值。例如： `options { quietPeriod(30) }`。

- `retry`

在失败时，会按所指定的次数重试整个流水线。例如：`options { retry(3) }`。

- `skipDefaultCheckout`

跳过 `agent` 指令中默认的从源代码控制系统中签出代码。例如： `options { skipDefaultCheckout() }`。

- `skipStagesAfterUnstable`

一旦构建状态变为 `UNSTABLE`，就跳过各阶段。例如： `options { skipStagesAfterUnstable() }`。

- `timeout`

设置流水线运行的超时时间，超时后 Jenkins 将终止流水线的运行。例如：`options { timeout(time: 1, unit: 'HOURS') }`。


*示例 8：全局超时，声明式流水线*

```groovy
pipeline {
    agent any
    options {
        timeout(time: 1, unit: 'HOURS') // 1
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

1. 指定一个小时的全局执行超时，超时后 Jenkins 将终止流水线运行。


- `timestamps`

在该次流水线运行所生成的全部控制台输出前加上行输出的时间。例如：`options { timestamps() }`。

- `parallelAlwaysFailFast`

对流水线中所有后续并行阶段设置 `failfast` 为 `true`。例如： `options { parallelsAlwaysFailFast() }`。

- `disableRestartFromStage`

完全禁用经典 Jenkins UI 和 Blue Ocean 中可见的 "Restart From Stage" 选项。例如：`options { disableRestartFromStage() }`。该选项不能在阶段内使用。

> 可用选项的完整列表正在等待 [help desk ticket 820](https://github.com/jenkins-infra/helpdesk/issues/820) 的完成。


**阶段选项**

**`stage` `options`**


`stage` 的 `options` 指令与流水线根处的 `options` 指令类似。不过，`stage` 级的 `options` 只能包含 `retry`、`timeout` 或 `timestamps` 等步骤，或与某个 `stage` 相关的声明式选项，如 `skipDefaultCheckout` 等。


在某个 `stage` 内，`options` 指令中的步骤会在进入 `agent` 或检查任何 `when` 条件之前被调用。


**可用的阶段选项**

**Available Stage Options**


{{#include ./syntax.md:725:727}}

- `timeout`

设置该阶段的超时时间，超时后 Jenkins 将终止该阶段。例如： `options { timeout(time: 1, unit: 'HOURS') }`。


*示例 9：阶段的超时，声明式流水线*

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            options {
                timeout(time: 1, unit: 'HOURS') // 1
            }
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

1. 为 `Example` 阶段指定一个小时的执行超时时间，之后 Jenkins 将中止该流水线运行。

- `retry`

在失败时，则重试该阶段指定的次数。例如：`options { retry(3) }`。

{{#include ./syntax.md:759:761}}


#### `parameters`

`parameters` 指令提供了在触发流水线时用户应提供的参数列表。这些用户指定的参数值可通过 `params` 对象提供给流水线步骤，具体用法请参阅 下面的 “参数，声明式管道” 示例。

每个参数都有一个 *名称, Name* 与 *值，Value*，具体取决于参数类型。在构建启动时，这些信息将作为环境变量输出，以便该构建配置的后续部分访问这些值。例如，对于 `bash` 和 `ksh` 等 POSIX 的 shell 中使用 `${PARAMETER_NAME}` 语法，在 PowerShell 中使用 `${Env:PARAMETER_NAME}` 语法，而在 Windows 的 `cmd.exe` 中则要使用 `%PARAMETER_NAME%` 语法。

<table>
  <tr>
    <th>是否必需</th>
    <td>非必需</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>无</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>仅一次，在 <code>pipeline</code> 代码块里</td>
  </tr>
</table>


**可用参数**


- `string`
- `text`
- `booleanParam`
- `choice`
- `password`
