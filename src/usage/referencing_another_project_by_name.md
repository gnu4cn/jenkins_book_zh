# 通过名字引用另一项目

在整个 Jenkins 的许多地方，咱们都可以用名字来指代另一项目/作业。例如，在某个管线脚本，a Pipeline Script，中，咱们可能想从另一项目 [拷贝构件，copy artificats](https://plugins.jenkins.io/copyartifact/)：

```pipeline
copyArtifacts projectName: 'myproject'
```

如果咱们的目标项目名称是简单的字母数字，alphanumeric，并且是一个没有子项目的简单项目，并且在整个 Jenkins 实例中具有唯一的名称，这就是咱们需要做的全部。请继续阅读，了解更复杂的情况......


## 区分同名的多个项目

如果咱们使用着 [文件夹插件](https://plugins.jenkins.io/cloudbees-folder)，并且咱们有着多个位于不同的文件夹中的同名项目，那么咱们可以使用路径来区分他们，类似于 Unix 文件系统的路径。有两种类型的路径：


### 绝对路径

绝对路径以正斜杠（`/`）开始，并通过描述完整的路径来引用某个项目，以便从咱们的 Jenkins 实例的主页导航到该项目。例如，在咱们的 Jenkins 实例根部引用某个项目：

```pipeline
/myproject
```

或者，引用一个子文件夹中的项目：

```pipeline
/myfolder/myproject
```


### 相对路径



