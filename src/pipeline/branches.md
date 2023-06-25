# 分支与拉取请求

**Branches and Pull Requests**


在 [前面小节](./jenkinsfile.md) 中，我们实现了一个可以检查到源码控制的 Jenkinsfile。本节介绍了 **多分支，Multibranch** 流水线的概念，其建立在 Jenkinsfile 的基础上，提供了 Jenkins 中更多的动态和自动功能。

*在 Jenkins 中创建多分支流水线*


[![Git 下的 Jenkins 多分支流水线教程](https://img.youtube.com/vi/B_2FXWI6CWg/0.jpg)](https://www.youtube.com/watch?v=B_2FXWI6CWg)

视频内容总结：

1. 刚开始使用 Jenkins 时，咱们通常会使用自由风格作业，或者是流水线作业；

2. 在使用流水线作业时，通常是构建一个分支；

3. 在应用开发团队需要咱们构建 git 代码仓库的多个分支时，若对每个分支建立一个 Jenkinsfile，就会浪费时间；

4. 创建构建项目时，选择 “多分支流水线，Multibranch Pipeline”；然后在 “分支源，Branch Sources” 字段，选择 “Git”；

5. 针对公开代码仓库，选择 `https` URI，并无需凭据；

6. 在没有发现 Jenkinsfile 时，会显示 `Does not meet criteria`，“This folder is empty”；

7. Jenkinsfile 是建立多分支构建项目的必要条件；

8. 通过 web 钩子，可以从 GitHub 推送变更到 Jenkins 控制器；

9. 多分支构建项目配置中，有 “扫描 多分支流水线 触发器” 选项；

10. 只能配置多分支构建项目，而无法配置其下的各个分支构建；

11. Jenkins 控制器会检出所有分支；

12. 在 “分支源” 下的 “行为” 字段，可以配置 “发现分支”，配置一些过滤条件，比如 “根据名称过滤（支持正则表达式）”、“根据名称过滤（支持通配符）” 等，过滤时在 “包含” 与 “排除” 字段，可以将包含或排除的分支，用空格分开列出；

13.
