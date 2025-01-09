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

下面是一个使用 `sh`（shell）的声明式管道中的例子，同时有着 `returnStatus` 和 `returnStdout`。

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any // 1
    environment {
        // Using returnStdout
        CC = """${sh(
                returnStdout: true,
                script: 'echo "clang"'
            )}""" // 2

        // Using returnStatus
        EXIT_STATUS = """${sh(
                returnStatus: true,
                script: 'exit 1'
            )}"""
    }
    stages {
        stage('Example') {
            environment {
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

1. 必须在流水线顶层设置一个 `agent`。如果代理被设置为 `agent none`，这将失败；

2. 使用 `returnStdout` 时，尾随空格将附加到返回的字符串。请使用 `.trim()` 删除它。


### 处理凭据

[在 Jenkins 中配置的凭证](../usage/using_credentials.md#配置凭据) 可以在流水线中处理，以便立即使用。请在 [使用凭据](../usage/using_credentials.md) 页面阅读更多关于在 Jenkins 中使用凭证的信息。

*Jenkins 中处理凭据的正确方式*

[![Jenkins 流水线中处理凭据的正确方式](https://img.youtube.com/vi/yfjtMIDgmfs/0.jpg)](https://www.youtube.com/watch?v=yfjtMIDgmfs)

视频总结：由于凭据涉及到机密信息，所以为了更好的安全性，应使用单引号的方式使用凭据，而不是使用 Groovy 的双引号插值方式使用凭据。


#### 对于秘文、用户名与口令及秘密文件

**For secret text, usernames and passwords, and secret files**

Jenkins 的声明式 Pipeline 语法有着 `credentials()` 辅助方法（在 [`environment` 指令](./syntax.md#envrionment-指令) 中使用），他支持秘密文本、用户名和密码，以及秘密文件等凭据。如果咱们想处理其他类型的凭据，请参考 [对于其他凭据类型](#对于其他凭据类型) 小节（见下文）。


**秘密文本**

下面的流水线代码展示了一个如何使用环境变量创建秘密文本凭据的流水线示例。

在此示例中，两个秘密文本凭证被赋值给了单独的环境变量以访问 Amazon Web Services (AWS)。这些凭据将在 Jenkins 中以其各自的凭据 ID `jenkins-aws-secret-key-id` 和 `jenkins-aws-secret-access-key` 而被配置。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent {
        // Define agent details here
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }

    stages {
        stage('Example stage 1') {
            steps {
                // 1
            }
        }

        stage('Example stage 2') {
            steps {
                // 2
            }
        }
    }
}
```

1. 咱们可以在本阶段的步骤中使用语法 `$AWS_ACCESS_KEY_ID` 和 `$AWS_SECRET_ACCESS_KEY`，引用这两个凭据环境变量（定义在此 Pipeline 的 `environment` 指令中）。例如，在这里咱们可以使用分配给这些凭据变量的秘密文本凭据来向 AWS 进行身份验证；

为了维护这些凭据的安全性和匿名性，如果作业在 Pipeline 内显示这些凭证变量的值（例如，`echo $AWS_SECRET_ACCESS_KEY`），Jenkins 只会返回值 `"****"`，以减少秘密信息被披露到控制台输出和任何日志的风险。凭据 ID 本身的任何敏感信息（如用户名）也会在流水线运行的输出中作为 `"****"` 返回。

这只是降低了 **意外暴露** 的风险。他并不能防止恶意用户通过其他方式获取凭据值。使用凭据的 Pipeline 也可以披露这些凭据。不要允许不受信任的流水线作业使用受信任的凭证。

2. 在这个流水线示例中，分配给两个 `AWS_...` 环境变量的凭据对整个 Pipeline 来说是全局范围的，所以这些凭据变量也可以在这个阶段的步骤中使用。但是，如果这个 Pipeline 中的 `environment` 指令被移到某个特定阶段（就像下面的用户名和密码流水线示例中的情况），那么这些 `AWS_...` 环境变量将只在该阶段的步骤中起作用。

> 在 Jenkins 凭据中存储静态 AWS 密钥不是很安全。如果咱们可以在 AWS 中运行 Jenkins 本身（至少是 Jenkins 代理），最好是为某台 [计算机](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html) 或 [EKS 服务账户](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) 使用 IAM 角色。也可以使用 [网络身份联盟](https://github.com/jenkinsci/oidc-provider-plugin#accessing-aws)。


**用户名与口令**

下面的 Pipeline 代码片断显示了一个为获得用户名与密码凭据，而如何使用环境变量创建 Pipeline 的示例。

在这个示例中，用户名和密码凭据被赋值给环境变量，以便在咱们组织的共同账户或团队中访问 Bitbucket 仓库；这些凭证在 Jenkins 中应以凭证 ID `jenkins-bitbucket-comm-creds` 被配置。

在 `environment` 指令中设置凭据环境变量时：


```groovy
environment {
    BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
}
```

这实际上设置了以下三个环境变量：

- `BITBUCKET_COMMON_CREDS` -- 包含用冒号分开的用户名和密码，格式为 `username:password`；

- `BITBUCKET_COMMON_CREDS_USR` -- 仅包含用户名部分的一个额外变量；

- `BITBUCKET_COMMON_CREDS_PSW` -- 仅包含密码部分的一个额外变量。

> 按照惯例，环境变量的名称通常用大写字母指定，各个单词之间用下划线分开。然而，咱们可以使用小写字母指定任何合法的变量名称。请记住，由 `credentials()` 方法（上文）创建的额外环境变量将总是会附加 `_USR` 和 `_PSW`（即，以下划线跟三个大写字母的格式）。

下面的代码片断显示了该 Pipeline 示例的全部内容：

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent {
        // Define agent details here
    }

    stages {
        stage('Example stage 1') {
            environment {
                BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
            }
            steps {
                // 1
            }
        }

        stage('Example stage 2') {
            steps {
                // 2
            }
        }
    }
}
```

以下凭据环境变量（在此 Pipeline 的 `environment` 指令中定义）在这个阶段的步骤中可用，可使用下面的语法来引用：

- `$BITBUCKET_COMMON_CREDS`

- `$BITBUCKET_COMMON_CREDS_USR`

- `$BITBUCKET_COMMON_CREDS_PSW`

1. 例如，在这里咱们可以使用分配给这些凭据变量的用户名和密码向 Bitbucket 进行身份验证。

为了维护这些凭据的安全性和匿名性，如果作业从管道中显示这些凭据变量的值，则上面秘密文本示例中所描述的同样行为也适用于这些用户名和密码凭据变量类型。

{{#include ./jenkinsfile.md:475}}

2. 在这个流水线示例中，分配给三个 `BITBUCKET_COMMON_CREDS...` 环境变量的凭据只适用于 `Example stage 1`，所以这些凭据变量不能用于这个 `Example stage 2` 的步骤中。然而，如果这个 Pipeline 中的 `environment` 指令被立即移到 `pipeline` 代码块中（就像上面的秘密文本流水线示例那样），那么这些 `BITBUCKET_COMMON_CREDS...` 环境变量就会具有全局作用域，可以在任何阶段的步骤中使用。


**秘密文件**

秘密文件是存储在文件中并上传到 Jenkins 的凭据。秘密文件用于以下的凭据：

- 直接输入 Jenkins 中太不方便，和/或

- 是二进制格式的，比如 GPG 文件。

在下面这个示例中，我们使用一个 Kubernetes 配置文件，该文件已被配置为一个名为 `my-kubeconfig` 的秘密文件凭据。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent {
        // Define agent details here
    }

    environment {
        // The MY_KUBECONFIG environment variable will be assigned
        // the value of a temporary file.  For example:
        //   /home/user/.jenkins/workspace/cred_test@tmp/secretFiles/546a5cf3-9b56-4165-a0fd-19e2afe6b31f/kubeconfig.txt
        MY_KUBECONFIG = credentials('my-kubeconfig')
    }

    stages {
        stage('Example stage 1') {
            steps {
                sh("kubectl --kubeconfig $MY_KUBECONFIG get pods")
            }
        }
    }
}
```

#### 对于其他凭据类型

**For other credentials types**

如果咱们需要在流水线中设置除秘密文本、用户名和密码或秘密文件（上文）以外的任何凭据 -- 即 SSH 密钥或证书等，那么请使用 Jenkins 的 **代码片段生成器，Snippet Generator** 功能，咱们可以通过 Jenkins 经典用户界面访问他。

要访问咱们流水线项目/条目的代码片段生成器：

1. 从 Jenkins 主页（即 Jenkins 经典用户界面的仪表板），点击咱们的流水线项目/条目的名称；

2. 在左边，点击 **流水线语法**，确保 **片段生成器** 的链接在左上方以粗体显示；(如果没有，请点击其链接）

3. 在 **示例步骤** 字段，选择 **withCredentials： Bind credentials to variables**；

4. 在 **绑定** 项下，点击 **新增** 并从下拉菜单中选择：

    + **SSH User Private Key** - 处理 [SSH 公/私钥对凭据](http://www.snailbook.com/protocols.html)，咱们可以从中指定：
        - **Key 文件变量** - 将被绑定到这些凭据的环境变量名字。Jenkins 实际上会给这个临时变量分配在 SSH 公/私钥对认证过程中所需私钥文件的安全位置；
        - **密码变量，Passphrase Variable**（可选项） - 将绑定到与 SSH 公/私钥对相关 [密码，passphrase](https://tools.ietf.org/html/rfc4251#section-9.4.4) 的环境变量名字；
        - **用户名变量**（可选项） - 将绑定到与 SSH 公/私钥对相关用户名的环境变量名字；
        - **凭据** - 选择存储在 Jenkins 中的 SSH 公/私钥凭据。这个字段的值为凭据的 ID，Jenkins 将其写到生成的代码片段中。


    SSH 用户私钥凭据代码片段示例：

    ```groovy
    withCredentials([sshUserPrivateKey(credentialsId: 'for-github-op', keyFileVariable: 'GITHUB_CRED')]) {
        // some block
    }
    ```

    + **证书，Certificate** - 处理 [PKCS#12 证书](https://tools.ietf.org/html/rfc7292)，咱们可以从中指定：
        - **密钥库变量，Keystore Variable** - 将绑定到这些凭据的环境变量名字。Jenkins 实际上将把证书身份验证过程中所需的证书密钥库的安全位置赋值给这个临时变量；
        - **密码变量，Password Variable** (可选项) - 将绑定到与证书关联的密码的环境变量名字；
        - **别名变量，Alias Variable** （可选项） - 将绑定到与证书关联的唯一别名的环境变量名字；
        {{#include ./jenkinsfile.md:607}}


5. 点击 **生成流水线脚本**，Jenkins 会为咱们所指定的凭据生成一个 `withCredentials( ... ) { ... }` 的流水线步骤片段，然后咱们便可以将其复制并粘贴到咱们的声明式或脚本化流水线代码中。


    **注意**：

    - **凭据** 字段（上文）会显示在 Jenkins 中配置的凭据名称。然而，在点击 **生成流水线脚本** 后，这些值将被转换为凭据 ID；
    - 要在一个 `withCredentials(...) { ...}` 流水线步骤中结合多个凭据，请参阅 在一个步骤中结合凭证（下文）。


下面的代码片段显示了一个完整的流水线示例，他实现了上面的 **SSH 用户私钥** 与 **证书** Pipeline 代码片段：

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent {
        // define agent details
    }

    stages {
        stage('Example stage 1') {
            steps {
                withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-ssh-key-for-abc', \
                                                             keyFileVariable: 'SSH_KEY_FOR_ABC')]) {
                  // 1
                }

                withCredentials(bindings: [certificate(credentialsId: 'jenkins-certificate-for-xyz', \
                                                       keystoreVariable: 'CERTIFICATE_FOR_XYZ', \
                                                       passwordVariable: 'XYZ-CERTIFICATE-PASSWORD')]) {
                  // 2
                }
            }
        }

        stage('Example stage 2') {
            steps {
                // 3
            }
        }
    }
}
```

1. 在这个步骤中，咱们可以用语法 `$SSH_KEY_FOR_ABC` 来引用那个凭据环境变量。例如，在这里咱们可以用其配置的 SSH 公钥/私钥对凭据来对 ABC 应用程序进行身份验证，其 **SSH 用户私钥文件** 被赋值给 `$SSH_KEY_FOR_ABC`；

2. 在这个步骤中，咱们可以用 `$CERTIFICATE_FOR_XYZ` 和 `$XYZ-CERTIFICATE-PASSWORD` 语法引用那个凭据环境变量。例如，在这里咱们可以用其配置的证书凭据来对 XYZ 应用程序进行身份验证，其证书的 keystore 文件和密码分别被赋值给了变量 `$CERTIFICATE_FOR_XYZ` 和 `$XYZ-CERTIFICATE-PASSWORD`；

3. 在这个流水线示例中，赋值给 `$SSH_KEY_FOR_ABC`、`$CERTIFICATE_FOR_XYZ` 和 `$XYZ-CERTIFICATE-PASSWORD` 环境变量的凭据只在其各自的 `withCredentials( ... ) { ... }` 步骤中起作用，所以这些凭据变量不能用于这个 `Example stage 2` 的步骤中。

为了维护这些凭据的安全性和匿名性，如果咱们试图从这些 `withCredentials( ... ) { ... }` 步骤中取得这些凭据变量的值，秘密文本示例（上文）中描述的行为也适用于这些 SSH 公/私钥对凭据和证书变量类型。

{{#include ./jenkinsfile.md:475}}

> - 当使用 **代码片段生成器** 中 **示例步骤** 字段的 **withCredentials: Bind credentials to variables** 选项时，只有咱们当前流水线项目/条目可以访问的凭据可以从任何的 **凭据，Credentials** 字段清单中选择。虽然咱们可以为咱们的流水线手动写一个 `withCredentials( ... ) { ... }` 步骤（就像上面的示例），但建议使用代码片段生成器，以避免指定超出此流水线项目/条目范围的凭据，这会在运行时使该步骤失败；
>
> - 咱们也可以使用 **代码片段生成器** 生成处理秘密文本、用户名和密码以及秘密文件的 `withCredentials( ... ) { ... }` 步骤。但是，如果咱们只需处理这些类型的凭据，建议使用 [上面](#对于秘文用户名与口令及秘密文件) 小节中描述的相关程序，以提高流水线代码的可读性；
>
> - 在上面的 Groovy 中使用了 **单引号** 而不是 **双引号** 来定义脚本（ `sh` 步骤的隐含参数）。单引号将导致秘密被 shell 扩展为环境变量。双引号可能不太安全，因为秘密是由 Groovy 插入的，而因此典型的操作系统进程列表会意外地泄露他。

```groovy
node {
  withCredentials([string(credentialsId: 'mytoken', variable: 'TOKEN')]) {
    sh /* WRONG! */ """
      set +x
      curl -H 'Token: $TOKEN' https://some.api/
    """
    sh /* CORRECT */ '''
      set +x
      curl -H 'Token: $TOKEN' https://some.api/
    '''
  }
}
```

**在一个步骤中结合多个凭据**

**Combining credentials in one step**


使用 **代码片段生成器，Snippet Generator**，咱们可以在一个 `withCredentials( ...) { ...}` 步骤中令到多个凭据可用，具体做法如下：

{{#include ./jenkinsfile.md:595:599}}

4. 点击 **绑定，Bindings** 下的 **新增**；

5. 从下拉列表中选择要添加到 `withCredentials( ...) { ...}` 步骤的凭据类型；

6. 指定凭据 **绑定，Bindings** 的细节。请在其他凭据类型下的程序中（上文）阅读更多关于这些的内容；

7. 对每一个（一组）要添加到 `withCredentials( ...) { ...}` 步骤中的凭据重复点击 **新增**（上文）；

8. 点击 **生成流水线脚本**，生成最终的 `withCredentials( ...) { ... }` 步骤代码片段。


### 字符串插值

**String interpolation**

Jenkins Pipeline 使用与 [Groovy](http://groovy-lang.org/) 相同的规则进行字符串插值。Groovy 的字符串插值支持可能会让许多刚接触这门语言的人感到困惑。虽然 Groovy 支持用单引号或双引号声明字符串，例如：

```groovy
def singlyQuoted = 'Hello'
def doublyQuoted = "World"
```

只有后一个字符串支持基于美元符号 (`$`) 的字符串插值，例如：

```groovy
def username = 'Jenkins'
echo 'Hello Mr. ${username}'
echo "I said, Hello Mr. ${username}"
```

将得到：

```console
Hello Mr. ${username}
I said, Hello Mr. Jenkins
```

了解如何使用字符串插值对于使用 Pipeline 的一些更高级的功能至关重要。


#### 敏感环境变量的插值

> **重要提示**：Groovy 字符串插值 **永远不** 应用于凭据。

Groovy 字符串插值可能会泄露敏感的环境变量（即凭据，参见：[处理凭据](#处理凭据)）。这是因为敏感环境变量将在 Groovy 执行过程中被插值，the sensitive environment variable will be interpolated during Groovy evaluation，而环境变量的值可能比预期的更早被提供，导致敏感数据在各种情况下泄露。

例如，设想下面的敏感的环境变量传递给 `sh` 步骤。

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any

    environment {
        EXAMPLE_CREDS = credentials('example-credentials-id')
    }

    stages {
        stage('Example') {
            steps {
                /* WRONG! */
                sh("curl -u ${EXAMPLE_CREDS_USR}:${EXAMPLE_CREDS_PSW} https://example.com/")
            }
        }
    }
}
```

如果 Groovy 执行插值，敏感值就将被直接注入 `sh` 步骤的参数中，除开其他问题，这意味着字面值将在操作系统进程列表中作为代理上 `sh` 进程的参数而可见。在引用这些敏感环境变量时，使用单引号而不是双引号可以防止这种类型的泄漏。

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any
    environment {
        EXAMPLE_CREDS = credentials('example-credentials-id')
    }
    stages {
        stage('Example') {
            steps {
                /* CORRECT */
                sh('curl -u $EXAMPLE_CREDS_USR:$EXAMPLE_CREDS_PSW https://example.com/')
            }
        }
    }
}
```

#### 通过插值注入

**Injection via interpolation**


> **重要提示**：Groovy 字符串插值可以通过特殊字符将恶意命令注入命令解释器。

另一个要注意的问题，对用户控制的变量使用 Groovy 字符串插值，并将其参数传递给命令解释器的步骤，如 `sh`、`bat`、`powershell` 或 `pwsh` 步骤，会导致类似于 SQL 注入的问题。当用户控制的变量（一般是环境变量，通常是传递给构建的参数）包含特殊字符（如 `/\ $ & % ^ > < | ; `）被传递到使用 Groovy 插值的 `sh`、`bat`、`powershell` 或 `pwsh` 步骤时，就会发生这种情况。下面是个简单的示例：


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
  agent any

  parameters {
    string(name: 'STATEMENT', defaultValue: 'hello; ls /', description: 'What should I say?')
  }

  stages {
    stage('Example') {
      steps {
        /* WRONG! */
        sh("echo ${STATEMENT}")
      }
    }
  }
}
```

在此示例中，`sh` 步骤的参数会由 Groovy 计算出来，`STATEMENT` 会被直接插在参数中，就好像 `sh('echo hello; ls /')` 已经写在流水线中一样。当这在代理上被处理时，其不是回显，echoing, `hello; ls /` 的值，而是将回显 `hello` 然后继续列出整个代理的根目录。任何能够控制这种步骤所插入的变量的用户，都能够使 `sh` 步骤在代理上运行任意代码。为了避免这个问题，请确保引用了参数或其他用户控制环境变量的 `sh` 或 `bat` 等步骤要使用单引号，以避免 Groovy 插值。

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
  agent any

  parameters {
    string(name: 'STATEMENT', defaultValue: 'hello; ls /', description: 'What should I say?')
  }

  stages {
    stage('Example') {
      steps {
        /* CORRECT */
        sh('echo ${STATEMENT}')
      }
    }
  }
}
```

凭据错乱，credential mangling, 是另一个问题，当含有特殊字符的凭据传递给某个使用 Groovy 插值的步骤时，可能会发生。当凭证值被破坏时，他就不再有效，也不会再被掩盖在控制台日志中。

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
  agent any

  environment {
    EXAMPLE_KEY = credentials('example-credentials-id') // Secret value is 'sec%ret'
  }

  stages {
    stage('Example') {
      steps {
          /* WRONG! */
          bat "echo ${EXAMPLE_KEY}"
      }
    }
  }
}
```

这里，那个 `bat` 步骤会收到 `echo sec%ret`，Windows 批处理 shell 将简单地去掉 `%`，并打印出值 `secret`。因为只有一个字符的差别，值 `secret` 就不会被掩盖。尽管该值与实际的凭证不一样，但这仍然是敏感信息的一次重大暴露。同样，单引号会避免这个问题。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
  agent any

  environment {
    EXAMPLE_KEY = credentials('example-credentials-id') // Secret value is 'sec%ret'
  }

  stages {
    stage('Example') {
      steps {
          /* CORRECT */
          bat 'echo %EXAMPLE_KEY%'
      }
    }
  }
}
```

### 处理参数

**Handling parameters**

声明式流水线支持开箱即用的参数，允许流水线在运行时通过 [`parameters` 指令](./syntax.md#parameters) 接受用户指定的参数。而在脚本化流水线下的配置参数是通过 `properties` 步骤完成的，这可以在 Pipeline 代码片段生成器中找到。

如果咱们使用 “参数化构建过程，Build with Parameters” 选项将流水线配置为接受参数，那么这些参数可以作为 `params` 变量的成员访问。

假设在 `Jenkinsfile` 中已经配置了一个名为 `Greeting` 的字符串参数，那么他就可以通过 `${params.Greeting}` 访问该参数：


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any

    parameters {
        string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }

    stages {
        stage('Example') {
            steps {
                echo "${params.Greeting} World!"
            }
        }
    }
}
```

<details>
<summary>切换至脚本化 Pipeline</summary>

```groovy
// Jenkinsfile (声明式 Pipeline)
properties([
    parameters([
        string(defaultValue: 'Hello', description: 'How should I greet the world?', name: 'Greeting')
    ])
])

node {
    echo "${params.Greeting} World!"
}
```
</details>

> **注意**：只需在 Pipeline 脚本中加入 `parameters` 指令，Jenkins 经典 UI 中 Pipeline 项目/条目页面左侧的 “立即构建” 就会变为 “Build with Paramters”，而无需在 Pipeline 项目/条目的 “配置” 页面进行 “参数化构建过程” 的设置。

### 处理失败

**Handling failure**

声明式流水线通过其 `post` 小节默认支持强大的失败处理，该小节允许声明许多不同的 "构建后情形，post conditions"，如：`always`、`unstable`、`success`、`failure` 及 `changed` 等。后面 [流水线语法](./syntax.md) 小节提供了关于如何使用各种构建后情形的更多细节。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh 'make check'
            }
        }
    }

    post {
        always {
            junit '**/target/*.xml'
        }
        failure {
            mail to: team@example.com, subject: 'The Pipeline failed :('
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
        try {
            sh 'make check'
        }
        finally {
            junit '**/target/*.xml'
        }
    }
    /* .. snip .. */
}
```
</details>

> 然而，脚本化流水线依赖于 Groovy 内置的 `try`/`catch`/`finally` 语义来处理流水线执行期间的失败。
>
> 在上面的 [测试](#测试) 示例中，`sh` 步骤被修改为绝不会返回非零的退出代码（ `sh 'make check || true'`）。这种方法虽然有效，但意味着接下来的阶段需要检查 `currentBuild.result`，以了解是否有测试失败的情况。
>
> 另一种处理方式是使用一系列的 `try`/`finally` 代码块，其会保留 Pipeline 中失败的早期退出行为，同时仍然给 `junit` 步骤捕捉测试报告的机会。


### 使用多个代理

在前面所有示例中，都只使用了一个代理。这意味着 Jenkins 会在任何有执行器可用的地方分配一个执行器，而不管其是如何被标记或配置的。这种行为不仅可被覆盖，而且 Pipeline 允许在 *同一* Jenkins 文件中利用 Jenkins 环境中的多个代理，这对更高级的用例很有帮助，比如在多个平台上执行构建/测试。

在下面的示例中，`Build` 阶段将在一个代理上进行，而在 `Test` 阶段，构建的结果将在随后的两个分别标记为 `linux` 与 `windows` 的代理上重用。


```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent none

    stages {
        stage('Build') {
            agent any

            steps {
                checkout scm
                sh 'make'
                stash includes: '**/target/*.jar', name: 'app' // 1
            }
        }

        stage('Test on Linux') {
            agent { // 2
                label 'linux'
            }

            steps {
                unstash 'app' // 3
                sh 'make check'
            }

            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }

        stage('Test on Windows') {
            agent {
                label 'windows'
            }

            steps {
                unstash 'app'
                bat 'make check' // 4
            }

            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
    }
}
```

<details>
<summary>切换至脚本化 Pipeline</summary>

```groovy
// Jenkinsfile (脚本化 Pipeline)
stage('Build') {
    node {
        checkout scm
        sh 'make'
        stash includes: '**/target/*.jar', name: 'app' // 1
    }
}

stage('Test') {
    node('linux') { // 2
        checkout scm

        try {
            unstash 'app' // 3
            sh 'make check'
        }
        finally {
            junit '**/target/*.xml'
        }
    }

    node('windows') {
        checkout scm

        try {
            unstash 'app'
            bat 'make check' // 4
        }
        finally {
            junit '**/target/*.xml'
        }
    }
}
```
</details>


1. `stash` 步骤允许捕获与包含模式（ `**/target/*.jar` ）相匹配的文件，以便在 *同一* 流水线内重复使用。一旦流水线完成了他的执行，储藏的文件将从 Jenkins 控制器中删除；

2. `agent`/`node` 中的参数允许任何有效 Jenkins 标签表达式。更多细节请参考 [流水线语法](./syntax.md) 小节；

3. `unstash` 步骤将从 Jenkins 控制器中获取那个命名的 “stash” 到流水线的当前工作区；

4. 这个 `bat` 脚本允许在基于 Windows 的平台上执行批处理脚本。


### 可选的步骤参数

**Optional step arguments**


Pipeline 遵循了 Groovy 语言允许在方法参数周围省略括号的约定。

许多 Pipeline 步骤还使用命名参数语法作为在 Groovy 中创建 Map 的速记方法，其用到语法 `[key1: value1, key2: value2]`。使得像下面这样的语句在功能上等同：

```groovy
git url: 'git://example.com/amazing-project.git', branch: 'master'
git([url: 'git://example.com/amazing-project.git', branch: 'master'])
```

为方便起见，在调用只带一个参数（或只带一个强制参数）的步骤时，可以省略参数名称，例如：

```groovy
sh 'echo hello' /* short form  */
sh([script: 'echo hello'])  /* long form */
```


### 高级脚本化 Pipeline

脚本化 Pipeline 是一种基于 Groovy 的领域特定语言，大多数 [Groovy 语法](http://groovy-lang.org/semantics.html) 无需修改即可在脚本化流水线中使用。


#### 并行执行

[上面小节](#使用多个代理) 中的示例会在两种不同平台上以线性序列方式运行测试。实际上，如果 `make check` 需要 30 分钟完成，那么现在 `Test` 阶段将需要 60 分钟才能完成！

幸运的是，Pipeline 有着用于并行执行脚本化 Pipeline 部分的内建功能，在一个恰如其分地被命名为 `parallel` 步骤中实现。


将上面的示例重构为使用这个 `parallel` 步骤：


```groovy
// Jenkinsfile (脚本化 Pipeline)
stage('Build') {
    /* .. snip .. */
}

stage('Test') {
    parallel linux: {
        node('linux') {
            checkout scm

            try {
                unstash 'app'
                sh 'make check'
            }
            finally {
                junit '**/target/*.xml'
            }
        }
    },
    windows: {
        node('windows') {
            /* .. snip .. */
        }
    }
}
```


现在，假设 Jenkins 环境中存在必要容量，在标记为 `linux` 与 `windows` 节点上的测试将并行执行，而不再是序列执行。


（End）


