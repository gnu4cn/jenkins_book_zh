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


相对路径以正斜杠之外的东西开头，并指向与当前项目有关的另一项目。例如，假设咱们有以下绝对路径的项目：

```pipeline
/thatproject
/folder/someproject
/folder/subfolder/myproject
/folder/subfolder/anotherproject
```

在 `/folder/subfolder/myproject` 的某个管道脚本， Pipeline Project，中，咱们可以使用下面这个相对路径引用 `/folder/subfolder/anotherproject`：


```pipeline
anotherproject
```

并可以用下面这个相对路径来引用 `/folder/someproject`，其中 `..` 表示在父文件夹中查找：

```pipeline
../someproject
```

并可以使用下面的相对路径，引用 `/thatproject`：

```pipeline
../../thatproject
```


## 引用项目内部的组件

某些类型的项目 -- 如 Maven 项目、Matrix 项目和 Multibranch 项目 -- 有着一些子组件。咱们可以用以下方式来引用这些子组件：


### Maven 项目

咱们可以引用整个 Maven 项目：

```pipleline
mymavenproject
```

或 Maven 项目里的某个组：

```pipeline
mymavenproject/my.group
```

或到某个特定模块的引用：

```pipeline
mymavenproject/my.group$MyModule
```


### Matrix 项目

咱们可以引用某个 Matrix 项目的所有配置：

```pipeline
mymatrixproject
```

或引用受某个轴线所限制的特定配置，a particular configuration, restricted by a axis：

```pipeline
mymatrixproject/someaxis=somevalue
```

或受多个轴线的限制：

```pipeline
mymatrixproject/someaxis=somevalue,anotheraxis=anothervalue
```


### 多分支的管线

**Multibranch Pipelines**

咱们可以引用某个特定分支：

```pipeline
mymultibranchproject/mybranch
```

## 名字编码

路径中的特殊字符应该用 URL 编码。例如，如果咱们的 Multibranch Pipeline 有一个带有斜线的分支（ `feature/myfeature` ），那么就要用 `%2F` 替换斜线：

```pipeline
mymultibranchproject/feature%2Fmyfeature
```


## 对于 Jenkins 和 Jenkins 插件的开发者

请参阅 [`Jenkins::getItem()`](https://javadoc.jenkins.io/jenkins/model/Jenkins.html#getItem-java.lang.String-hudson.model.ItemGroup-) 函数。
