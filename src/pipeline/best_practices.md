# 流水线最佳实践

**Pipeline Best Practices**


本指南提供了有关流水线的一小部分最佳实践，并指出了一些最常见的错误。

其目的是引导流水线作者和维护者们，朝着能得到更好的流水线执行的模式，而远离他们可能没有意识到的陷阱。本指南不会详尽无遗地列出所有可能的流水线最佳实践，而是提供了有助于追踪常见实践的一些具体示例。请将本指南用作“要这样做”的一般指南，而不是极为详细的“如何做”的指南。

本指南按领域、准则编排，然后列出具体实例。


## 一般情况

**General**

### 确保将流水线中的 Groovy 代码用作胶水语言

**Making sure to use Groovy code in Pipelines as glue**


要使用 Groovy 代码连接一系列操作，而不是将其作为流水线的主要功能。换句话说，与其依赖流水线功能（Groovy 或流水线步骤）来驱动构建过程，不如使用单一步骤（如 `sh`）来完成构建的多个部分。随着流水线复杂性的增加（Groovy 代码量、用到的步骤数量等），就需要更多的控制器资源（CPU、内存、存储）。请将流水线视为完成构建的工具，而不是构建的核心。

示例：使用单个 Maven 构建步骤，来驱动构建/测试/部署等过程。


### 在 Jenkins 流水线中运行 shell 脚本

在 Jenkins 流水线中使用 shell 脚本，可通过将多个步骤合并到单个阶段中，来简化构建过程。shell 脚本还允许用户添加或更新命令，而无需单独修改每个步骤或阶段。

下面这个视频，回顾了在 Jenkins 流水线中使用 shell 脚本，以及其所提供的好处。

[![怎样在 Jenkins 流水线中运行 shell 脚本](https://img.youtube.com/vi/mbeQWBNaNKQ/0.jpg)](https://www.youtube.com/watch?v=mbeQWBNaNKQ)


### 避免在流水线中使用复杂的 Groovy 代码

**Avoiding complex Groovy code in Pipelines**


对于流水线，Groovy 代码 **始终** 是在控制器上执行，这意味着用的是控制器资源（内存和 CPU）。因此，减少流水线执行的 Groovy 代码量（包括流水线中导入的类中调用的任何方法）至关重要。以下是要避免用到的 Groovy 方法的最常见示例：


1. **`JsonSlurper`**：该函数（以及其他类似函数，如 `XmlSlurper` 或 `readFile`）可用于读取磁盘上的文件，将文件中的数据解析为 JSON 对象，然后使用 `JsonSlurper().parseText(readFile("$LOCAL_FILE"))` 等命令将该对象注入流水线。该命令会两次将本地文件，加载到控制器的内存中，如果文件非常大或命令执行频繁，就会占用大量内存。

    a. 解决方案：与其使用 `JsonSlurper`，不如使用 shell 步骤并返回标准输出。这种 shell 将如下所示： `def JsonReturn = sh label: '', returnStdout: true, script： echo "$LOCAL_FILE" | jq "$PARSING_QUERY"'`。这将使用 Jenins 代理的资源来读取文件，而 `$PARSING_QUERY` 则将有助于将文件解析成较小的大小。

2. **`HttpRequest`**： 该命令通常用于从外部源，抓取数据并将其存储到变量中。这种做法并不理想，因为请求不仅直接从控制器来（如果控制器没有加载证书，HTTPS 请求等可能会产生错误结果），而且该请求的响应会被存储两次。

    a. 解决方案：使用 shell 步骤从 Jenkins 代理执行 HTTP 请求，例如酌情使用 `curl` 或 `wget` 等工具。如果请求结果必须要在流水线的稍后部分出现，就要尽量在代理端过滤结果，以便只向 Jenkins 控制器传回所需的最少信息。


### 减少重复的相似流水线步骤

**Reducing repitition of similar Pipeline steps**


尽可能地将多个流水线步骤，合并为单一步骤，以减少流水线执行引擎本身造成的开销。例如，如果连续运行三个 shell 步骤，每个步骤都必须启动和停止，这就需要创建和清理 Jenkins 代理与控制器上的连接和资源。但是，如果将所有命令放在一个 shell 步骤中，则只需启动和停止一个步骤。

示例：与其创建一系列 `echo` 或 `sh` 步骤，不如将他们合并为一个步骤或脚本。


### 避免对 `Jenkins.getInstance` 的调用

**Avoiding calls to `Jenkins.getInstance`**


在流水线或共享库中，使用 `Jenkins.instance` 或其访问器方法，表明在该流水线/共享库中存在代码滥用。在无沙箱的共享库中使用 Jenkins API，意味着该共享库既是一个共享库，也是某种 Jenkins 插件。在流水线种与 Jenkins API 交互时，咱们需要非常小心，以避免严重的安全与性能问题。如果咱们必须在构建中使用 Jenkins API，建议的方法是使用流水线的步骤 API，Pipeline's Step API, 以 Java 语言创建一个对咱们打算访问的 Jenkins API 进行安全封装的最小插件。直接从沙箱化的 `Jenkinsfile` 中使用 Jenkins API，意味着咱们很可能必须将一些方法列入白名单，这些方法允许任何能修改流水线的人，绕过沙箱保护，这是一个重大的安全风险。白名单方法以系统用户身份运行，拥有总体管理员权限，这可能导致开发人员拥有比预期更高的权限。


解决方案：最好的解决方案是绕过正在进行的调用，但如果必须调用，则最好实现一个能够收集所需数据的 Jenkins 插件。


### 清理旧的 Jenkins 构建

**Cleaning up old Jenkins builds**



