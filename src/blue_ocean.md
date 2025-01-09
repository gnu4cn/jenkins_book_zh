# Blue Ocean

本章涵盖 Blue Ocean 功能的各个方面，包括如何使用：

- [Blue Ocean 入门](./blue_ocean/getting_started.md)，内容包括如何在 Jenkins 中设置 Blue Ocean，以及访问 Blue Ocean 界面；

- 在 Blue Ocean 中 [创建一个新的 Pipeline 项目](./blue_ocean/create_a_pipeline.md)；

- 使用 Blue Ocean 的 [仪表盘](./blue_ocean/dashboard.md)。

- 使用 [“活动, Activity” 视图](./blue_ocean/activity_view.md)，其中可以访问到 [当前和历史运行数据](./blue_ocean/activity_view.md#活动)、流水线的 [分支](./blue_ocean/activity_view.md#分支)，以及任何开放的 [拉取请求](./blue_ocean/activity_view.md#拉取请求)；

- 使用 [“流水线运行详情” 视图](./blue_ocean/details_view.md)，访问特定流水线或项目运行的控制台输出等详细信息；

- 使用 [“流水线编辑器”](./blue_ocean/pipeline_editor.md)，修改流水线代码，然后咱们可以将其提交到源代码控制系统 SCM 中。


本章面向所有技能水平的 Jenkins 用户，但初学者可能需要参考 [“流水线”](./pipeline.md) 章节，以理解本章涵盖的一些主题。



有关 Jenkins 用户手册内容的概述，请参阅 [用户手册概述](./Ch00_Overview.md)。


{{#include ./pipeline/get_started.md:56:58}}
>
> 可用的 Pipeline 可视化替代选项，比如 [Pipeline: Stage View](https://plugins.jenkins.io/pipeline-stage-view/)  与 [Pipeline Graph View](https://plugins.jenkins.io/pipeline-graph-view/) 插件，提供了部分相同功能。虽然他们不能完全替代 Blue Ocean，但社区鼓励为这些插件的持续开发做出贡献。
>
{{#include ./pipeline/get_started.md:60}}


## 何为 Blue Ocean？

Blue Ocean 作为现有工具，提供了易于使用的 Pipeline 可视化功能。其旨在重新构思 Jenkins 用户体验，从头开始专为 [Jenkins Pipeline](./pipeline.md) 设计。Blue Ocean 的目标是减少混乱，增加全体用户的清晰度。

然而，Blue Ocean 将不再接收功能或增强性的更新。他仅会接收针对重大安全问题，或功能缺陷的有选择性更新。如果咱们刚刚开始使用 Jenkins，仍然可以使用 Blue Ocean，或者可以考虑替代方案，例如 [Pipeline: Stage View](https://plugins.jenkins.io/pipeline-stage-view/) 和 [Pipeline Graph View](https://plugins.jenkins.io/pipeline-graph-view/) 插件。这些插件提供了部分相同功能。

总而言之，Blue Ocean 的主要功能包括：

- 持续交付 (CD) 流水线的 **复杂可视化**，可快速、直观地了解流水线状态；

- **流水线编辑器** 通过可视化流程，引导用户创建流水线，使流水线创建更加简单易行；

- **个性化，personalization**，以满足团队每个成员基于角色的需求；

- 在需要干预或出现问题时 **精确定位，pinpoint precision**。Blue Ocean 可显示需要关注的地方，便于处理异常情况，提高工作效率；

- **原生的分支和拉取请求集成**，使开发人员在 GitHub 和 Bitbucket 上，协作处理代码时能最大限度地提高工作效率；

如果咱们想要开始使用 Blue Ocean，请参阅 [Blue Ocean 入门](./blue_ocean/getting_started.md)。


[![Jenkins Blue Ocean: 每个团队的持续交付](https://img.youtube.com/vi/k_fVlU1FwP4/0.jpg)](https://www.youtube.com/watch?v=k_fVlU1FwP4)


## 常见问题（FAQs）


### 为什么会有 Blue Ocean？

**Why does Blue Ocean exist**?

DevOps 世界已经从纯粹功能性的开发人员工具，过渡到作为 “开发人员体验” 一部分的开发人员工具。他不再是单一的工具，而是开发人员全天使用的众多工具，以及这些工具如何协同工作，为开发人员实现更好的工作流程。

Heroku、Atlassian 和 GitHub 等开发者工具公司，提高了良好开发者体验的标准。渐渐地，开发人员越来越喜欢那些不仅功能强大，而且能无缝融入现有工作流程的工具。这种转变代表着对设计和功能的更高标准，开发人员期待着卓越的用户体验。Jenkins 需要提升以满足这一更高标准。

对于许多 Jenkins 用户来说，创建出持续交付管道并将其可视化，一直是件非常有价值的事情。Jenkins 社区为满足他们的需求而创建的插件，就证明了这一点。这也表明有必要重新审视 Jenkins 目前是如何表达这些概念的，并将交付流水线，视为 Jenkins 用户体验的核心主题。

他不仅仅是持续交付概念，还包括开发人员日常使用的工具，如 GitHub、Bitbucket、Slack、Puppet 或 Docker。其不仅涉及 Jenkins，还包括围绕 Jenkins 的开发人员工作流程，其中包含多种工具。


新团队在学习如何积累他们自己的 Jenkins 经验时，可能会遇到挑战。但是，他们的目标是一致的，即通过更稳定地交付更好的软件，来缩短上市时间。作为一个由 Jenkins 用户和贡献者组成的社区，我们可以共同定义理想的 Jenkins 体验。随着时间的推移，开发人员对良好用户体验的期望会发生变化，而 Jenkins 项目也需要接受这些期望。


Jenkins 社区一直致力于打造技术能力最强、可扩展性最强的软件自动化工具。如果今天不彻底改变 Jenkins 开发人员的体验，就可能意味着未来会有封闭源代码选项，试图利用这一点。


Blue Ocean 就是为了满足当时这种需求而诞生的。然而，随着时间的推移，更多的现代工具出现并取代了他。现在，其他类似功能的插件已经崛起。因此，Blue Ocean 的任何新开发或改进都已停止。如果你有兴趣为类似功能的插件做出贡献，请考虑上文 [何为 Blue Ocean？](#何为-blue-ocean) 一节中提出的其他选择。



[![Jenkins World 2016 - Blue Ocean：Jenkins的全新用户体验](https://img.youtube.com/vi/mn61VFdScuk/0.jpg)](https://www.youtube.com/watch?v=mn61VFdScuk)


### 这个名字从何而来？

**Where is the name from**?

Blue Ocean 这一名称，来自 [“蓝海策略，Blue Ocean Strategy”](https://en.wikipedia.org/wiki/Blue_Ocean_Strategy) 一书。这种策略涉及在更大的无竞争空间，而不是在有竞争的空间内，看待战略问题。更简单地说，可以引用冰球传奇人物韦恩-格雷茨基（Wayne Gretzky）的名言： “滑向冰球即将出现的地方，而不是冰球已经出现的地方”。


### Blue Ocean 支持自由风格的作业吗？


**Does Blue Ocean support freestyle jobs**?


Blue Ocean 的目标是提供与流水线相关的出色体验，并与 Jenkins 实例中已配置的任何自由风格作业相兼容。不过，咱们将无法受益于为流水线构建的一些功能，例如流水线的可视化。


Blue Ocean 的设计具有可扩展性，因此 Jenkins 社区可以扩展 Blue Ocean 的功能。虽然 Blue Ocean 不会再添加任何其他功能，但他仍提供了流水线可视化，以及用户认为有价值的其他一些功能。


### 这对咱们的插件有什么影响？

**What does this mean for my plugins**?


可扩展性是 Jenkins 的核心功能之一，而能够扩展 Blue Ocean UI 也同样重要。`<ExtensionPoint name=...>` 可用于 Blue Ocean 的标记中，为插件对用户界面作出贡献，留下了空间。这意味着插件可以拥有自己的 Blue Ocean 扩展点。Blue Ocean 本身就是使用这些扩展点实现的。


扩展照常由插件提供。插件必须包含一些额外的 JavaScript，才能连接到 Blue Ocean 的扩展点。为 Blue Ocean 用户体验做出贡献的开发者，会相应地添加这些 JavaScript。


### 目前用到的技术有哪些？

**What technologies are currently in use**?


Blue Ocean 是作为一组 Jenkins 插件构建的。与其他插件的关键区别在于，Blue Ocean 提供了自己的 HTTP 请求端点，在不使用现有的 Jenkins UI 标记或脚本，而是通过不同路径来提供 HTML/JavaScript。Blue Ocean 的 JavaScript 组件使用 React.js 和 ES6 构建。受 React.js 这个优秀开源项目的启发，咱们可以在 [构建 React 应用插件](https://nylas.com/blog/react-plugins) 这篇博文中，找到相关信息，建立了一个允许来自任何带有 JavaScript 的 Jenkins 插件的扩展的 `<ExtensionPoint>` 模式。如果扩展加载失败，其故障将被隔离。


### 在哪里可以找到源代码？

源码可在 GitHub 上找到：


- [Blue Ocean](https://github.com/jenkinsci/blueocean-plugin)

- [Jenkins 设计语言](https://github.com/jenkinsci/jenkins-design-language)


### 加入 Blue Ocean 社区


由于 “Blue Ocean” 的开发工作已经冻结，我们预计不会再为其代码库的新功能再做出贡献。不过，咱们仍然可以通过以下几种方式加入社区：


- 在 Gitter 上与社区和开发团队交谈 [chat on gitter](https://app.gitter.im/#/room/#jenkinsci_blueocean-plugin:gitter.im)；

- [在 JIRA 中针对 `blueocean-plugin` 组件](https://issues.jenkins.io/)，发起功能请求或报告错误；

- 订阅 [Jenkins 用户邮件列表](https://groups.google.com/g/jenkinsci-users) 并提问；

- 开发人员？我们已经 [标注了一些问题](https://issues.jenkins.io/issues/?filter=16142)，非常适合想要开始开发 Blue Ocean 的人。别忘了来 Gitter 聊天室做个自我介绍！


（End）


