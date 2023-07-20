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
    <td>在 `pipeline` 代码块的顶层及各个 `stage` 代码块中</td>
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


