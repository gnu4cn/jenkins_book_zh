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

字符串类型的参数，比如：`parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }`。

- `text`

文本参数，可包含多个行，比如：`parameters { text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '') }`。

- `booleanParam`

布尔值参数，比如：`parameters { booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') }`。

- `choice`

单选参数，比如：`parameters { choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '') }`。

- `password`

口令参数，比如： `parameters { password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password') }`。


*示例 10， 参数，声明式流水线*


```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')

        text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')

        booleanParam(name: 'TOGGLE', defaultValue: true, description: 'Toggle this value')

        choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')

        password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"

                echo "Biography: ${params.BIOGRAPHY}"

                echo "Toggle: ${params.TOGGLE}"

                echo "Choice: ${params.CHOICE}"

                echo "Password: ${params.PASSWORD}"
            }
        }
    }
}
```

> **注意**：可用参数的完整列表正在等待 [help desk ticket 820](https://github.com/jenkins-infra/jenkins.io/issues/5221) 的完成。



#### `triggers`

`triggers` 指令定义了流水线应以哪些自动方式被再度触发。对于以诸如 GitHub 或 BitBucket 集成的流水线，由于多半已经有了基于 webhooks 的集成，`triggers` 指令就没那么必要。当前可用的触发器有 `cron`、`pollSCM` 与 `upstream`。

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


- `cron`

接受 `cron` 样式的字符串，来定义出流水线应被重新触发的定期间隔，比如：`triggers { cron('TZ=Asia/Shanghai H(25-40) 8 * * *') }`。

- `pollSCM`

接受 `cron` 样式的字符串，来定义 Jenkins 应检查新的源代码更改的定期间隔。如果存在新的变更，则将重新触发流水线。比如： `triggers { pollSCM('H */4 * * 1-5') }`。

- `upstrem`

接受以逗号分隔的作业字符串以及一个阈值（threshold）。在字符串中的任何作业以最小阈值完成时，该流水线都将被重新触发。比如：`triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }`。


> **注意**：`pollSCM` 触发器只在 Jenkins 2.22 及以后版本中可用。


*示例 11，触发器，声明式流水线*


```groovy
// Declarative //
pipeline {
    agent any
    triggers {
        cron('H */4 * * 1-5')
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


#### Jenkins 的 `cron` 语法

Jenkins 的 `cron` 语法遵循了 [cron 实用工具（utility）](https://en.wikipedia.org/wiki/Cron) （有些许细微差别）的语法。具体来说，每个行都是由 5 个 TAB 或空格分隔开的字段组成：

<table>
  <tr>
    <th>分钟（MINUTE）</th>
    <th>小时（HOUR）</th>
    <th>几日（DOM）</th>
    <th>月份（MONTH）</th>
    <th>周几（DOW）</th>
  </tr>
  <tr>
    <td>小时里的分钟（0-59）</td>
    <td>一天中的小时（0-23）</td>
    <td>一个月中的天（1-31）</td>
    <td>第几月（1-12）</td>
    <td>一周中的周几（0-7），其中 0 和 7 都表示周日。</td>
  </tr>
</table>

要为某个字段指定多个值，可以使用以下的运算符。按照优先顺序，

- `*` 指定全部有效值；

- `M-N` 指明某个值的范围；

- `M-N/X` 或 `*/X` 按 `X` 的间隔对所指定的范围步进，或对整个有效范围步进；

- `A,B,...,Z` 枚举多个值。


为了令到定期调度任务在系统上产生均匀的负载，那么就要尽可能使用符号 `H`（表示 “哈希（hash）”）。例如，对十余个的日常作业使用 `0 0 * * *`，将导致午夜出现大幅峰值负载。相比之下，使用 `H H * * *` 仍会每天执行各个作业一次，却不是同时执行所有作业，这样就更好地利用有限的资源。

哈希符号可与范围一起使用。比如，`H H(0-7) * * *` 就表示 12:00 AM（午夜） 到 7:59 AM 之间的某个时间。咱们还可以在带或不带范围下，讲步长间隔与 `H` 一起使用。

请注意，对于几日（the day of month）字段，由于几日字段的长度可变，`*/3` 或 `H/3` 等短周期在大多数月末附近不会一致性地工作。例如，`*/3` 将在长月的第 1 天、第 4 天、......第 31 天运行，然后在下个月的下一天运行。哈希值会始终选择在 1-28 日范围内，因此 `H/3` 将在月底的运行之间产生出 3 到 6 天的间隙。较长的周期也会有不一致的长度，但其效果可能相对不太明显。

空行及以 `#` 开头的行将作为注释而被忽略。

此外，还支持 `@yearly`、`@annually`、`@monthly`、`@weekly`、`@daily`、`@midnight` 和 `@hourly` 作为方便的别名。他们会使用哈希系统进行自动的负载均衡。例如，`@hourly` 与 `H * * * *` 相同，可以表示一小时内的任何时间。`@midnight` 实际上是指 12:00 AM 到 2:59 AM 之间的某个时间。

<table>
<tr><td>
每 15 分钟（也许是在 :07、:22、:37、:52）。
</td></tr>
<tr><td>
<code>triggers{ cron('H/15 * * * *') }</code>
</td></tr>
<tr><td>
每个小时的前半段每十分钟一次（三次，可能在 :04, :14, :24）。
</td></tr>
<tr><td>
<code>triggers{ cron('H(0-29)/10 * * * *') }</code>
</td></tr>
<tr><td>
每个工作日从上午 9:45 开始到下午 3:45 结束，每两小时在 45 分时一次。
</td></tr>
<tr><td>
<code>triggers{ cron('45 9-16/2 * * * 1-5') }</code>
</td></tr>
<tr><td>
每个工作日上午 9 点到下午 5 点之间的每两小时时间槽中一次（可能是上午 10:38、中午 12:38、下午 2:38、下午 4:38）。
</td></tr>
<tr><td>
<code>triggers{ cron('H H(9-16)/2 * * * 1-5') }</code>
</td></tr>
<tr><td>
除 12 月外，每个月的 1 日和 15 日每天一次。
</td></tr>
<tr><td>
<code>triggers{ cron('H H 1,15 1-11 *') }</code>
</td></tr>
</table>

*表 1， Jenkins 的 `cron` 语法示例*


#### `stage`

`stage` 指令位于 `stages` 小节里，应包含一个 `steps` 小节、可选的 `agent` 小节或其他特定于阶段的指令。实际上，流水线完成的所有具体工作，都将包含在一个或多个 `stage` 指令中。

<table>
  <tr>
    <th>是否必需</th>
    <td>至少一个</td>
  </tr>
  <tr>
    <th>参数</th>
    <td>一个强制性参数，该阶段的名称字符串。</td>
  </tr>
  <tr>
    <th>在何处允许使用</th>
    <td>在 <code>stages</code> 小节内。</td>
  </tr>
</table>

*示例 12，阶段，声明式流水线*


```groovy
// Declarative //
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

#### `tools`

定义自动安装并放在 PATH 中工具的小节。如果指定了 `agent none`，则这会被忽略。


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
    <td>在 <code>pipeline</code> 或 <code>stage</code> 代码块内。</td>
  </tr>
</table>


**所支持的工具**

- `maven`

- `jdk`

- `gradle`


*示例 13，工具，声明式流水线*

```groovy
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1' // 1
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

1. 工具名字务必要在 Jenkins 的 **Manage Jenkins -> Tools** 下预先配置好。


#### `input`

`stage` 上的 `input` 指令允许咱们使用 [输入步骤, `input` step](https://www.jenkins.io/doc/pipeline/steps/pipeline-input-step/#input-wait-for-interactive-input) 提示输入。在应用任何 `options` 后、进入该 `stage` 的 `agent` 代码块或计算该 `stage` 的 `when` 条件之前，该 `stage` 将暂停。如果 `input` 获得批准，那么该 `stage` 将继续进行。作为 `input` 提交一部分所提供的任何参数，都将在这个 `stage` 其余部分的环境中可用。


**配置选项**

- `message`

必需的。当用户提交 `input` 时，该信息将呈现给用户。

- `id`

此 `input` 的可选标识符。其默认值基于 `stage` 的名字。

- `ok`

该 `input` 表单上可选的 “Ok” 按钮文本。

- `submitter`

一个可选的允许提交此 `input` 的用户或外部组名称的逗号分隔列表。默认允许任何用户。

- `submitterParameter`

在提交者名称存在时，与其一起设置的可选环境变量名称。

- `parameters`

提示提交者要提供的一个可选参数列表。请参阅 [`parameters`](#parameters) 以获取更多信息。


*示例 14，输入步骤，声明式流水线*

```groovy
pipeline {
    agent any

    stages {
        stage('Example') {
            input {
                message "Should we continue?"
                ok "Yes, we should."
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                }
            }
            steps {
                echo "Hello, ${PERSON}, nice to meet you."
            }
        }
    }
}
```

#### `when`

`when` 指令允许流水线根据所给的条件，确定出是否应执行该阶段。 `when` 指令必须至少包含一个条件。如果 `when` 指令包含多个条件，则所有子条件都必须返回 `true` 才能执行该阶段。这与子条件嵌套在 `allOf` 条件中相同（请参阅下面的示例）。在使用 `anyOf` 条件时，就请注意，一旦找到首个 `true` 条件，这个 `anyOf` 条件就会跳过其余条件测试。


可使用嵌套条件 `not`、`allOf` 或 `anyOf`，构建出更为复杂的条件结构。嵌套条件可以嵌套到任意深度。


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
    <td>在 <code>stage</code> 指令内。</td>
  </tr>
</table>


**内建条件**

- `branch`

当正在构建的分支与给定分支模式（[ANT 样式全局路径](https://stackoverflow.com/a/22636142)）匹配时执行该阶段，例如：`when {branch 'master' }`。请注意，这仅适用于多分支流水线。

可选参数 `comparator` 可以添加在属性之后，以指定出如何计算匹配的任何模式如何计算：

* 可用的 `comparator`:
    * `EQUALS` 用于简单的字符串比较；
    * `GLOB`（默认）用于 ANT 样式的全局路径（比如与 `changeset` 用到的全局路径相同）；
    * `REGEXP` 用于正则表达式匹配。

比如：`when { branch pattern: "release-\\d+", comparator: "REGEXP"}`。

- `buildingTag`

当构建为正在构建某个标签时执行该阶段。例如：`when {buildingTag() }`。

- `changelog`

在构建的源代码管理系统（SCM）变更日志，包含了给定正则表达式模式时执行该阶段，例如：`when {changelog '.*^\\[DEPENDENCY\\] .+$' }`。

- `changeset`

在构建的 SCM 变更集，包含一个或多个与给定模式匹配的文件时执行该阶段。示例：`when {changeset "**/*.js" }`。

可选参数 `comparator` 可以添加在属性之后，以指定出如何计算匹配任何的模式如何计算：

* 可用的 `comparator`:
    * `EQUALS` 用于简单的字符串比较；
    * `GLOB`（默认）用于 ANT 样式的全局路径（可使用 `caseSensitive` 参数关闭）；
    * `REGEXP` 用于正则表达式匹配。

比如：`when { changeset pattern: ".TEST\\.java", comparator: "REGEXP" }` 或 `when { changeset pattern: "*/*TEST.java", caseSensitive: true }`。

- `changeRequest`

在当前构建是针对某个“变更请求”（即，GitHub 与 Bitbucket 上的拉取请求，Pull Request、GitLab 上的合并请求，Merge Request、Gerrit 中的更改，Change，等）时，执行该阶段。当没有传递参数时，该阶段会在每个变更请求时运行，例如：`when {changeRequest() }`。

通过向变更请求添加带参数的过滤器属性，adding a filter attribute with parameter to the change request, 可以使阶段仅在匹配的变更请求上运行。可行的属性有 `id`、`target`、`branch`、`fork`、`url`、`title`、`author`、`authorDisplayName` 和 `authorEmail`。其中每个都对应了一个 `CHANGE_*` 环境变量，例如：`when {changeRequest target: 'master' }`。

{{#include ./syntax.md:1216:1221}}

示例：`when { changeRequest authorEmail: "[\\w_-.]+@example.com", comparator: 'REGEXP' }`。

- `environment`

当所定的环境变量被设置为给定值时，执行该阶段，例如：`when { environment name：'DEPLOY_TO'，value：'production' }`。

- `equals`

当期望值等于实际值时执行该阶段，例如：`when { equals expected: 2, actual: currentBuild.number }`。

- `expression`

当所指定的 Groovy 表达式计算结果为 `true` 时执行该阶段，例如：`when { expression { return params.DEBUG_BUILD } }`。

> **注意**：当从表达式返回字符串时，必须将他们转换为布尔值或返回 `null` 以计算为 `false`。简单地返回 `0` 或 `false` 仍将计算为 `true`。


- `tag`

当 `TAG_NAME` 变量与给定模式匹配时，则执行该阶段。例如：`when { tag "release-*" }`。而如果提供了空的模式，则当 `TAG_NAME` 变量存在时，该阶段将执行（这就与 `buildingTag()` 相同了）。

{{#include ./syntax.md:1216:1221}}

例如：`when { tag pattern: "release-\\d+", comparator: "REGEXP" }`。

- `not`

当嵌套的条件为 `false` 时执行该阶段。必须包含一个条件。例如：`when { not { branch 'master' } }`。

- `allOf`

当所有嵌套条件都为真时执行该阶段。必须至少包含一个条件。例如：`when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value：'production' } }`。

- `anyOf`

当至少一个嵌套条件为真时执行该阶段。必须至少包含一个条件。例如：`when { anyOf { branch 'master'; branch 'staging' } }`。

- `triggeredBy`

在当前构建是由给定参数触发的时，执行该阶段。例如：

* `triggeredBy` 参数类型：

    * `when { triggeredBy 'SCMTrigger' }`
    * `when { triggeredBy 'TimerTrigger' }`
    * `when { triggeredBy 'BuildUpstreamCause' }`
    * `when { triggeredBy cause: "UserIdCause", detail: "vlinde" }`

**于某个 `stage` 中在进入 `agent` 前计算 `when`**

默认情况下，如果定义了阶段的 `when` 条件，则将会在进入该阶段的 `agent` 后计算该 `when` 条件。但是，可以通过在 `when` 代码块中指定 `beforeAgent` 选项来修改这一点。如果 `beforeAgent` 被设置为 `true`，那么将首先计算 `when` 条件，并且仅当 `when` 条件得出为 `true` 时才会进入代理。

**在 `input` 指令前计算 `when`**

默认情况下，如果定义了阶段的 `when` 条件，则在该阶段的输入之前不会计算阶段的 `when` 条件。但是，可以通过在 `when` 代码块中指定 `beforeInput` 选项来更改这一点。如果 `beforeInput` 设置为 `true`，则将首先计算出 `when` 条件，并且仅当 `when` 条件评估为 `true` 时才会进入输入环节。

`beforeInput true` 优先于 `beforeAgent true`。

**在 `options` 指令前计算 `when`**

默认情况下，如果定义了阶段的 `options` 与 `when` 代码块，那么将在进入该阶段的 `options` 后计算该阶段的 `when` 条件。但是，可以通过在 `when` 代码块中指定 `beforeOptions` 选项来更改此行为。如果 `beforeOptions` 设置为 `true`，则将首先计算 `when` 条件，并且仅当 `when` 条件得出为 `true` 时才会进入 `options` 代码块。


`beforeOptions true` 优先于 `beforeInput true` 及 `beforeAgent true`。


*示例 15，单一条件，声明式流水线*


```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

*示例 16，多重条件，声明式流水线*


```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
                environment name: 'DEPLOY_TO', value: 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

*示例 17，嵌套的条件（与上个示例行为一致）*


```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                allOf {
                    branch 'production'
                    environment name: 'DEPLOY_TO', value: 'production'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

*示例 18，多重条件与嵌套条件*


```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

*示例 19，表达式条件与嵌套条件*

```groovy
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                expression { BRANCH_NAME ==~ /(production|staging)/ }
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```


*示例 20，`beforeAgent`*


```groovy
pipeline {
    agent none
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            agent {
                label "some-label"
            }
            when {
                beforeAgent true
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

*示例 21，`beforeInput`*


```groovy
pipeline {
    agent none
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                beforeInput true
                branch 'production'
            }
            input {
                message "Deploy to production?"
                id "simple-input"
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```


*示例 22，`beforeOptions`*


```groovy
pipeline {
    agent none
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                beforeOptions true
                branch 'testing'
            }
            options {
                lock label: 'testing-deploy-envs', quantity: 1, variable: 'deployEnv'
            }
            steps {
                echo "Deploying to ${deployEnv}"
            }
        }
    }
}
```

*示例 23，`triggeredBy`*


```groovy
pipeline {
    agent none
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                triggeredBy "TimerTrigger"
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

### 顺序式阶段

**Sequential Stages**

声明式流水线中的阶段，可能有一个其中包含要按先后顺序运行的嵌套阶段清单的 `stages` 小节。


> 阶段必须有且只有一个 `steps`、`stages`、`parallel` 或 `matrix` 之一。如果 `stage` 指令是嵌套在 `parallel` 或 `matrix` 代码块本身中，则在这个 `stage` 指令中嵌套 `parallel` 或 `matrix` 代码块是不可行的。但是，`parallel` 或 `matrix` 代码块中的 `stage` 指令可以使用 `stage` 的所有其他功能，包括 `agent`、`tools`、`when` 等等。


*示例 24，顺序式阶段，声明式流水线*


```groovy
pipeline {
    agent none
    stages {
        stage('Non-Sequential Stage') {
            agent {
                label 'for-non-sequential'
            }
            steps {
                echo "On Non-Sequential Stage"
            }
        }
        stage('Sequential') {
            agent {
                label 'for-sequential'
            }
            environment {
                FOR_SEQUENTIAL = "some-value"
            }
            stages {
                stage('In Sequential 1') {
                    steps {
                        echo "In Sequential 1"
                    }
                }
                stage('In Sequential 2') {
                    steps {
                        echo "In Sequential 2"
                    }
                }
                stage('Parallel In Sequential') {
                    parallel {
                        stage('In Parallel 1') {
                            steps {
                                echo "In Parallel 1"
                            }
                        }
                        stage('In Parallel 2') {
                            steps {
                                echo "In Parallel 2"
                            }
                        }
                    }
                }
            }
        }
    }
}
```


### 并行

**Parallel**


声明式管道中的阶段，可能有一个其中包含着要并行运行的嵌套阶段列表的 `parallel` 小节。

{{#include ./syntax.md:1569}}


此外，咱们可以通过将 `failFast true` 添加到包含 `parallel` 的 `stage` 代码块，从而强制咱们的 `parallel` 阶段在其中任何一个阶段失败时整个 `parallel` 都中止。添加 `failFast` 的另一个选项，是向流水线定义添加一个选项：`parallelsAlwaysFailFast()`。

*示例 25，并行阶段，声明式流水线*


```groovy
pipeline {
    agent any
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            failFast true
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
                stage('Branch C') {
                    agent {
                        label "for-branch-c"
                    }
                    stages {
                        stage('Nested 1') {
                            steps {
                                echo "In stage Nested 1 within Branch C"
                            }
                        }
                        stage('Nested 2') {
                            steps {
                                echo "In stage Nested 2 within Branch C"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

*示例 26，`parallelsAlwaysFailFast`*


```groovy
pipeline {
    agent any
    options {
        parallelsAlwaysFailFast()
    }
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
                stage('Branch C') {
                    agent {
                        label "for-branch-c"
                    }
                    stages {
                        stage('Nested 1') {
                            steps {
                                echo "In stage Nested 1 within Branch C"
                            }
                        }
                        stage('Nested 2') {
                            steps {
                                echo "In stage Nested 2 within Branch C"
                            }
                        }
                    }
                }
            }
        }
    }
}
```


### 矩阵

**Matrix**

声明式流水线中的阶段可能会有一个定义出要并行运行的名称-值组合多维矩阵的 `matix` 小节。我们将这些组合称为矩阵中的“单元”。矩阵中的每个单元均可包括一个或多个要使用该单元配置的顺序运行的阶段，stages in Declarative Pipeline may have a `matrix` section defining a multi-dimensional matrix of name-value combinations to be run in parallel. We'll refer these combinations as "cells" in a matrix. Each cell in a matrix can include one or more stages to be run sequentially using the configuration for that cell。

{{#include ./syntax.md:1569}}

此外，咱们可以通过将 `failFast true` 添加到包含着 `matrix` 的 `stage`，强制 `matrix` 的单元在其中任何一个失败时全部中止。添加 `failFast` 的另一选项，是向流水线定义添加一个选项：`parallelsAlwaysFailFast()`。

`matrix` 小节必须包括 `axes` 小节和 `stages` 小节。`axes` 小节定义出矩阵中每个 `axis` 的值。`stages` 小节定义出在各个单元中按顺序运行的 `stage` 列表。`matrix` 可能会有一个 `excludes` 小节，用于从矩阵中移除无效单元。`stage` 上可用的许多指令，包括 `agent`、`tools`、`when` 等等，也可以添加到 `matrix`，以控制各个单元的行为。

#### `axes`

`axes` 小节指定一或多个 `axis` 指令。各条 `axis` 有一个 `name` 及 `values` 清单构成。各个维度（轴向）的全部值，会与其他维度（轴向）结合，以产生出那些单元。


*示例 27，有 3 个单元的一维度（轴向）矩阵*

```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
    }
    // ...
}
```


*示例 28，有着 12 个单元的两维度（轴向）矩阵*

```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
    }
    // ...
}
```


*示例 29，有着 24 个单元的三维度（轴向）矩阵*


```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
        axis {
            name 'ARCHITECTURE'
            values '32-bit', '64-bit'
        }
    }
    // ...
}
```


#### `stages`

`stages` 小节指定了要在每个单元中顺序执行的一个或多个 `stage`。这个小节与任何其他的 [`stages` 小节](#顺序式阶段) 都相同。


*示例 30，有着 3 个单元，每个单元会运行三个阶段 -- “构建”、“测试” 与 “部署” 的一维度矩阵*


```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
    }
    stages {
        stage('build') {
            // ...
        }
        stage('test') {
            // ...
        }
        stage('deploy') {
            // ...
        }
    }
}
```

*示例 31，有着 12 单元（三乘四）的二维矩阵*


```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
    }
    stages {
        stage('build-and-test') {
            // ...
        }
    }
}
```


#### `excludes` （可选小节）


可选的 `excludes` 小节允许脚本编写者指定一个或多个的 `exclude` 过滤器表达式，这些表达式从扩展的矩阵单元集中，选择一些要排除的单元格（也称为 “稀疏”，sparsening）。过滤器是使用一个或多个排除 `axis` 指令，每个排除轴指令都具有 `name` 和 `values` 列表，的基本指令结构构造出的。

`exclude` 代码块内的 `axis` 指令，会生成一套组合（类似于生成矩阵单元）。与 `exclude` 组合中的所有值匹配的矩阵单元，将从矩阵中移除。如果提供了多个 `exclude` 指令，则会分别计算每个排除指令以移除单元格。


当处理长的排除值列表时，排除 `axis` 指令可以使用 `notValues` 而非 `values`。这将排除与传递给 `notValues` 的值之一 **不** 匹配的单元格。


*示例 32，有着 24 个单元，排除了 `32-bit, mac`（4 个单元被排除）的三维度矩阵*

```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
        axis {
            name 'ARCHITECTURE'
            values '32-bit', '64-bit'
        }
    }
    excludes {
        exclude {
            axis {
                name 'PLATFORM'
                values 'mac'
            }
            axis {
                name 'ARCHITECTURE'
                values '32-bit'
            }
        }
    }
    // ...
}
```


排除 `linux`、`safari` 组合，并排除任何 **不是** 带有 `edge` 浏览器的 `windows` 平台。

*示例 33，带有 24 个单元，排除 `32-bit, mac` 与无效浏览器组合（9 个单元被排除）的三维度矩阵*


```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'BROWSER'
            values 'chrome', 'edge', 'firefox', 'safari'
        }
        axis {
            name 'ARCHITECTURE'
            values '32-bit', '64-bit'
        }
    }
    excludes {
        exclude {
            // 4 cells
            axis {
                name 'PLATFORM'
                values 'mac'
            }
            axis {
                name 'ARCHITECTURE'
                values '32-bit'
            }
        }
        exclude {
            // 2 cells
            axis {
                name 'PLATFORM'
                values 'linux'
            }
            axis {
                name 'BROWSER'
                values 'safari'
            }
        }
        exclude {
            // 3 more cells and '32-bit, mac' (already excluded)
            axis {
                name 'PLATFORM'
                notValues 'windows'
            }
            axis {
                name 'BROWSER'
                values 'edge'
            }
        }
    }
    // ...
}
```

#### 矩阵的单元级指令（可选的）

矩阵特性实现了通过在 `matrix` 本身下添加阶段级指令，令到用户有效地配置每个单元的整体环境。这些指令的行为与在阶段上的行为相同，但他们也可以接受矩阵为每个单元提供的值。

`axis` 和 `exclude` 指令定义了组成矩阵的静态单元集。该套组合是在流水线运行开始之前生成的。而另一方面，“各单元” 的指令，则是在运行时计算出的。

这些指令包括：

- [`agent`](#agent)

- [`environment`](#environment)

- [`input`](#input)

- [`options`](#options)

- [`post`](#post)

- [`tools`](#tools)

- [`when`](#when)


*示例 34，完整矩阵示例，声明式流水线*


```groovy
pipeline {
    parameters {
        choice(name: 'PLATFORM_FILTER', choices: ['all', 'linux', 'windows', 'mac'], description: 'Run on specific platform')
    }
    agent none
    stages {
        stage('BuildAndTest') {
            matrix {
                agent {
                    label "${PLATFORM}-agent"
                }
                when { anyOf {
                    expression { params.PLATFORM_FILTER == 'all' }
                    expression { params.PLATFORM_FILTER == env.PLATFORM }
                } }
                axes {
                    axis {
                        name 'PLATFORM'
                        values 'linux', 'windows', 'mac'
                    }
                    axis {
                        name 'BROWSER'
                        values 'firefox', 'chrome', 'safari', 'edge'
                    }
                }
                excludes {
                    exclude {
                        axis {
                            name 'PLATFORM'
                            values 'linux'
                        }
                        axis {
                            name 'BROWSER'
                            values 'safari'
                        }
                    }
                    exclude {
                        axis {
                            name 'PLATFORM'
                            notValues 'windows'
                        }
                        axis {
                            name 'BROWSER'
                            values 'edge'
                        }
                    }
                }
                stages {
                    stage('Build') {
                        steps {
                            echo "Do Build for ${PLATFORM} - ${BROWSER}"
                        }
                    }
                    stage('Test') {
                        steps {
                            echo "Do Test for ${PLATFORM} - ${BROWSER}"
                        }
                    }
                }
            }
        }
    }
}
```


### 步骤

**Steps**

声明式流水线可以使用 [流水线步骤参考，Pipeline Steps reference](https://www.jenkins.io/doc/pipeline/steps/) 中，所记录的所有可用步骤，其中包含了完整的步骤列表，此外还附带了下面列出的仅在声明式流水线中支持的步骤。


#### `script`

`script` 步骤取 [脚本化流水线，Scripted Pipeline](#脚本化流水线) 的一个代码块，并在声明式流水线中执行。对于大多数用例，声明式流水线中的 `script` 步骤应是不必要的，但他可以提供有用的 “逃生舱口，escape hatch”。规模和/或复杂性较大的 `script` 代码块块，应移至 [共享库](./shared-libraries.md) 中。


*示例 35，声明式流水线中的脚本代码块*

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'

                script {
                    def browsers = ['chrome', 'firefox']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                }
            }
        }
    }
}
```

