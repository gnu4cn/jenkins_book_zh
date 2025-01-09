# 运行流水线

**Running Pipelines**

## 运行流水线

**Running a Pipeline**


### 关于多分支

**Multibranch**


请参阅 [多分支文档](./branches.md) 了解更多信息。


### 参数

**Parameters**


请参阅 [`Jenkinsfile` 文档](./jenkinsfile.md#处理参数) 了解更多信息。


## 重启或重新运行流水线

**Restarting or Rerunning a Pipeline**


有数种可以重新运行或重新启动一个已完成管道的方法。


### 重放

**Replay**


请参阅 [重放文档](./development_tools.md#修改后的重放流水线运行) 了解更多信息。


### 从某个阶段重启

**Restart from a Stage**

咱们可以从任何已完成流水线中运行的任何顶层阶段重新启动该流水线。比如，这允许咱们从某个由于瞬态或环境因素而失败的阶段重新运行流水线。该流水线的所有输入将是相同的。这包括 SCM 信息、构建参数，以及原始流水线中的任何 `stash` 步骤调用的内容（如有指定的话）。


#### 怎样使用

Jenkinsfile 中无需额外配置来允许咱们重新启动咱们的声明式流水线中的阶段。这是声明式流水线的固有特性，可以自动使用。


**从经典 UI 重启**

一旦咱们的管道完成，其不管是成功还是失败，咱们都可以前往经典用户界面中该次运行的侧面板，点击 “从指定阶段重新运行”。


![从指定阶段重新运行](../images/restart-stages-sidebar.png)


咱们将被提示从原始运行中执行的顶层阶段列表中选择，以他们被执行的顺序。由于先前失败而被跳过的阶段将不能被重新启动，但由于 `when` 条件不满足而被跳过的阶段将可被选中。一组 `parallel` 阶段的父级阶段，或者一组顺序运行的嵌套 `stages` ，也将不可用 -- 这里只允许那些顶层阶段。


![阶段重启下拉菜单](../images/restart-stages-dropdown.png)


一旦咱们选择了要重启的阶段并点击提交，那么一个新的构建将被启动，并有新的构建编号。所选阶段之前的所有阶段将被跳过，管道将在所选阶段开始执行。从那时起，管道将正常地运行。


**从 Blue Ocean 用户界面重启**

阶段的重启也可以在 Blue Ocean 用户界面中完成。一旦咱们的流水线完成，不管是成功还是失败，咱们都可以点击代表该阶段的节点。然后咱们可以点击该阶段的重启链接。

![从 Blue Ocean 用户界面重启某个阶段](../images/pipeline-restart-stages-blue-ocean.png)


{{#include ./get_started.md:56:59}}
> 流水线可视化的其他选择，如 [流水线：阶段视图](https://plugins.jenkins.io/pipeline-stage-view/) 和 [流水线图形视图](https://plugins.jenkins.io/pipeline-graph-view/) 插件都是可用的，并提供一些同样功能。虽然不能完全取代 Blue Ocean，但我们鼓励社区为这些插件的继续发展做出贡献。


#### 保留 “存储” 以供重启的阶段使用

**Preserving `stash`es for Use with Restarted Stages**

通常，当咱们在流水线中运行 `stash` 步骤时，无论流水线结果如何，产生的成品暂存（stash）都会在流水线完成时被清空。由于 `stash` 成品在创建他们的流水线运行之外是不可访问的，这对使用并未产生任何限制。但是在声明式阶段重启时，咱们可能希望能够 `unstash` 咱们的重启阶段前运行阶段中的成品。

> 关于 `stash`：`stash` 是指将修改存储于某个隐藏之处（`stash` 栈）。参见：[怎样使用 `git stash` 命令](https://www.freecodecamp.org/news/git-stash-commands/)


为了实现这一点，有一项作业属性允许咱们配置已完成运行的最大数量，这些运行的 `stash` 成品应该被保留下来以便在重启的运行中重用。咱们可以指定从 `1` 到 `50` 的任何数字作为要保存的运行数量。

这项作业属性可以在咱们声明式流水线的 `options` 小节进行配置，如下所示：


```groovy
options {
    preserveStashes() // 1
    // or
    preserveStashes(buildCount: 5) // 2
}
```

1. 默认保留的运行次数是 `1`，仅最近完成的构建；

2. 如果为 `buildCount` 指定的数字超出了 `1` 到 `50` 的范围，流水线将以验证错误而失败。

当某个流水线完成时，他将检查是否有任何先前完成的运行应该清除其 `stash` 成品。


## Jenkins 中编排作业

**Scheduling jobs in Jenkins**


作业编排功能允许咱们安排作业在下班时间或停机时间自动运行。作业编排可以帮助咱们在 Jenkins 使用量增加时扩展咱们的环境。下面这个视频提供了对作业编排功能及其各种配置选项的深入了解。


[![怎样将某项 Jenkins 作业编排为每小时运行](https://img.youtube.com/vi/JhvVJtYFUm0/0.jpg)](https://www.youtube.com/watch?v=JhvVJtYFUm0)

视频内容总结：

1. Jenkins 作业可以手动运行，可以由某个 Web 钩子触发，也可以定时运行；

2. 定时运行 Jenkins 作业，是经由 “声明式指令生成器” 中的 “triggers: Triggers” 指令，“cron: Build periodically” 完成的；

3. 这里的 `cron` 与 \*nix 中 `cron` 有略微差异，包括了 `H` 及 `@midnight` 等语法，其中 `H` 指 `hash`，可避免在某个时刻过多的作业（作业竞争），`@midnight` 这样的写法也可以避免作业竞争；

4. `TZ=America/New_York` 这种写法可以给 Jenkins `cron` 加入时区；

5. 将声明式指令生成器生成的代码片段：


```groovy
triggers {
    cron '* * * * *'
}
```

放在 Jenkinsfile 中：

```groovy
pipeline {
    agent any

    triggers {
        cron '* * * * *'
    }

    // --跳过代码--
}
```

6. 在 Jenkinsfile 中配置的 `triggers`，只有在构建一次后，才会生效；

7. 在以分钟为粒度的 `cron` 无法满足咱们的需要时，就要使用具有秒级认知的系统，比如 rest api 调用，或 Jenkins CLI 来满足咱们的目标。


（End）


