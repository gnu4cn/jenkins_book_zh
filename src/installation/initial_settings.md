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


### 杂项参数

其他 Jenkins 初始化选项也由命令行参数控制。杂项配置参数有：

| **命令行参数** | **说明** |
| :-- | :-- |
| `--argumentsRealm.passwd.$USER=$PASS` | 为用户 `$USER` 指定密码。如果启用了 Jenkins 的安全功能，咱们必须以拥有 *admin* 角色的用户身份登录来配置 Jenkins。 |
| `--argumentsRealm.roles.$USER=admin` | 指定用户 `$USER` 为管理员角色。即使在 Jenkins 中启用了安全功能，该用户也可以配置 Jenkins。更多信息请参考 [加固 Jenkins](../security/seuring_jenkins.md)。 |
| `--paramsFromStdIn` | 从标准输入（`stdin`）读取参数。当参数通过命令行传递时，只要进程继续运行，就可以用 Unix 的 `ps(1)` 或 Windows 的 Process Explorer 查看他们。当传递敏感参数如 `--httpsKeyStorePassword` 时，这是不可取的。有了 `--paramsFromStdIn` 参数，咱们就可以用 <br> <code>echo '--httpPort=-1 --httpsPort=443 --httpsKeyStore=path/to/keystore --httpsKeyStorePassword=keystorePassword' &#124; java -jar jenkins.war --paramsFromStdIn</code> <br> 取代 <br> <code>java -jar jenkins.war --httpPort=-1 --httpsPort=443 --httpsKeyStore=path/to/keystore --httpsKeyStorePassword=keystorePassword</code>。 |
| `--useJmx` | 启用 [Jetty Java 管理扩展（JMX）](https://www.eclipse.org/jetty/documentation/current/#jmx-chapter) |

*表 2 - Jenkins 杂项命令行参数*

### Jenkins 的属性

一些 Jenkins 的行为是用 Java 属性配置的。Java 属性是由启动 Jenkins 的命令行设置的。属性分配使用 `-DsomeName=someValue` 的形式，将值 `someValue` 分配给名为 `someName` 的属性。例如，如果要给属性 `testName` 赋值为 `true`，命令行参数为 `-DtestName=true`。

更多信息请参考 [Jenkins 属性](../managing/features_controlled_with_system_properties.md) 的详细列表。


## 配置 HTTP

### 使用现有证书的 HTTPS


如果咱们正在使用内置的 Winstone 服务器设置 Jenkins，并想使用现有的 HTTPS 证书：

```bash
--httpPort=-1 \
--httpsPort=443 \
--httpsKeyStore=path/to/keystore \
--httpsKeyStorePassword=keystorePassword
```


### 使用 HTTP/2

[HTTP/2 协议](https://tools.ietf.org/html/rfc7540) 允许网络服务器通过管道化请求、多路复用请求来减少加密连接的延迟，并允许服务器在某些情况下，在收到客户对数据的请求之前进行推送。Jenkins 使用的 Jetty 服务器通过添加应用层协议协商 (Application-Layer Protocol Negotiation, ALPN) 的 TLS 扩展来支持 HTTP/2。

> 即使没有设置 HTTPS 端口，启用 HTTP/2 也会隐式地启用 TLS，从使用 Winstone 5.23 的 Jenkins 2.339 开始，咱们还必须指定一个 HTTPS 密钥存储文件。


```bash
--httpPort=-1 \
--http2Port=9090 \
--httpsKeyStore=path/to/keystore \
--httpsKeyStorePassword=keystorePassword
```

### Windows 下的 HTTPS 证书问题

<略>, 请参考 [HTTPS certificates with Windows](https://www.jenkins.io/doc/book/installing/initial-settings/#https-certificates-with-windows)。


（End）


