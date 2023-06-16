# 管道入门

**Getting started with Pipeline**

[如前](../pipeline.md) 所述，Jenkins Pipeline 是一套插件，支持在 Jenkins 中实现和集成持续交付管道。Pipeline 提供了一套可扩展的工具，通过 Pipeline DSL 将简单到复杂的交付管道 “作为代码” 建模。

本节描述了如何开始在 Jenkins 中创建咱们的管道项目，并介绍了创建和存储 `Jenkinsfile` 的各种方式。


## 前提条件

要使用 Jenkins 管道，咱们将需要：

- Jenkins 2.x 或更新版本；（1.642.3 之后的旧版本可以工作，但不建议使用。）

- 管道插件，其作为 “建议的插件” 的一部分被安装（在 [安装 Jenkins](../Ch01_Installing_Jenkins.md) 后通过运行 [安装后设置向导](../installation/docker.md#安装后设置向导) 时指定）。

请在在 [管理插件](../managing/plugins.md) 中阅读有关如何安装和管理插件的更多信息。


## 定义管道

[声明式和脚本化管道](../pipeline.md#声明式与脚本式的管道语法) 均属于领域特定语言，用于描述咱们软件交付管道的那些部分。脚本化管道则是以 [Groovy 语法](groovy-lang.org/semantics.html) 的有限形式编写的。

Groovy 语法的相关组件将在本文档中根据需要进行介绍，因此虽然了解 Groovy 是有帮助的，但其并不是使用 Pipeline 所必需的。

可以通过以下方式之一创建管道：

- [通过 Blue Ocean](#经由-blue-ocean) -- 在 Blue Ocean 中设置管道项目后，Blue Ocean UI 可帮助咱们编写管道的 Jenkinsfile 并将其提交到源代码管理；

- [通过经典 UI](#经由经典-UI) —— 咱们可通过经典 UI 直接在 Jenkins 中进入一个基本的 Pipeline；

- [在 SCM 中](#在-SCM-中) -- 咱们可以手动编写 `Jenkinsfile`，可将其提交到咱们项目的源代码控制仓库中。


以这两种方法定义管道的语法是相同的，不过尽管 Jenkins 支持直接在经典用户界面中输入管道，一般认为最好的做法是在 `Jenkinsfile` 中定义管道，然后 Jenkins 将直接从源代码控制系统中加载。

下面这个视频提供了关于如何编写声明式和脚本式管道的基本说明。

*在 Jenkins 中编写管道脚本*

（只提供该视频的中文字幕。）


```
在今天的视频中，我们将

讨论如何

在 Jenkins

[音乐] 中编写管道脚本，

如果您是新来的，欢迎您，

如果您是新来的，您不知道我是谁，



我的名字是 Darin Pope，我是

cloudbees 的开发者倡导者，

大约在 2014 年 12 月，Jenkins

项目

引入了脚本化语法管道，

并且在 2017 年 2 月左右，声明式

管道进入了

现在为什么要

使用

管道你有两种不同的语法，

你可以选择使用声明性的，

或者你可以 在

声明性出来之前选择在 2015 年使用脚本，

你所有的东西都是脚本化的，但是一旦

声明式

在 2017 年 2 月左右成为一般可用 GA，

这给了你一个更好的方法

来构建你的大部分

管道

现在我们今天要看的是什么



如果您从未创建过管道

脚本，那么这是一种非常基本的方法，在

我将向您展示一种

在 Jenkins 本身中执行此操作的快速简便的方法

现在您可能会问什么是 Jenkins

管道，

那么 Jenkins 管道是一套插件


使得在一个文件中创建出作业定义可行


如果您过去使用过 Jenkins，则

可能熟悉

freestyle 作业或 maven 作业，

或者有许多作业类型，但

管道

是唯一可以实现

的 能够将

您的作业定义保存到



诸如 git 之类的 SCM 中，这样您就可以将



作业定义作为代码进行管理让我们谈谈

我们今天用来做演示的东西



我有一个运行 2.277.2 的 Jenkins 控制器



当它被安装时，它是

使用安装建议的插件安装的，

并且我们已经

附加了一个代理到这个控制器

所以让我们继续并直接进入它

我们今天要做的是我们

将创建

两个示例作业 我要说的是

创建一个作业，我要说的是

脚本化的管道，这样你就可以看到

怎么做了 将







作业添加到

SCM

我们今天不会这样做

如果您

刚刚开始使用它会更高级一些 我希望您

了解

如何快速创建管道

作业

并且您可以查看其他视频以

了解如何 保存它并将其拉

回到

作业中供以后使用，所以让我们

继续并向下滚动到

称为管道脚本的部分



，实际上在右侧这里有一个小帮手，

它说

尝试一个示例 管道

让我们继续并单击脚本化

管道，

您将在这里看到这是一个

非常

先进的类型管道让我们

继续并简化它

所以让我们继续并首先将

节点代码块留在顶部然后

我们' 我们将留下一个

原料类型作业发生的阶段，

所以将原料想作，因为咱们很快会看到，



在这里我们想要做的而不是

结果我只是想打个

招呼，

我也会说 echo Hello World

我会把它们改成 singles doubles

对于这个用例来说并不重要 让我们

今天选择 singles 好的

现在我们有

节点 阶段 echo hello world 让我们

点击保存

然后点击构建 现在

你会 在这里看到它运行了，你

可以看到它说在

agent1 上运行，这是我们已经

连接到我们的控制器的代理

，然后它回显出 hello world 所以让我们

再看一次

我们大约有五行 给我们

一个 hello world

节点说确保它在代理上运行



我的阶段是

这个工作中发生的事情然后这是

在这种情况下发生的实际事情

这是一个 echo

现在让我们创建一个声明式作业

所以我们将点击 在仪表板上的新项目，

我们将调用此

声明式管道我们将单击

管道

并再次单击确定我们将向下滚动

到我们的管道脚本

部分我们将尝试一个 hello world

，这个 hello world 正是 与

我们刚刚对

我们的脚本化语法管道所做的相同

现在脚本语法是五行

这一段

大致有额外的空间 11

行

所以让我们继续并保存它只是为了

证明

当它运行时它做的

事情与 我们的脚本语法

让我们继续，现在单击构建，

当我们查看输出时，我们可以

看到它在 agent1 上运行

并且它也回显了 hello world 这



与我们在脚本语法管道中看到的输出完全相同 这



是一个简短的视频，只是为了给

你

一些信息，如果你在



看到我们有两种

不同的语法

可以使用之前从未见过什么是 Jenkins 管道，我们有脚本化语法

，那时我们有声明式语法

在脚本化语法和

声明式语法之间

显然，我们必须使用的只是

脚本化语法，

但是一旦声明式语法出现，这就为

我们提供了构建管道的其他选择，

因为

声明式管道的发布是

基本建议，



当您开始时从声明式语法开始 遇到



声明式语法的一些问题，

无论

我们已经在其他视频中介绍过，

然后您将添加到共享库中，请

查看描述中列出的共享库视频，



或者它可能在我这里



如果声明式库

和共享库的组合开始分崩离析，那么



您也已经将它带到了边缘，



然后才应该

使用脚本化语法完全构建管道，

这是脚本化语法可能

更复杂的原因是的

与我们在声明式中看到的相比，它有点短



一个警告词 有时人们会将



脚本化语法视为一种通用

编程语言，

它不是脚本化语法，

而且声明式语法

仅适用于 ci 目的，

如果你 需要与其他系统集成



使用真正的编程语言 go

java 你的选择无关紧要



在真正的编程语言中编写任何功能，

你可以进行

真正的测试，然后一旦你



编写了它是否是一个简单的脚本

也许你 正在编写一个 bash 脚本

或者一个 powershell 脚本或者



作为一个完整的客户端可能更复杂的东西

现在你可以从命令行调用一些东西



一旦你能够从命令行测试所有这些



然后它是 如果您

使用的是



Linux 或 bat

调用或 powershell 调用（如果您使用的是

Windows）来

调用

您编写的工具（

如果您有任何工具）



如果该视频对

您有帮助，

请在

cloudbeesdevs 上与我们联系



铃铛

，您会



在 cloudbees 电视上有新内容时随时收到通知，感谢收看，

我们将

在下一个视频中见到

您

```

### 经由 Blue Ocean


如果你是 Jenkins Pipeline 的新手，Blue Ocean UI 会帮助咱们 [设置咱们的 Pipeline 项目](../blue_ocean/creating_pipelines.md)，并通过图形化的 Pipeline 编辑器自动为咱们创建和编写咱们的 Pipeline（即 `Jenkinsfile`）。

作为在 Blue Ocean 中设置 Pipeline 项目的一部分，Jenkins 会配置一个安全且经过适当的认证的连接到咱们项目的源代码控制库。因此，咱们通过 Blue Ocean 的 Pipeline 编辑器对 `Jenkinsfile` 所做的任何修改都会自动保存并提交到源代码控制系统。

请在 [Blue Ocean](../blue_ocean.md) 章节和 [Getting started with Blue Ocean](../blue_ocean/getting_started.md) 页面中阅读更多关于 Blue Ocean 的信息。

> *Blue Ocean 的现状*
>
> Blue Ocean 将不再接收更多的功能更新。Blue Ocean 将继续提供易于使用的管道可视化，但他不会得到进一步增强。他只会在重大安全问题或功能缺陷方面得到选择性的更新。
>
> [Pipeline 语法片段生成器](#Pipeline-语法片段生成器) 会在用户以参数定义 Pipeline 步骤时为他们提供帮助。他是创建 Jenkins 管道的首选工具，因为他为咱们 Jenkins 控制器中的管道步骤提供在线帮助。他会使用安装在咱们 Jenkins 控制器上的插件来生成 Pipeline 语法。关于所有可用的 Pipeline 步骤的信息，请参考 [Pipeline 步骤参考页](./steps.md)。


### 经由经典 UI


使用经典用户界面创建的 `Jenkinsfile` 由 Jenkins 本身存储（在 Jenkins 的主目录内）。

通过 Jenkins 经典用户界面创建一个基本 Pipeline：

1. 如果需要，请确保咱们已经登录到 Jenkins；

2. 在 Jenkins 主页（即 Jenkins 经典用户界面的仪表板），点击左上方的 “新建任务”；

![经典 UI 的左侧菜单栏](../images/classic-ui-left-column.png)

3. 在 **输入一个任务名称** 表单字段，为咱们新管道项目指定名称；

    **注意**：Jenkins 会使用这个项目名称在磁盘上创建目录。建议避免在项目名称中使用空格，因为这样做可能会暴露出脚本中不能正确处理目录路径中空格的错误。

4. 向下滚动并点击 **流水线**，然后在页面末尾点击 **确定**，打开管道配置页面（其 **General** 标签页会被选中）；

![新构建项目的创建](../images/new-item-creation.png)

5. 点击页面左侧的 **流水线** 分页标签，以滚动到 **流水线** 部分；

    **注意**：如果咱们是在源码控制系统中定义咱们的 `Jenkinsfile`，请按照下面 [在 SCM 中](#在-SCM-中) 的说明。

6. 在 **流水线** 小节，请确保 **定义** 字段表明是 **Pipeline script** 选项；

7. 将咱们的 Pipeline 代码输入到那个 **脚本** 文本区表单字段；

例如，请复制下面的声明式示例 Pipeline 代码（*`Jenkinsfile（...）`* 标题以下的）或其脚本化版本的等价代码，并将其粘贴到 **脚本** 文本区表单字段。(下面的声明式示例在本程序的其余部分都会用到）。

```groovy
// Jenkinsfile (声明式 Pipeline)
pipeline {
    agent any // 1
    stages {
        stage('Stage 1') {
            steps {
                echo 'Hello world!' // 2
            }
        }
    }
}
```

<details>
    <summary>切换脚本化 Pipeline</summary>

```groovy
// Jenkinsfile （脚本化 Pipeline）
node { // 3
    stage('Stage 1') {
        echo 'Hello World' // 2
    }
}
```
</details>

1). `agent` 指示 Jenkins 为整个管道分配一个执行器（在 Jenkins 环境中任何可用的代理/节点上）和工作空间；

2). `echo` 在控制台输出中写下简单的字符串；

3). `node` 有效地完成与 `agent` 同样的事情（上文）。

![经典 UI 中的示例 Pipeline](../images/example-pipeline-in-classic-ui.png)

**注意**：咱们也可以从 **脚本** 文本区右上方的 **try sample Pipeline** 选项中选择预制的 **脚本化** 流水线实例。请注意，这个字段中没有预制的声明式 Pipeline 的示例。

8. 点击 **保存** 打开这个 Pipeline 的项目/条目视图页面；

9. 在此页面上，点击左侧的 **立即构建** 运行这个 Pipeline。

![经典 UI 中构建项目页面上的左侧菜单栏](../images/classic-ui-left-column-on-item.png)

10. 在左边的 **Build History** 下，点击 **#1** 来访问这个特定 Pipeline 运行的细节；

11. 点击 **Console Output**，查看这个 Pipeline 运行的全部输出。下面的输出显示咱们管道的成功运行。


![`Hello world` 的控制台输出](../images/hello-world-console-output.png)


**注意**：

- 咱们也可以通过点击构建号（如 **#1**）的上下文菜单，直接从仪表板上访问控制台输出；

- 通过经典用户界面定义 Pipeline，对于测试 Pipeline 代码片段，或处理简单的 Pipeline 或不需要从代码仓库检出/克隆源代码的 Pipeline 来说是很方便的。如上所述，与咱们通过 Blue Ocean（ [上文](#经由-blue-ocean) ）或在源代码控制系统（ [下文](#在-SCM-中) ）中定义的 `Jenkinsfile` 不同，输入到 Pipeline 项目 **脚本** 文本区的 `Jenkinsfile` 由 Jenkins 本身在 Jenkins 主目录内存储。因此，为了对咱们 Pipeline 有更大的控制力和灵活性，特别是对于在源代码控制系统中可能会越来越复杂的项目，建议咱们使用 [Blue Ocean](#经由-blue-ocean) 或 [源代码控制系统](#在-SCM-中) 来定义咱们的 `Jenkinsfile`。


### 在 SCM 中

复杂的流水线很难在流水线配置页面的 [经典 UI](#经由经典-ui)  **脚本** 文本区字段内编写和维护。

为了令到复杂流水线的编写更简单，咱们 Pipeline 的 `Jenkinsfile` 可以用文本编辑器或集成开发环境（IDE）编写，并提交到源代码控制系统[（可选择地与 Jenkins 将构建的应用程序代码一起）。然后，作为咱们项目的构建过程的一部分，Jenkins 便可从源代码控制系统中检出咱们的 `Jenkinsfile`，然后继续执行咱们的 Pipeline。

要配置咱们的 Pipeline 项目使用来自源代码控制系统的 `Jenkinsfile`：

1. 按照上面的程序，[通过经典用户界面](#经由经典-ui) 定义咱们的管道，直到咱们到达第五步（访问流水线配置页面上的 **流水线** 部分）；

2. 在 **定义** 字段，请选择 **Pipeline script from SCM** 选项；

3. 在 **SCM** 字段，选择包含咱们 `Jenkinsfile` 的代码仓库的源代码控制系统的类型；

4. 完成特定于咱们存储库源代码控制系统的字段;

**提示**：如果咱们不确定要为某个给定字段指定什么值，请点击其右侧的 **？** 图标以获得更多信息。

![来自源代码控制系统的 Pipeline 脚本](../images/pipeline_script_from_scm.png)

5. 在 **脚本路径** 字段，指定你的 `Jenkinsfile` 的位置（及名称）。这个位置是 Jenkins 检出/克隆包含咱们的 `Jenkinsfile` 的版本库位置，他应该与版本库的文件结构一致。这个字段的默认值假定咱们的 `Jenkinsfile` 被命名为 “Jenkinsfile” 并位于版本库的根部。


只要 Pipeline 被配置了 SCM 轮询触发器，那么当咱们更新指定的存储库时，就会触发新的构建。

> 由于流水线代码（特别是脚本化流水线）是用类似 Groovy 的语法编写的，如果咱们的 IDE 没有正确地语法突出显示咱们的 `Jenkinsfile`，请尝试在 `Jenkinsfile` 的顶部插入行 `#!/usr/bin/env groovy`，footnotegroovy_shebang:[[Shebang line (Groovy syntax)](http://groovy-lang.org/syntax.html#_shebang_line)] 就会纠正这个问题。

> 在使用 `gnu4cn/jenkins_book_zh` 做测试 Pipeline 时，发现要设置 “仪表板” > “系统管理” > “全局安全配置” > “Git Host Key Verification Configuration” > “Host Key Verification Strategy” 为 “No verification”，然后在配置 Pipeline 时，选择之前添加的全局密钥 `xfoss-com` 既可。


## 内建文档

Pipeline 具有内建文档特性，使得创建各种不同复杂程度的 Pipeline 更加容易。这种内建文档是根据 Jenkins 实例中安装的插件自动生成和更新的。

内建文档可以全局性地在 `${YOUR_JENKINS_URL}/pipeline-syntax` 中找到。同样的文档也在任何配置好的 Pipeline 项目边栏中作为 **流水线语法** 而被链接出来。

![经典 UI 构建项目左侧栏](../images/classic-ui-left-column-on-item.png)


### 代码片段生成器

**Snippet Generator**

内建的 “代码片段生成器” 实用工具有助于为单个步骤创建少量的代码，发现那些由插件所提供的新步骤，或为特定步骤试验不同的参数。

代码片段生成器是由 Jenkins 实例可用步骤列表动态产生出的。可用步骤的数量取决于所安装的插件，这些插件会显式暴露在 Pipeline 中使用的步骤。

要用代码片段生成器生成一个步骤片段：

1. 从某个已配置好的 Pipeline 导航至 **流水线语法** 链接，或在 `${YOUR_JENKINS_URL}/pipeline-syntax` 处；

2. 在 **示例步骤** 下拉菜单中选择所需的步骤；

3. 使用 **示例步骤** 下拉菜单下面动态产生出的区域来配置所选步骤；

4. 点击 "生成流水线脚本"，创建一个 Pipeline 代码片段，可将其复制并粘贴到流水线中。

![代码片段生成器](../images/snippet-generator.png)

要访问有关所选步骤的其他信息和/或文档，请点击帮助图标（**？**）。

### 全局变量参考

**Global Variable Reference**

除了仅显示步骤的 Snippet Generator 之外，Pipeline 还提供了一个内建的 “ **全局变量参考，Global Variable Reference** ”。与代码片段生成器一样，他也由插件动态产生。然而，与代码片段生成器不同的是，全局变量参考仅包含由 Pipeline 或插件所提供变量的文档，这些变量可用于流水线。

全局变量在 Pipeline 中直接可用，而不是作为步骤。他们暴露了可在咱们 Pipeline 脚本中访问的方法与变量。

Pipeline 中默认提供的变量有：

#### `env`

暴露环境变量，例如： `env.PATH` 或 `env.BUILD_ID`。请参考 `${YOUR_JENKINS_URL}/pipeline-syntax/globals#env` 的内建全局变量参考，以获得 Pipeline 中可用的完整、最新环境变量列表。


#### `params`

将为管道定义的所有参数公开为只读 [映射，Map](http://groovy-lang.org/syntax.html#_maps)，例如：`params.MY_PARAM_NAME`。


#### `currentBuild`

可用于发现有关当前执行的 Pipeline 的信息，其属性包括 `currentBuild.result`、`currentBuild.displayName` 等。请参考 `${YOUR_JENKINS_URL}/pipeline-syntax/globals#currentBuild` 的内置全局变量参考，以获得 `currentBuild` 的完整和最新的属性列表。


#### `docker`

`docker` 变量提供了从 Pipeline 脚本方便地访问 Docker 相关功能的方法。详情请参考：`${YOUR_JENKINS_URL}/pipeline-syntax/globals#docker`。


#### `pipeline`

`pipeline` 步骤允许咱们以更有条理的方式定义咱们的管线。更多信息请见 [wiki](https://github.com/jenkinsci/pipeline-model-definition-plugin/wiki/Getting-Started)。


#### `scm`

在多分支项目构建中表示 SCM 的配置。使用 `checkout scm` 来检出与 Jenkinsfile 相匹配的那些源代码。

在用 *Pipeline script from SCM* 配置的独立项目中，咱们也可以使用这个变量，不过在这种情况下，签出的只是分支中的最新版本，可能比加载 Pipeline 的版本还要新。

下面这个视频讲了在 Jenkins Pipeline 中使用 `currentBuild` 变量。

[![在 Jenkinsfile 中使用 `currentBuild` 环境变量的几种方式：从简单到复杂](https://img.youtube.com/vi/gcUORgHuna4/0.jpg)](https://www.youtube.com/watch?v=gcUORgHuna4)

[![在 Jenkinsfile 中使用 `currentBuild` 环境变量的几种方式：从简单到复杂](https://img.youtube.com/vi/4KghHJEz5no/0.jpg)](https://www.youtube.com/watch?v=4KghHJEz5no)

视频内容总结：

- 可以通过 `echo "Build number is ${currentBuild.number}"` 在以 groovy 作为语法的 `Jenkinsfile` 中使用环境变量（其中 `${}` 属于 groovy 中与其他语言一样的字符串插值语法）；

- 可以通过 `currentBuild.result = 'FAILURE'` 这种方式，修改/设置 `currentBuild` 环境变量，并通过 `echo currentBuild.result` 访问到该环境变量；

- 通过 `@Library("shared-library")` 这样的 groory 导入库语法，编写更复杂的 Pipeline Jenkinsfile 时，可在库中使用 `currentBuild` 环境变量 `currentBuild.rawBuild`，使用该变量的 `addAction` 方法，编写咱们定制的方法 `addSidebarLink`，从而实现更复杂的构建流水线；

- 在第 3 步的基础上，通过 groovy 的 `def` 关键字定义出一些变量，并结合 `docker` 命令与 DockerHub 登录凭据登录 DH，并将 `docker build` 出的镜像 `push` 到 DH，然后返回一个 DH 上该镜像的链接

> 其中：
>
> 1. `sh` 步骤多行的情形，此时用 `"""` 把多行的 SH 脚本包围起来即可；
>
> 2. `sh` 不仅是个步骤，也可以作为函数使用，如 `sh(returnStdout: true, script:"docker push ${imageTag} | grep sha256 | awk -F':' '{print \$4}' | awk '{print \$1}'")`。


