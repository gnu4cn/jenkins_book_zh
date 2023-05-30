# 使用凭据

**Using Credentials**

有许多第三方网站和应用程序可以与 Jenkins 互动，例如，工件库（artiface repositories）、基于云的存储系统和服务等等。

此类应用程序的系统管理员可以在应用程序中配置凭据，以供 Jenkins 专门使用。这通常是为了 “锁定（lock down）” 应用程序中对 Jenkins 可用的功能区域，通常是通过对这些凭证应用访问控制实现的。一旦 Jenkins 管理员（即管理 Jenkins 站点的 Jenkins 用户）在 Jenkins 中添加/配置了这些凭证，这些凭证就可以被 Pipeline 项目用来与这些第三方应用程序交互。

**注意**：本页面和相关页面上描述的 Jenkins 凭据功能均是由 [凭证绑定插件](https://plugins.jenkins.io/credentials-binding) 提供的。

可以使用存储在 Jenkins 中的凭据的地方：

- 整个 Jenkins 中任何适用之处（即全局凭证）；

- 为某个特定的 Pipeline 项目/条目（在 [使用 `Jenkinsfile`](../pipeline/jenkinsfile.md) 的 [处理凭据](../pipeline/jenkinsfile.md#handling-credentials) 小节中阅读更多关于这一点）所使用；

- 由某名特定 Jenkins 用户（如 [在 Blue Ocean 中创建的管道项目](../blue_ocean/creating_pipelines.md)）所使用。


Jenkins 可以存储以下类型的凭据：

- **密文，secret text** - 一个令牌，如 API 令牌（如 GitHub 的个人访问令牌）；

- **用户名与口令，username and password** - 可以作为独立的组件来处理，也可以作为一个冒号分隔的字符串来处理，其格式为 `username:password`（在[处理凭据](../pipeline/jenkinsfile.md#handling-credentials) 中阅读更多关于这个问题）;

- **秘密文件，secret file** - 这本质上是文件中的秘密内容；

- **带有私钥的 SSH 用户名，SSH Username with private key** - 一个 [SSH 公钥/私钥对](http://www.snailbook.com/protocols.html)；

- **证书，certificate** - 一个 [PKCS#12 证书文件](https://tools.ietf.org/html/rfc7292) 与可选的密码，或

- **Docker 主机证书认证，Docker Host Certificate Authentication** 凭据。
