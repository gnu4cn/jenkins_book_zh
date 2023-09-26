# 流水线即代码

**Pipeline as Code**

*流水线即代码，Pipeline as Code* 这一小节，描述了允许 Jenkins 用户使用源码仓库中存储和版本化的代码，定义流水线式作业流程的一组功能。这些功能允许 Jenkins 发现、管理并运行多个源码仓库与多个分支的作业，从而消除手动创建和管理作业的需要。

要用上 *流水线即代码* 特性，项目必须在仓库根目录下，包含一个名为 `Jenkinsfile` 的文件，其中包含一个 “流水线脚本，Pipeline script”。


此外，还需要在 Jenkins 中配置以下的一个启用作业：


- *多分支流水线，Multibranch Pipeline*：自动构建 *单一* 代码仓库的多个分支；

- *组织文件夹，Organization Folders*：扫描 **GitHub 组织** 或 **Bitbucket 团队**，以发现组织的某个代码仓库，自动为组织/团队创建出受管理的 *多分支流水线* 作业；

- *流水线，Pipeline*： 常规管道作业在指定管道时有一个 "使用 SCM "选项。


