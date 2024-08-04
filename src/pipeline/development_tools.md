# 流水线开发工具

**Pipeline Development Tools**

Jenkins 的流水线包含了 [内置文档](#内建文档) 和 [代码片段生成器](#代码片段生成器)，他们是开发时的关键资源。他们提供了根据当前安装的 Jenkins 与相关插件的版本进行定制的详细帮助和信息。在本节中，我们将讨论可能有助于开发 Jenkins 流水线的其他工具和资源。


## 命令行的流水线静态分析程序

**Command-line Pipeline Linter**

Jenkins 可以在实际运行某个声明式流水线之前从命令行中验证，或者说 “[静态分析，lint](https://en.wikipedia.org/wiki/Lint_(software))”。这可以通过使用 Jenkins CLI 命令或带适当参数的 HTTP POST 请求来完成。我们建议使用 [SSH 接口](../managing/jenkins_cli.md#通过-ssh-使用命令行界面) 来运行静态分析程序。关于如何正确配置 Jenkins 以实现安全命令行访问的细节，请参见 [Jenkins CLI 文档](../managing/jenkins_cli.md)。


```bash
# 通过 SSH 下的 CLI 进行静态检查
# ssh (Jenkins CLI)
# JENKINS_PORT=[sshd port on controller]
# JENKINS_HOST=[Jenkins controller hostname]
ssh -p $JENKINS_PORT $JENKINS_HOST declarative-linter < Jenkinsfile
```

> 在执行了 `alias jenkins-cli='ssh -l ci_cd_scm -p 32222 ci.xfoss.com'` 时，执行如下命令：


```bash
jenkins-cli declarative-linter < Jenkinsfile
```

```bash
# 使用 `curl` 通过 HTTP POST 进行静态检查
# curl (REST API)
# Assuming "anonymous read access" has been enabled on your Jenkins instance.
# JENKINS_URL=[root URL of Jenkins controller]
# JENKINS_CRUMB is needed if your Jenkins controller has CRSF protection enabled as it should
JENKINS_CRUMB=`curl "$JENKINS_URL/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)"`
curl -X POST -H $JENKINS_CRUMB -F "jenkinsfile=<Jenkinsfile" $JENKINS_URL/pipeline-model-converter/validate
```


### 示例


下面是流水线静态检查器的两个实际例子。第一个示例显示了当传递一个无效的 `Jenkinsfile` 给静态检查器时的输出，这个文件缺少 `agent` 声明的部分。


```groovy
// Jenkinsfile
pipeline {
  agent
  stages {
    stage ('Initialize') {
      steps {
        echo 'Placeholder.'
      }
    }
  }
}
```

```console
# 无效 Jenkinsfile 的静态检查器输出
# 传递一个未包含 `agent` 小节的 Jenkinsfile

$ jenkins-cli declarative-linter < ./Jenkinsfile
Errors encountered validating Jenkinsfile:
WorkflowScript: 4: Not a valid section definition: "agent". Some extra configuration is required. @ line 4, column 5.
       agent
       ^

WorkflowScript: 3: Missing required section "agent" @ line 3, column 1.
   pipeline {
   ^

```

在第二个示例中，这个 `Jenkinsfile` 已经被更新，以包括缺少的在 `agent` 上的 `any`。现在静态检查器就会报告说这个流水线脚本是有效的。


```groovy
// Jenkinsfile
pipeline {
  agent any
  stages {
    stage ('Initialize') {
      steps {
        echo 'Placeholder.'
      }
    }
  }
}
```

```console
# 有效 Jenkinsfile 的静态检查器输出

$ jenkins-cli declarative-linter < ./Jenkinsfile
Jenkinsfile successfully validated.
```


## Blue Ocean 编辑器


[Blue Ocean 的流水线编辑器](../blue_ocean/pipeline_editor.md) 提供了一种 [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) 方式来创建声明式流水线。该编辑器提供了某个流水线中所有阶段、平行分支及步骤的结构化视图。该编辑器在进行流水线修改时进行验证，在提交之前就消除了许多错误。而在幕后，其仍会生成声明式流水线代码。


{{#include ./get_started.md:56:60}}


## 附带修改的 “回放” 流水线运行

**"Replay" Pipeline Runs with Modifications**


通常情况下，流水线将在经典的 Jenkins Web UI 中定义，或者通过提交到源码控制系统中的 `Jenkinsfile` 定义。不幸的是，这两种方法对于流水线的快速迭代或原型设计都不理想。“回放，Replay” 特性允许快速修改和执行现有的流水线，而无需改变流水线的配置或创建新的提交，commit。


### 用法

要使用 “回放” 特性：

1. 在构建历史中选择一个以前完成的运行；


![回放操作的前一次流水线运行](replay-previous-run.png)


2. 点击左侧菜单中的 “回放”；


![回放操作的左侧菜单栏](replay-left-bar.png)


3. 进行修改并点击 "运行"；

![回放操作的已修改流水线](replay-modified.png)


4. 检查修改后的结果。


一旦咱们对修改感到满意，就可以使用回放来再次查看他们，将他们复制到咱们的 Pipeline 作业或 `Jenkinsfile` 中，然后使用咱们常规的工程流程提交这些修改。


### 特点

**Features**


- **可在同一运行中多次调用** -- 可以方便地对不同修改进行平行测试；

- **也可以对仍在进行中的流水线运行进行调用** -- 只要流水线包含语法正确的 Groovy 代码并且能够启动，就可以被回放；

- **引用的共享库代码也可以修改** -- 如果流水线运行引用了 [共享库](./shared-libraries.md)，那么共享库的代码也会作为 Replay 页面的一部分显示并可以修改；

- **通过专门的 "运行/回放" 权限进行访问控制** -- 这一点由 “作业/配置” 所暗示。如果流水线是不可配置的（如多分支的分支流水线）或 “作业/配置” 未被授权，用户仍然可以通过回放试验流水线的定义;

- **可用于重新运行，Re-run** -- 缺乏 “运行/回放” 权限但被授予了 “作业/构建” 的用户仍然可以使用回放来以同一流水线定义重新运行构建。


### 局限性

**Limitations**


- **有语法错误的流水线运行不能被重放** -- 这意味着他们的代码无法被查看，在其中所做的任何修改也不能被检索到。当使用回放进行一些较为重要的修改时，请在运行之前将咱们的修改保存到 Jenkins 之外的文件或编辑器中。参见 [JENKINS-37589](https://issues.jenkins.io/browse/JENKINS-37589)；

- **所回放的流水线行为可能与通过其他方法启动的运行有区别** - 对于不属于多分支流水线一部分的流水线，原始运行和回放运行的代码提交信息可能不同。参见 [JENKINS-36453**](https://issues.jenkins.io/browse/JENKINS-36453)



## IDE 集成

**IDE Integrations**


### Eclipse 的 Jenkins 编辑器


Eclipse 的 `Jenkins Editor` 插件可以在 [Eclipse Marketplace](https://marketplace.eclipse.org/content/jenkins-editor) 上找到。这个特别的文本编辑器提供了一些用于定义流水线的功能，例如：

- 通过 [Jenkins 的静态检查器检查](#命令行的流水线静态分析程序) 而验证流水线脚本。检查失败会显示为 eclipse 标记；

- 带有专用图标的大纲（用于声明式 Jenkins 流水线）；

- 语法/关键词高亮显示；

- Groovy 语法检查。


> 这个 Jenkins 编辑器插件是个第三方工具，不受 Jenkins 项目方所支持。


### VisualStudio Code 的 Jenkins 流水线静态检查器连接器


**VisualStudio Code Jenkins Pipeline Linter Connector**


[VisualStudio Code](https://code.visualstudio.com/) 的 `Jenkins Pipeline Linter Connector` 扩展会将咱们当前打开的文件，推送到咱们的 Jenkins 服务器，并在 VS Code 中显示验证结果。


你可以在 VS Code 的扩展浏览器中找到这个扩展，也可以在以下网址找到：

[marketplace.visualstudio.com/items?itemName=janjoerke.jenkins-pipeline-linter-connector](https://marketplace.visualstudio.com/items?itemName=janjoerke.jenkins-pipeline-linter-connector)


该扩展会在 VS Code 中增加了四个设置项，选择咱们打算用来验证的 Jenkins 服务器。

- `jenkins.pipeline.linter.connector.url` 是 Jenkins 服务器所期望的包含咱们打算验证 `Jenkinsfile` 的 POST 请求端点，the endpoint。通常，这会指向 `<your_jenkins_server:port>/pipeline-model-converter/validate`；

- `jenkins.pipeline.linter.connector.user` 允许咱们指定咱们的 Jenkins 用户名；

- `jenkins.pipeline.linter.connector.pass` 允许咱们指定咱们的 Jenkins 密码；

- 如果咱们的 Jenkins 服务器启用了 CRSF 保护，那么就必须指定 `jenkins.pipeline.linter.connector.crumbUrl`。通常这会指向 `<your_jenkins_server:port>/crumbIssuer/api/xml?xpath=concat（//crumbRequestField,%22:%22,//crumb）`。


### Neovim 的 `nvim-jenkinsfile-linter` 插件

这个 [`nvim-jenkinsfile-linter`](https://github.com/ckipp01/nvim-jenkinsfile-linter) Neovim 插件允许咱们通过使用 Jenkins 实例的流水线静态检查器 API 来验证 `Jenkinsfile`，并在编辑器中报告任何既有的诊断信息。


### Atom 的 `linter-jenkins` 包

这个 [`linter-jenkins`](https://atom.io/packages/linter-jenkins) Atom 软件包允许咱们通过使用运行中的 Jenkins 的流水线静态检查器 API 来验证某个 `Jenkinsfile`。咱们可以直接从 Atom 包管理器中安装他。他还需要安装 [Atom 中的 `Jenkinsfile` 语言支持](https://atom.io/packages/language-jenkinsfile)。


### Sublime Text 的 `Jenkinsfile` 包


这个 [`Jenkinsfile`](https://github.com/june07/sublime-Jenkinsfile) Sublim Text 包允许咱们通过安全通道（SSH）使用运行中的 Jenkins 实例的流水线静态检查器 API 来验证某个 `Jenkinsfile`。咱们可以直接从 Sublime Text 包管理器中安装他。


咱们可以在 Sublime Text 界面中的 Package Control 包，在 GitHub 上， 或在 packagecontrol.io 找到该包：

- [https://github.com/june07/sublime-Jenkinsfile](https://github.com/june07/sublime-Jenkinsfile)

- [https://packagecontrol.io/packages/Jenkinsfile](https://packagecontrol.io/packages/Jenkinsfile)


## 流水线的单元测试框架

**Pipeline Unit Testing Framework**


[流水线的单元测试框架](https://github.com/jenkinsci/JenkinsPipelineUnit) 允许咱们在完全运行流水线和 [共享库](./shared-libraries.md) 之前对其进行 [单元测试](https://en.wikipedia.org/wiki/Unit_testing)。他提供了一个模拟执行环境，其中真实的流水线步骤会被模拟对象取代，咱们可以用他来检查预期行为。这个框架有着新的和略微粗糙的边缘，但很有希望，new and rough around the edges, but promising。这个项目的 [README](https://github.com/jenkinsci/JenkinsPipelineUnit/blob/master/README.md) 包含了一些示例及使用说明。
