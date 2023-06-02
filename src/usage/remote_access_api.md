# 远程访问 API

Jenkins 为其功能提供了机器可消费的远程访问 API。目前他有三种类型，comes in three flavors：

1. XML

2. 带有 JSONP 支持的 JSON

3. Python


远程访问 API 是以类似 REST 的方式提供的。也就是说，所有特性都没有单一的入口，而是在 `.../api/` URL下提供，其中 `...` 部分是其所作用的数据。

例如，如果咱们的 Jenkins 安装在 `https://ci.xfoss.com`，那么访问 `https://ci.xfoss.com/api/`，将只显示可用的顶层 API 功能 -- 主要是这个 Jenkins 实例下做配置的作业列表。

或者，如果咱们想访问某个特定构建有关的信息，例如 `https://build.senscomm.com/job/demo-job/lastSuccessfulBuild/`，那么前往 `https://build.senscomm.com/job/demo-job/lastSuccessfulBuild/api/`，咱们就会看到那个构建的功能列表。


## 咱们能用他做什么？

远程 API 可以用来做下面这些事情：


1. 从Jenkins中获取用于程序性消费的信息；

2. 触发一个新的构建；

3. 创建/拷贝作业。


## 提交作业


### 不带参数的作业


咱们只需要对 `JENKINS_URL/job/JOBNAME/build` 执行一次 HTTP POST。


### 带参数的作业


简单示例 -- 发送“字符串参数”：

```bash
curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:TOKEN \
  --data id=123 --data verbosity=high
```

另一个示例 -- 发送一个 "文件参数"：


```bash
curl JENKINS_URL/job/JOB_NAME/buildWithParameters \
  --user USER:PASSWORD \
  --form FILE_LOCATION_AS_SET_IN_JENKINS=@PATH_TO_FILE
```

符号 `@` 在这个例子中很重要。另外，文件的路径是绝对路径。为了使这个命令工作，咱们需要将咱们的 Jenkins 配置为接受一个文件参数，并将 Jenkins 作业配置中的 **文件位置，File location** 字段与 `--form` 选项中的键相匹配。


## 远程 API 和安全性

当咱们的 Jenkins 是受保护的时，咱们可以使用 HTTP BASIC 身份验证来验证远程 API 请求。有关详细信息，请参阅 [验证脚本客户端，Authenticating scripted clients](../administration/authenticating_scripted_clients.md)。


### CSRF 防护

**注意**：在 CSRF 保护方面，首选 API 令牌而不是 Crumb 令牌，API tokens are preferred instead of crumbs for CSRF protection。


> 什么是 Crumb 令牌？
>
> Crumb 是用来减少 CSRF 攻击的，他使用一个随机的唯一令牌，在服务器端进行验证。只要你想防止恶意代码执行由 HTTP 请求执行的系统命令，就可以使用 Crumb。
> >https://hapi.dev/module/crumb/api/?v=9.0.1


