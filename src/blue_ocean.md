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
