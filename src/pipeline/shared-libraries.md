# 使用共享库扩展 Pipeline

**Extending with Shared Libraries**


随着组织中越来越多的项目采用流水线，可能会出现一些常见的模式，common patterns。通常情况下，在不同的项目之间共用管道的一些部分，来减少冗余并保持代码的 “DRY” 会很有用。


> 什么是 DRY，Don't Repeat Yourself, 参见：[Wikipedia: Don't repeat yourself](https://en.wikipedia.org/wiki/Don't_repeat_yourself)。


Pipeline 支持创建可以在外部源代码控制仓库中定义并加载到现有的 Pipeline 中的 “共享库，Shared Libraries”。


*Jenkins 中共享库入门*


[![Jenkins 中共享库入门](https://img.youtube.com/vi/Wj-weFEsTb0/0.jpg)](https://www.youtube.com/watch?v=Wj-weFEsTb0)


## 定义共享库

**Defining Shared Libraries**


共享库由名称、源代码检索方法（例如通过 SCM）以及可选的默认版本来定义。名称应是一个简短的标识符，因为他将在脚本中使用。

版本可以是该 SCM 所理解的任何东西；例如，分支、标签与提交哈希值都适用于 Git。咱们还可以声明脚本是否需要明确地请求该库（详见下文），或者他是否默认存在。此外，如果咱们在 Jenkins 的配置中指定了某个版本，咱们可以阻止脚本选择某个 *不同* 版本。

指定SCM的最好方法是使用某种 SCM 插件，该插件已经被特别更新以支持新的 API，用于检出任意命名的版本（ *现代 SCM* 选项）。截至目前，最新版本的 Git 和 Subversion 插件均支持这种模式；其他插件应该也会支持。

如果咱们的 SCM 插件没有被集成，那么咱们可以选择 *Legacy SCM*，并选择所提供的任何选项。在这种情况下，咱们需要在 SCM 配置中的某处包含 `${library.yourLibName.version}`，这样在检出代码的时候，插件就会展开这个变量来选择需要的版本。例如，对于 Subversion，咱们可以将 *代码仓库 URL* 设置为 `svnserver/project/${library.yourLibName.version}`，然后使用诸如 `trunk` 或 `branches/dev` 或 `tags/1.0` 等版本。


### 目录结构

**Directory structure**

共享库代码仓库的目录结构如下所示：


```console
(root)
+- src                     # Groovy source files
|   +- org
|       +- foo
|           +- Bar.groovy  # for org.foo.Bar class
+- vars
|   +- foo.groovy          # for global 'foo' variable
|   +- foo.txt             # help for 'foo' variable
+- resources               # resource files (external libraries only)
|   +- org
|       +- foo
|           +- bar.json    # static helper data for org.foo.Bar
```


`src` 目录应类似于标准的 Java 源代码目录结构。执行 Pipelines 时，该目录会被添加到类路径中。


`vars` 目录存放着在流水线中作为变量而暴露的脚本文件。文件的名称就是流水线中变量的名称。因此，如果咱们有一个名为 `vars/log.groovy` 的文件，其中有一个类似 `def info(message)...` 的函数，咱们就可以在流水线中访问这个函数，如 `log.info "hello world"`。咱们可以在这个文件里放上喜欢的任何函数。请继续阅读以下内容，了解更多的示例和选项。


每个 `.groovy` 文件的基本名称应该是一个 Groovy（即 Java）的标识符，通常是 `camelCased` 命名方式的。而所匹配的 `.txt`，如果存在的话，则可以包含文档，通过系统配置的 [markup 格式化程序](https://wiki.jenkins.io/display/JENKINS/Markup+formatting) 进行处理（所以实际上可以是 HTML、Markdown 等，尽管扩展名为 `.txt` 是必须的）。该文档仅在 [全局变量参考](./get_started.md#全局变量参考) 页面上可见，该页面可从导入了共享库的流水线作业导航侧边栏中访问。此外，这些作业必须成功运行一次，才能生成共享库的文档。


这些目录中的 Groovy 源文件会得到与脚本管道中相同的 “CPS 变换”。


> 注：CPS 变换，continuation-passing style transformation, 连续传递式变换，参见：[Continuation-passing style--CPS变换漫谈](https://misakatang.cn/2018/10/04/Continuation-passing-style-CPS%E5%8F%98%E6%8D%A2%E6%BC%AB%E8%B0%88/)。

`resources` 目录允许从外部库中使用 `libraryResource` 步骤来加载相关的非 Groovy 文件。目前这个特性不支持内部库。

根目录下的其他目录被保留给未来的增强功能。


### 全局共享库

**Global Shared Libraries**


根据用例，可以在多个位置定义共享库。*系统管理* » *System* » *Global Pipeline Libraries* 可以根据需要配置尽可能多的库。

![添加全局的流水线库](../images/add-global-pipeline-libraries.png)

由于这些库将是全局可用的，系统中的任何流水线都可以利用这些库中实现的功能。


这些库被认为是 “可信的”：他们可以运行 Java、Groovy、Jenkins 的内部 API、Jenkins 插件或第三方库中的任何方法。这允许咱们定义出将个别不安全的 API 封装在一个更高层次的包装中，可以从任何管道中安全使用的库。请注意， **任何能够向这个 SCM 仓库推送提交的人都可以获得对 Jenkins 的无限访问**。咱们需要 *Overall/RunScripts* 权限来配置这些库（通常这样的权限将被授予 Jenkins 管理员）。


### 文件夹级别的共享库

**Folder-level Shared Libraries**

所创建的任何文件夹构建项目都可以有与之相关的共享库。这种机制允许对文件夹或子文件夹构建项目内的所有流水线进行特定库的范围控制。


基于文件夹构建项目的库不被认为是 “可信的, trusted”：他们就像典型流水线一样在 Groovy 的沙盒中运行。


### 自动共享库

**Automatic Shared Libraries**


其他一些插件可能会带来一些临时定义库的方法。例如，[Pipeline： GitHub Groovy 库](https://plugins.jenkins.io/pipeline-github-lib) 插件就允许脚本使用一个名为 `github.com/someorg/somerepo` 的不受信任库，而无需任何额外的配置。在这种情况下，指定的 GitHub 仓库将从 `master` 分支，使用匿名签出而加载。



## 使用库

**Using libraries**


标记为 *Load implicity* 的共享库允许流水线立即使用由任何此类库所定义的类或全局变量。而要访问其他共享库，`Jenkinsfile` 就需要使用 `@Library` 注解，指定库的名称：


![配置全局的流水线库](../images/configure-global-pipeline-library.png)



