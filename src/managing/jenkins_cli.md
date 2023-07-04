# Jenkins 命令行界面

**Jenkins CLI**


Jenkins 有个内置的命令行界面，允许用户及管理员从脚本或 shell 环境访问 Jenkins。对于常规任务、批量更新、故障排除等的脚本编写很方便。

可以通过 SSH 或使用 Jenkins CLI 客户端（随 Jenkins 分发的 `.jar` 文件）访问命令行界面。

> 此文档假设 Jenkins 2.54 或更新版本。旧版本的 CLI 客户端被认为是不安全的，不应使用。
>
> 当使用服务器和客户端 2.217 或更新版本时，可支持 WebSocket。


## 通过 SSH 使用命令行界面

**Using the CLI over SSH**


在新的 Jenkins 安装中，SSH 服务默认是禁用的。管理员可以选择设置一个特定的端口或要求 Jenkins 在 [全局安全配置](../security/managing_security.md#tcp-端口) 页面中选择某个随机端口。为了确定出随机分配的 SSH 端口，就要检查 Jenkins URL 上返回的头文件，例如：


```bash
$ curl -Lv https://ci.senscomm.com/login 2>&1 | grep -i 'x-ssh-endpoint'
< X-SSH-Endpoint: ci.senscomm.com:32222
```


