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



