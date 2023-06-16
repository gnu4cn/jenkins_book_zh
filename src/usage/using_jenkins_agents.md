# 使用 Jenkins 代理

Jenkins 的架构是为分布式的构建环境设计的。他允许我们为每个构建项目使用不同的环境，在并行运行作业的多个代理之间平衡工作负荷。

Jenkins 控制器是 Jenkins 安装中的原始节点，the original node。Jenkins 控制器管理 Jenkins 代理并协调他们的工作，包括在代理上调度作业与监控代理。代理可以是使用本地或云计算机连接到 Jenkins 控制器。

代理需要 Java 安装并与 Jenkins 控制器建立网络连接。请观看下面 3 分钟的视频，了解 Jenkins 代理的简要说明。


*什么是 Jenkins 代理*



[![在 Jenkinsfile 中使用 `currentBuild` 环境变量的几种方式：从简单到复杂](https://img.youtube.com/vi/4KghHJEz5no/0.jpg)](https://www.youtube.com/watch?v=4KghHJEz5no)
