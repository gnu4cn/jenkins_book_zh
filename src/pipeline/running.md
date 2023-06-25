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

**从 Blue Ocean 用户界面重启**
