# 使用 Jenkins 代理

Jenkins 的架构是为分布式的构建环境设计的。他允许我们为每个构建项目使用不同的环境，在并行运行作业的多个代理之间平衡工作负荷。

Jenkins 控制器是 Jenkins 安装中的原始节点，the original node。Jenkins 控制器管理 Jenkins 代理并协调他们的工作，包括在代理上调度作业与监控代理。代理可以是使用本地或云计算机连接到 Jenkins 控制器。

代理需要 Java 安装并与 Jenkins 控制器建立网络连接。请观看下面 3 分钟的视频，了解 Jenkins 代理的简要说明。

（这里仅提供该视频的字幕中文翻译。）

*什么是 Jenkins 代理*


```txt
在今天的视频中，我们将

讨论什么是 Jenkins 代理

[音乐]

如果你是新来的，欢迎你，

如果你是新来的，你不知道我是谁，



我的名字是 Darin Pope，我是

cloudbees 的开发者倡导者，您可能

对整个 Jenkins 世界都不熟悉，

并且您已经听过一些短语，

也许是 Jenkins 控制器

和 Jenkins 代理，

现在 Jenkins 代理的目的是什么，

通常代理要么是一台机器，

要么是一台机器 虚拟

机，甚至可能是裸机，或者

你最喜欢的树莓派，

或者它甚至可能只是一个

连接到 Jenkins 控制器

并按照控制器的指示执行任务的容器，



Jenkins 控制器的目的是

管理连接，甚至

可能是 这些代理的工具



有时

正如我们之前所说，代理可以是

裸机或虚拟化的，它甚至可以

是云，

比如 ec2

，甚至可以是 azure

这些类型的实例可以

动态配置并连接到

Jenkins 控制器，基本上

任何类型的机器都可以 运行 java

可以是

Jenkins 控制器的代理 你可能会问

自己为什么我需要一个代理

我有我的 jeekins 控制器我可以

直接在控制器上运行作业

从安全

角度和潜在的

性能角度来看都是一种很好的做法



最好使用代理来完成您的所有

工作，而不是

直接在您的控制器上进行任何类型的构建，因此无论

您实际上是使用 Jenkins

为您的部门

还是为您的整个公司构建应用程序，

总是在以下情况下使用代理 构建你的

工作

如果你有任何问题或意见，

你可以在 cloudbeesdevs 的 twitter 上与我们联系 点击









订阅

按钮，然后按铃，

您会

在 cloudbeast 电视上有新内容时随时收到通知，

感谢收看，我们将

在下一个视频中见到您

```


## 使用 Docker 配置代理

Jenkins 代理可以在物理机 physical machines、虚拟机 virtual machines、Kubernetes 集群和 Docker 镜像中启动。这个小节使用 SSH 将 Docker 的代理连接到 Jenkins（控制器）。


### 环境

要运行本指南，咱们需要一台具有以下功能的机器：

- Java 安装；

- Jenkins 的安装；

- Docker 的安装；

- SSH 密钥对。

> 如果您在安装 Java、Jenkins 和 Docker 方面需要帮助，请访问 [安装 Jenkins](../installation/docker.md) 小节。


### 生成 SSH 密钥对

要生成 SSH 密钥对，咱们必须在咱们能访问的机器上执行一个名为 `ssh-keygen` 的命令行工具。机器可以是：

- 咱们 Jenkins 控制器所运行的机器；

- 宿主机（在使用容器时）；

- 某台咱们运行着代理的机器；

- 甚至咱们的开发机器。


> SSH 密钥对的生成可以在任何操作系统上完成：
>
> - 在 Windows 上，您可以使用任何的 OpenSSH 安装，例如 [Windows OpenSSH](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)、`ssh-keygen` 在 [用于 Windows 的 git](https://gitforwindows.org/)， 或 [Cygwin](https://cygwin.com/) 中均有包含；
> - 在 Unix（Linux、macOS、BSD 等）上，咱们也可以使用任何与咱们系统打包的 OpenSSH 安装。

> 请注意，咱们必须要能够在事后将密钥值复制到咱们的控制器和代理，所以请检查咱们是否能够事先将文件内容复制到剪贴板中。

1. 请在某个终端窗口中运行命令：`ssh-keygen -f ~/.ssh/jenkins_agent_key`；

2. 提供一个与密钥一起使用的口令（可以是空的）；

3. 确认输出看起来像这样：


```bash
$ ssh-keygen -f ~/.ssh/jenkins_agent_key
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/lenny.peng/.ssh/jenkins_agent_key
Your public key has been saved in /home/lenny.peng/.ssh/jenkins_agent_key.pub
The key fingerprint is:
SHA256:1tae8bKe1mcQ38cKPcGj900LgTRyQ9CnDW1oJ8YkDgY lenny.peng@sw-build-01.senscomm.com
The key's randomart image is:
+---[RSA 3072]----+
|     E.o o*oo    |
|      . o..& =   |
|         .* %.   |
|         . + o=  |
|        S o oo.*.|
|       . . .o== *|
|            ++o*+|
|            .+o.=|
|           o+  o |
+----[SHA256]-----+
```


## 创建一个 Jenkins SSH 凭证

1. 前往咱们的 Jenkins 配置面板；

2. 前往主菜单中的 “系统管理” 选项，并点击 “Credentials” 按钮；

![Jenkins 凭据管理入口](../images/credentials-1.png)

3.
