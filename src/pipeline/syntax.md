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


**顶级代理和阶段性代理之间的区别，differences between top level agents and stage level agents**


当应用了 `options` 指令时，在顶层或阶段级别添加某个代理时，会有一些细微的差别。请查看 [选项](#选项) 部分了解更多信息。



**顶层的代理，Top Level Agents**


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

此超时将包括代理配置时间。由于超时包括了代理配置时间，因此在代理分配延迟的情况下，流水线就可能会失败。


**参数, Parameters**



