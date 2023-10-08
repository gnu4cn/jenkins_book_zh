# Blue Ocean

本章涵盖 Blue Ocean 功能的各个方面，包括如何使用：

- [Blue Ocean 入门](./blue_ocean/get_started.md)，内容包括如何在 Jenkins 中设置 Blue Ocean，以及访问 Blue Ocean 界面；

- 在 Blue Ocean 中 [创建一个新的 Pipeline 项目](./blue_ocean/create_a_new_pipeline_project.md)；

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

如果咱们想要开始使用 Blue Ocean，请参阅 [Blue Ocean 入门](./blue_ocean/get_started.md)。


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



