# Linux 下安装 Jenkins

对于数个 Linux 发行版，都有 Jenkins 的安装程序。

- [Debian/Ubuntu]()

- [Fedora]()

- [Red Hat/Alma/Rocky]()


## 前置条件

最低硬件要求：

- 256 MB 运行内存；

- 1 GB 硬盘空间（尽管如果以Docker容器的形式运行Jenkins，建议最小为10GB）。


建议小团队的硬件配置：

- 4 GB 以上的 RAM；

- 50 GB 以上的硬盘空间。


全面的硬件建议：

- 硬件：请参阅 [硬件建议](../scaling/hardware_recommendation.md) 页面。

{{#include ../installation/docker.md:27:33}}


## Debian/Ubuntu

在 Debian 和基于 Debian 的发行版，如 Ubuntu，咱们可以通过 `apt` 安装 Jenkins。

### 长期支持发布版本

[长期支持发布版本，LTS(Long-Term Support) release](https://www.jenkins.io/download/lts/) 每 12 周从常规版本中选出一个作为该时间段的稳定版本。他可以从 [debian-stable apt 仓库](https://pkg.jenkins.io/debian-stable/) 中安装。

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

### 每周发布版本

每周都会有一个新的版本，向用户和插件开发者提供错误修复和功能。他可以从 [debian apt 仓库](https://pkg.jenkins.io/debian/) 中安装。


```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

从 Jenkins 2.335 和 Jenkins 2.332.1 开始，软件包的配置采用了 `systemd`，而不是老式的 System V `init`。参见 [DigitalOcean 社区 `systemd` 教程](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)，以更好地了解 `systemd` 和 `systemctl` 命令的好处。


该软件包的安装将：

- 将 Jenkins 设置为系统启动时启动的守护程序。请运行 `systemctl cat jenkins` 了解更多细节；

- 创建一个 `jenkins` 用户来运行这个服务；

- 将控制台日志输出到 `systemd-journald`。如果咱们正对 Jenkins 进行故障排除，请运行 `journalctl -u jenkins.service`；

- 产生有着比如 `JENKINS_HOME` 等启动配置参数的 `/lib/systemd/system/jenkins.service`；

- 将 Jenkins 设置为监听 `8080` 端口。用咱们的浏览器访问这个端口，开始配置。


> 如果 Jenkins 因为某个端口被使用而无法启动，运行 `systemctl edit jenkins` 并添加以下内容：

```conf
[Service]
Environment="JENKINS_PORT=8081"
```
> 这里选择了 `8081`，但咱们也可以设置另一个可用的端口。


### Java 的安装

Jenkins 需要 Java 才能运行，但某些发行版默认不包含他，并且 [某些 Java 版本与 Jenkins 不兼容](../administration/java_requirements.md)。

咱们可以使用多种 Java 实现。 [OpenJDK](https://openjdk.java.net/) 是目前最流行的，我们将在本指南中使用他。


请使用以下命令更新 Debian apt 软件库，安装 OpenJDK 11，并检查安装情况：

```bash
sudo apt update
sudo apt install openjdk-11-jre
java -version
openjdk version "11.0.12" 2021-07-20
OpenJDK Runtime Environment (build 11.0.12+7-post-Debian-2)
OpenJDK 64-Bit Server VM (build 11.0.12+7-post-Debian-2, mixed mode, sharing)
```

> 为什么使用 `apt` 而不是 `apt-get` 或其他命令？`apt` 命令从 2014 年起就开始使用了。他的命令结构与 `apt-get` 类似，但其创建目的是为了让典型用户有更愉快的体验。简单的软件管理任务，如安装、搜索和删除，用 `apt` 更容易。


## Fedora

咱们可以通过 `dnf` 来安装 Jenkins。咱们需要先把 Jenkins 网站上的 Jenkins 资源库添加到软件包管理器中。


### 长期支持发布版本


[长期支持发布版本，LTS(Long-Term Support) release](https://www.jenkins.io/download/lts/) 每 12 周从常规版本中选出一个作为该时间段的稳定版本。他可以从 [redhat-stable](https://pkg.jenkins.io/redhat-stable/) `yum` 仓库安装。

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo dnf upgrade
# Add required dependencies for the jenkins package
sudo dnf install java-11-openjdk
sudo dnf install jenkins
sudo systemctl daemon-reload
```

### 每周发布版本


每周都会有一个新的版本，向用户和插件开发者提供错误修复和功能。他可以从 [redhat](https://pkg.jenkins.io/redhat/) `yum` 仓库安装。

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io-2023.key
sudo dnf upgrade
# Add required dependencies for the jenkins package
sudo dnf install java-11-openjdk
sudo dnf install jenkins
```

### 启动 Jenkins

咱们可以用以下命令，使 Jenkins 服务在系统启动时启动：

```bash
sudo systemctl enable jenkins
```

咱们可以用以下命令启动 Jenkins 服务：

```bash
sudo systemctl start jenkins
```

咱们可以使用以下命令检查 Jenkins 服务的状态：

```bash
sudo systemctl status jenkins
```

如果一切设置正确，咱们应该看到这样的输出：

```bash
Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2018-11-13 16:19:01 +03; 4min 57s ago
```

> 如果安装了防火墙，则必须将 Jenkins 添加为例外。咱们必须将下面脚本中的 `YOURPORT` 更改为咱们要使用的端口。端口 `8080` 是最常见的。

```bash
YOURPORT=8080
PERM="--permanent"
SERV="$PERM --service=jenkins"

firewall-cmd $PERM --new-service=jenkins
firewall-cmd $SERV --set-short="Jenkins ports"
firewall-cmd $SERV --set-description="Jenkins port exceptions"
firewall-cmd $SERV --add-port=$YOURPORT/tcp
firewall-cmd $PERM --add-service=jenkins
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```


## Red Hat/Alma/Rocky

咱们可以通过 `yum` 在 Red Hat Enterprise Linux、Alma Linux、Rocky Linux、Oracle Linux 和其他基于 Red Hat 的发行版上安装 Jenkins。

咱们需要选择 Jenkins 长期支持版本或 Jenkins 每周版本。

{{#include ./linux.md:119:191}}

{{#include ./docker.md:292:}}


（End）


