# 远程访问 API

Jenkins 为其功能提供了机器可消费的远程访问 API。目前他有三种类型，comes in three flavors：

1. XML

2. 带有 JSONP 支持的 JSON

3. Python


远程访问 API 是以类似 REST 的方式提供的。也就是说，所有特性都没有单一的入口，而是在 `.../api/` URL下提供，其中 `...` 部分是其所作用的数据。

例如，如果咱们的 Jenkins 安装在 `https://ci.xfoss.com`，那么访问 `https://ci.xfoss.com/api/`，将只显示可用的顶层 API 功能 -- 主要是这个 Jenkins 实例下做配置的作业列表。

或者，如果咱们想访问某个特定构建有关的信息，例如 `https://build.xfoss.com/job/demo-job/lastSuccessfulBuild/`，那么前往 `https://build.xfoss.com/job/demo-job/lastSuccessfulBuild/api/`，咱们就会看到那个构建的功能列表。


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


[Jenkins API 客户端](https://rubygems.org/gems/jenkins_api_client) 是一个面向对象的 Ruby 封装器，他消费 Jenkins 的 JSON API，旨在提供对 Jenkins 提供的所有远程 API 的访问。他可以作为 Rubygem 使用，并且可以与作业 Job、节点 Node、视图 View、构建队列 BuildQueue 及系统 System 等相关功能进行交互。目前提供的服务 Services 包括：

- 通过发送 xml 文件或将参数指定为选项来创建作业，有着更多的定制选项，包括源控制、通知等；

- 构建作业（带参数）、停止构建、查询最近构建的细节、获取构建参数等；

- 使用作业名称过滤器、作业状态过滤器列出 Jenkins 中可用的作业；

- 添加/删除下游项目；

- 链式作业，即给定一个项目清单，每个项目作为下游项目加入到前一个项目中；

- 获取渐进式控制台输出，progressive console output；

- 基于用户名/密码的身份验证；

- 有着库中所提供的大量选项的命令行界面；

- 创建、列出视图；

- 将作业添加到视图和从视图中删除作业；

- 添加/删除 Jenkins 代理，查询代理的详细信息；

- 获取构建队列中的任务，以及他们的年龄 age、原因 cause、理由 reason、预计用时 ETA、ID、参数和更多信息；

- 静默关机 quiet down，取消静默关机 canel quiet down，安全重启 safe restart，强制重启 force restart，以及等待重启后 Jenkins 变得可用等；

- 能够列出已安装/可用的插件、获取有关插件的信息、安装/卸载插件以及使用插件进行更多操作。

该项目源代码在 [这里](https://github.com/arangamani/jenkins_api_client)。


## Java API 包装器

**Java API wrappers**

[jenkins-rest](https://github.com/cdancy/jenkins-rest) 库是一个面向对象的 Java 项目，他以编程方式提供对 Jenkins REST API 的访问，以获得 Jenkins 提供的一些远程 API。他是使用 [jclouds 工具包](https://jclouds.apache.org/) 建立的，可以很容易地扩展到支持更多的 REST 端点。他的功能集不断发展，用户被邀请通过 pull-requests 贡献新的端点。在目前的状态下，通过这个库可以提交一个作业，跟踪其在队列中的进度，监控其执行情况，直到完成，并获得构建状态。目前提供的服务包括：

- 端点定义（属性或环境变量）；

- 身份验证（经由属性或环境变量的基本和 API 令牌）；

- Crumb 发行者支持（自动检查 crumbs，此特性与前面提到的 [CRSF 防护](#csrf-防护) 有关）；

- 文件夹支持；

- 作业 API（构建 `build`、构建信息 `buildInfo`、带参数构建 `buildWithParameters`、配置 `config`、创建 `create`、删除 `delete`、描述 `description`、禁用 `disable`、启用 `enable`、作业信息 `jobInfo`、上一次构建编号 `lastBuildNumber`、上一次构建时间戳 `lastBuildTimestamp` 及渐进式文本 `progressiveText` 等）；

- 插件管理器 API（安装必要插件 `installNecessaryPlugins`, 列出当前的插件等）；

- 队列 API（取消、列出队列项目、查询队列项目）；

- 统计 API（总体负荷）；

- 系统 API（系统信息 `systemInfo`）。

该项目可能会迅速发展，这份清单仅在撰写日期前是准确的。


## 检测 Jenkins 版本

要检查 Jenkins 的版本，请加载首页或任何 `.../api/*` 页面并检查 `X-Jenkins` 响应头。这包含了 Jenkins 的版本号，如 `2.401.1`，这也是检查一个 URL 是否是 Jenkins URL 的好方法。

```bash
$ curl -o - -I https://ci.xfoss.com/
HTTP/1.1 403 Forbidden
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 06 Jun 2023 06:39:58 GMT
Content-Type: text/html;charset=utf-8
Content-Length: 541
Connection: keep-alive
X-Content-Type-Options: nosniff
Set-Cookie: JSESSIONID.a2a3cfcf=node0j5t0jvptqotv1dgrt8kuass8h20.node0; Path=/; HttpOnly
Expires: Thu, 01 Jan 1970 00:00:00 GMT
X-Hudson: 1.395
X-Jenkins: 2.401.1
X-Jenkins-Session: 28a2145a
```


（End）


