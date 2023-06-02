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
>
> 参见：https://hapi.dev/module/crumb/api/?v=9.0.1


### XPath 选取

XML API 通过使用查询参数 `xpath`，支持通过 XPath 进行选择。这对于在 XML 操作繁琐的环境中提取信息是很方便的（比如 shell 脚本）。关于如何使用这个的示例，请参阅 [issue #626](https://issues.jenkins.io/browse/JENKINS-626)。

请查看咱们 Jenkins 服务器上的 `../api/` 了解更多更新的细节。


### XPath 排除


与上面的 `xpath` 查询参数类似，咱们可以使用（可能是多个）`exclude` 查询模式来从结果的 XML 中排除节点。所有与指定 XPath 相匹配的节点都将被从 XML 中删除。


## 深度控制

有时候，远程 API 在一次调用中并不能给咱们足够的信息。例如，如果咱们想找到某个给定视图的最后一次成功构建，就会发现调用该视图的远程 API 不会给到咱们这个信息，咱们必须递归调用每个项目的远程 API。深度控制解决了这个问题。深度控制从根本上与 Jenkins 的数据模型有关。

Jenkins 内部维护的数据模型可被认为是一个大的树状结构，当咱们进行远程 API 调用时，咱们得到的是其中的一个小子树。该子树以咱们进行远程 API 调用的对象为根，子树在超过一定深度时被切断，以避免返回过多的数据。咱们可以通过指定深度查询参数来调整这种截断行为。当咱们指定一个正的深度值时，子树的截断就会晚一些。

所以最终结果是，如果咱们指定了一个更大的深度值，咱们会看到远程 API 现在会返回更多的数据。由于算法的原因，这种工作方式使更大的深度值所返回的数据，会包括更小的深度值所返回的所有数据。

请参阅 Jenkins 服务器上的 `.../api/` 以获取更多最新详细信息。


## Python API 封装器


[JenkinsAPI](https://pypi.python.org/pypi/jenkinsapi)、[Python-Jenkins](https://pypi.python.org/pypi/jenkinsapi)、[api4jenkins](https://pypi.org/project/api4jenkins/)、[aiojenkins](https://pypi.org/project/aiojenkins/) 是 Python REST API 的面向对象的 python 封装，旨在提供一种更传统的 pythonic 方式来控制 Jenkins 服务器。这提供了一个更高层次的 API，包含一些便利的功能。目前提供的服务包括：

- 查询某个已完成构建的测试结果；

- 获取代表某项作业最新构建的对象;

- 通过简单的标准搜索工件，artifacts；

- 阻塞直到作业完成；

- 将工件，artifacts，安装到定制指定的目录结构；

- 对 Jenkins 实例的身份验证支持；

- 根据 subversion 修订搜索构建的能力；

- 添加/删除/查询 Jenkins 代理的能力。


## Ruby API 封装器


[Jenkins API 客户端](https://rubygems.org/gems/jenkins_api_client) 是一个面向对象的 Ruby 封装器，他消费 Jenkins 的 JSON API，旨在提供对 Jenkins 提供的所有远程 API 的访问。
