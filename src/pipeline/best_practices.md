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


作为 Jenkins 管理员，删除旧的或不需要的构建，可保持 Jenkins 控制器高效运行。如果不删除旧的构建，就会减少用于当前相关构建的资源。本视频介绍了在单个流水线作业中，使用 `buildDiscarder` 指令的情况。视频还回顾了保留特定历史构建的过程。

*如何清理早先的 Jenkins 构建*

[![如何清理早先的 Jenkins 构建](https://img.youtube.com/vi/_Z7BlaTTGlo/0.jpg)](https://www.youtube.com/watch?v=_Z7BlaTTGlo)


## 使用共享库

**Using Shared Libraries**


### 请勿覆盖内置管道步骤

**Do not override built-in Pipeline steps**


尽可能避免定制/重写流水线步骤。所谓覆盖内建流水线步骤，是指使用共享库覆盖标准的流水线 API（如 `sh` 或 `timeout` 等）的过程。这一过程非常危险，因为流水线 API 随时可能发生变化，从而导致自定义代码崩溃，或给出与预期不同的结果。当自定义代码因流水线 API 更改而崩溃时，故障排除就会变得很困难，因为即使自定义代码没有更改，在 API 更新后也可能无法正常工作。因此，即使自定义代码没有更改，也并不意味着在 API 更新后它会保持同样的工作状态。最后，由于这些步骤在整个流水线中无处不在，如果编码不正确/不充分，结果可能会给 Jenkins 带来灾难性的影响。


### 避免使用大型全局变量声明文件

**Avoiding large global variable declaration files**


因为无论是否需要这些变量，每个流水线都要加载该文件，所以变量声明文件过大可能会占用大量内存，却几乎没有任何益处。建议创建只包含与当前执行相关变量的小型变量文件。


### 避免使用超大型共享库

**Avoiding very large shared libraries**

在流水线中使用大型共享库，就需要在流水线开始之前检出一个非常大的文件，并且对于当前正在执行的每个作业，都要加载同一共享库，这可能会导致增加内存开销，以及较慢的执行时间。


## 解答其他常见问题

**Answering additional FAQs**

### 处理流水线中的并发问题

**Dealing with Concurrency in Pipelines**

尽量不要在多个流水线执行，或多个不同流水线之间共享工作空间。这种做法可能导致每个流水线内的文件意外修改或工作空间重命名。


理想情况下，共享的卷/磁盘，应该挂载在某个独立位置，文件应从那个位置，复制到当前的工作空间。然后，在构建完成时，如果有更新，可以将文件复制回来。


要在不同的容器中进行构建，这些容器可以从头开始创建所需的资源（云类型的 Jenkins 代理就非常适用于这一点）。构建这些容器，将确保每次构建过程都从头开始，并且易于重复。如果构建容器无法使用，则可以在流水线上禁用并发，或者使用 [可上锁资源插件，Lockable Resources Plugin](https://plugins.jenkins.io/lockable-resources)，在运行时锁定工作空间，以防其他构建在锁定期间使用他。**警告**：如果这些资源被随意锁定，那么在等待资源时，禁用并发，或在运行时锁定工作空间，则可能会导致流水线被阻塞。


**另外，请注意，这两种方法都比为每个作业使用唯一资源，会更慢地产生构建结果**。


## 避免使用 `NotSerializableException`


流水线代码会经过CPS转换，以便在 Jenkins 重新启动后，能够恢复流水线的执行。也就是说，当流水线运行咱们的脚本时，咱们可以关闭 Jenkins 或失去与代理的连接。当其恢复运行时，Jenkins 会记住其正在进行的工作，咱们的流水线脚本将继续执行，就好像从未中断过一样。一种称为 [“继续传递式（continuation-passing style, CPS）”](https://en.wikipedia.org/wiki/Continuation-passing_style) 执行的技巧，在恢复流水线方面起着关键作用。然而，由于 CPS 转换的结果，一些 Groovy 表达式会无法正确工作。


在底层，CPS 依赖于能够将流水线的当前状态，以及待执行的其余部分进行序列化。这意味着在流水线中使用不可序列化的对象，将在流水线尝试持久化其状态时触发 `NotSerializableException` 异常。


请参阅 [流水线 CPS 方法不匹配](./CPS_method_mismatches.md) 小节，了解更多详情以及可能存在问题的一些示例。


以下内容将介绍确保流水线能够按预期运行的技巧。


### 确保持久化变量可序列化

**Ensure Persisted Variables Are Serializable**


在序列化过程中，本地变量会作为流水线状态的一部分而被捕获。这意味着，如果在流水线执行过程中，将不可序列化的对象存储到变量中，就会产生 `NotSerializableException` 异常。


### 请勿将不可序列化的对象赋值给变量

**Do not assign non-serializable objects to variables**


利用不可序列化的对象的一种策略，便是始终在需要时才推断出他们的值，而不是计算出他们的值并将该值存储在变量中，one strategy to make use of non-serializable objects to always infer their value "just-in-time" instead of calculating their value and storing that value in a variable。


### 使用 `@NonCPS`

如果有必要，咱们可以使用 `@NonCPS` 注解，来禁用特定方法的 CPS 转换，若对该方法进行 CPS 转换，其主体将无法正确执行时。需要注意的是，这也意味着这个 Groovy 函数必须整个重启，因为其没有经过转换。

> 一些异步流水线步骤（如 `sh` 和 `sleep`）总是经过了 CPS 转换，而不得在注解为 `@NonCPS` 的方法中使用。一般来说，应避免在注解为 `@NonCPS` 的方法中，使用流水线步骤。



### 流水线持久性

**Pipeline Durability**

值得注意的是，改变流水线的持久性，可能会导致原本会抛出 `NotSerializableException` 异常的情况不再抛出。这是因为通过 `PERFORMANCE_OPTIMIZED` 降低流水线的持久性，意味着流水线的当前状态不会被频繁持久化。因此，流水线不会尝试序列化不可序列化的值，因此不会抛出异常。


> **注意**：这条说明旨在告知用户，此行为的根本原因。不建议将流水线的持久性设置，纯粹为了避免可序列化问题而设置为性能优化，Performance Optimized。


（End）