## 脚本化流水线

**Scripted Pipeline**

脚本化流水线，与 [声明式流水线](#声明式流水线) 一样，是构建在底层所采用的流水线子系统之上的。与声明式不同，脚本化流水线实际上是以 [Groovy](http://groovy-lang.org/syntax.html) 构建的通用 DSL <sup>[1]</sup>。 Groovy 语言提供的大部分功能，都可供脚本流水线用户使用，这意味着他可以是一种非常具有表现力和灵活的工具，人们可以用他来编写持续交付的流水线。


### 流程控制

**Flow Control**


脚本化流水线从 `Jenkinsfile` 的顶部向下串行执行，就像 Groovy 或其他语言中的大多数传统脚本一样。因此，提供流程控制取决于 Groovy 表达式，譬如 `if/else` 条件，例如：

*示例 36，条件语句 `if`，脚本化流水线*

```groovy
node {
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
```

管理脚本化流水线流程控制的另一种方法，是使用 Groovy 的异常处理支持。当 [步骤-脚本化流水线](#步骤) 由于某种原因失败时，他们会抛出异常。处理错误时的行为必须使用 Groovy 中的 `try/catch/finally` 块，例如：


*示例 37，Try-Catch 代码块，脚本化流水线*

```groovy
node {
    stage('Example') {
        try {
            sh 'exit 1'
        }
        catch (exc) {
            echo 'Something failed, I should sound the klaxons!'
            throw
        }
}
```

### 步骤

**Steps**

正如本章开头所讨论的，流水线最基本的部分是“步骤”。从根本上讲，步骤告诉 Jenkins 要做什么，并作为声明式和脚本化流水线语法的基本构建代码块。

脚本化流水线 **未** 引入任何特定于其语法的步骤；[流水线步骤参考](https://www.jenkins.io/doc/pipeline/steps) 包含流水线及插件所提供的步骤完整列表。



### 与普通 Groovy 的区别


为了提供 *持久性，durability*，即运行中的流水线在 Jenkins [控制器](../glossary.md#控制器) 重启后仍能存活，脚本化流水线必须将数据序列化并返回控制器。由于这一设计要求，某些 Groovy 习惯做法（如 `collection.each { item → /* perform operation */ }`）并不完全受支持。有关详细信息，请参阅 [JENKINS-27421](https://issues.jenkins.io/browse/JENKINS-27421) 和 [JENKINS-26481](https://issues.jenkins.io/browse/JENKINS-26481)。


## 语法比较

[![脚本化与声明式流水线之间的区别](https://img.youtube.com/vi/GJBlskiaRrI/0.jpg)](https://www.youtube.com/watch?v=GJBlskiaRrI)


这个视频分享了脚本式流水线和声明式流水线语法之间的一些差异。


在创建 Jenkins 流水线之初，Groovy 被选为基础。长期以来，Jenkins 都附带了一个内嵌的 Groovy 引擎，为管理员和用户提供高级脚本功能。此外，Jenkins 流水线的实现人员，发现 Groovy 是一个坚实的基础，可以在此基础上构建现在被称为 "脚本化流水线 "的 DSL。<sup>[1]</sup>

作为一个功能齐全的编程环境，脚本化流水线为 Jenkins 用户提供了极大的灵活性和可扩展性。Groovy 的学习曲线，通常并不适合特定团队的所有成员，因此，声明式流水线应运而生，为编写 Jenkins 流水线提供了一种更简单、更有主见的语法。

从根本上说，两者都属于同样的流水线子系统。他们都是 “流水线即代码，Pipeline as code” 的持久化实现，durable implementation。他们都能使用流水线内置的，或插件所提供的步骤。两者都能利用上 [共享库](./shared-libraries.md)。

然而，他们在语法和灵活性方面有着差异。声明式以更严格和预定义的结构，来限制用户可用的内容，使其成为更简单的持续交付流水线的理想选择。脚本化的限制很少，因为对结构和语法的唯一限制，往往是由 Groovy 本身定义的，而不是任何特定于流水线的系统，这使他成为强大用户和有更复杂需求的用户的理想选择。顾名思义，声明式流水线鼓励使用声明式编程模型。<sup>[2]</sup>而脚本化流水线，则遵循的是命令式编程模式。<sup>[3]</sup>


1. [领域专用语言, domain-specific language, DSL](https://en.wikipedia.org/wiki/Domain-specific_language)

2. [声明式编程模型，declarative programming](https://en.wikipedia.org/wiki/Declarative_programming)

3. [命令式编程模型, imperative programming](https://en.wikipedia.org/wiki/Imperative_programming)
