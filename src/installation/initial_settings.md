# 初始设置

大多数 Jenkins 配置的修改可以通过 Jenkins 用户界面或通过 [配置即代码插件, configuration as code plugin](https://plugins.jenkins.io/configuration-as-code) 进行。有一些配置值只能在 Jenkins 启动时修改。本节描述了这些设置以及咱们使用他们。


## Jenkins 的参数

Jenkins 的初始化也可以由作为参数传递的运行时参数控制。命令行参数可以调整网络、安全、监控和其他设置。

### 网络参数

Jenkins网络配置一般由命令行参数控制。网络配置的参数有：

| **命令行参数** | **说明**
| :-- | :-- |
| `--httpPort=$HTTP_PORT` | 使用标准 `http` 协议在端口 `$HTTP_PORT` 上运行 Jenkins 监听器。默认是 `8080` 端口。要禁用（因为咱们使用着 `https`），就使用端口 `-1`。 这个选项不影响在 Jenkins 逻辑（UI、入站的 agent 文件等）中生成的根 URL。他是由全局配置中指定的 Jenkins URL 定义的。 |
| `--httpListenAddress=$HTTP_HOST` | 将 Jenkins 绑定到 `$HTTP_HOST` 表示的 IP 地址。默认是 `0.0.0.0` - 即监听所有可用接口。例如，要只监听来自 `localhost` 的请求，咱们可以使用： `--httpListenAddress=127.0.0.1` |
| `--httpsPort=$HTTPS_PORT` | 使用 HTTPS 协议，端口为 `$HTTPS_PORT`。这个选项不影响在 Jenkins 逻辑（UI，入站 agent 文件等）中生成的根 URL。他是由全局配置中指定的 Jenkins URL 定义的。 |
| `--httpsListenAddress=$HTTPS_HOST` | 绑定 Jenkins 以监听 `$HTTPS_HOST` 所表示 IP 地址上的 HTTPS 请求。 |
| `--http2Port=$HTTP_PORT` | 使用 HTTP/2 协议，端口为 `$HTTP_PORT`。这个选项不影响在 Jenkins 逻辑（UI、入站 agent 文件等）中生成的根 URL。他是由全局配置中指定的 Jenkins URL 定义的。 |
| `--http2ListenAddress=$HTTPS_HOST` | 绑定 Jenkins 以监听 `$HTTPS_HOST` 所表示 IP 地址上的 HTTP/2 请求。 |
| `--prefix=$PREFIX` | 运行 Jenkins，在 URL 的末尾包括 `$PREFIX`。例如，设置 `--prefix=/jenkins`，使 Jenkins 可以在 `http://myServer:8080/jenkins` 处访问到。 |
| `--sessionTimeout=$SESSION_TIMEOUT` | 设置 `http` 会话超时值为 `$SESSION_TIMEOUT` 分钟。默认为 webapp 指定的值，然后是 60 分钟。 |

*表 1 - Jenkins 网络方面的命令行参数*


