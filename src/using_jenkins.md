# 使用 Jenkins

本章包含了典型 Jenkins 用户（各种技能水平）关于 Jenkins 使用的主题，这些主题不在 Jenkins 核心功能的范围内：管线，Pipeline，和 Blue Ocean。

如果咱们想通过 `Jenkinsfile` 或 Blue Ocean 来创建和配置一个 Pipeline 项目，或者想了解更多关于这些 Jenkins 核心功能的信息，请参考 [Pipeline](src/pipeline.md) 和 [Blue Ocean](src/blue_ocean.md) 各自章节中的相关主题。

如果你是一名 Jenkins 管理员，想知道更多关于管理 Jenkins 节点和实例的信息，请查看 [管理 Jenkins](src/managing.md)。

如果你是系统管理员，想学习如何备份、恢复、维护作为 Jenkins 服务器和节点，请看 [Jenkins 系统管理](src/administration.md)。

如果你是一名 Jenkins 的用户，正在寻找一些故障排除的技巧，请看 [Jenkins 故障排除](src/troubleshooting.md)。

关于这本 Jenkins 用户手册的内容概述，请参见 [用户手册概述](src/Ch00_Overview.md)。

## 使用 `emailext` 时报错问题


```groovy
            emailext body: '''$PROJECT_NAME - Build # $BUILD_NUMBER - FAILURE:
                Check console output at $BUILD_URL to view the results. ${BUILD_LOG, maxLines=100, escapeHtml=false}''', recipientProviders: [developers(), culprits()], subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - FAILURE!', to: 'lenny.peng@xfoss.com, michael.rong@xfoss.com'
```

因为其中有参数 `recipientProviders: [developers(), culprits()]` 而会报出如下错误：

```console
Not sending mail to unregistered user unisko@gmail.com because your SCM claimed this was associated with a user ID ‘unisko' which your security realm does not recognize; you may need changes in your SCM plugin
...
```

解决方法：

Turn on the configuration under **Manage Jenkins** -→ **Configure System** -→ **Extended E-mail Notification** -→ **Allow sending to unregistered users**.


参见：[Not sending mail to unregistered user because your SCM claimed this was associated with a user ID which your security realm does not recognize](https://docs.cloudbees.com/docs/cloudbees-ci-kb/latest/client-and-managed-controllers/not-sending-mail-to-unregistered-user-associated-with-scm)。
