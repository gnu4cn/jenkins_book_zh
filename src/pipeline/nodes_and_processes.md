下面这个插件提供了通过与管道兼容的步骤而可用的功能。请在 [Pipeline 语法](./syntax.md) 的 [步骤](./syntax.md#步骤) 小节中，阅读更多关于如何将步骤整合到咱们管道中的信息。

对于其他此类插件的清单，请参阅 [Pipeline 步骤参考](https://www.jenkins.io/doc/pipeline/steps/) 页面。


# Pipeline：节点与进程

**Pipeline: Nodes and Processes**

[在插件站点查看此插件](https://plugins.jenkins.io/workflow-durable-task-step/)


## `bat`：Windows 批处理脚本


- `script: String`

执行一个批处理脚本。允许多行。当使用 `returnStdout` 开关时，咱们或许希望用 `@` 作为前缀，以免命令本身被包含在输出中。

- `encoding: String` (可选项)

进程输出的编码。在 `returnStdout` 的情况下，会应用到该步骤的返回值；否则，或始终用于标准错误，控制了文本如何被复制到构建日志。如果没有指定，则使用运行该步骤节点的系统默认编码。如果有任何进程输出可能包括非 ASCII 字符的预期，那么最好明确指定编码。例如，如果咱们知道一个给定的进程将产生 UTF-8，但将在一个具有不同系统编码的节点上运行（通常是 Windows，因为每个 Linux 发行版都默认为 UTF-8 很长时间了），咱们就可以通过指定：`encoding: 'UTF-8'` 来确保正确的输出。

- `label: String` (可选项)

在 Pipeline 步骤视图，及步骤的 Blue Ocean 细节中，而非步骤类型中显示的标签。因此，视图就更有意义，而且是针对领域的，而不是技术性的。


- `returnStatus: boolean` (可选项)

通常情况下，以非零状态码退出的脚本将导致该步骤以某个异常失败。如果这个选项被选中，那么该步骤的返回值将是状态代码。然后咱们便可以将其与零进行比较，比如说。


- `returnStdout: boolean` (可选项)

如果选中，任务的标准输出将作为 `String` 的步骤值返回，而不是被打印到构建日志。(如果有任何标准错误，仍然会被打印到日志中。)咱们会经常打算在结果上调用 `.trim()` 来去除尾部的新行。


## `dir`：修改当前目录

修改当前目录。`dir` 代码块内的所有步骤都将使用这个目录作为当前目录，而任何相对路径都将使用他作为基本路径。

- `path: String`

工作区中目录的相对路径，作为新的工作目录使用。


## `node`：分配节点

在某个节点（通常是个构建代理）上分配一个执行器，并在该代理的工作区上下文中运行进一步的代码。


- `label: String`

计算机名称、标签名称或任何其他如同 `linux && 64bit` 的标签表达式，以限制该步骤于何处构建。可以留空，在这种情况下，将采用任何可用的执行器。


**支持的运算符**

支持以下运算符，按优先级降序排列：


`(expression)`

小括号 -- 用于显式定义表达式结合性。


`!expression`

`NOT` -- 否定；表达式的结果必须 **不** 是真。


`a && b`

`AND` -- 表达式 `a` 和 `b` **都** 必须为真。


`a || b`

`OR` -- 表达式 `a` 或 `b` 中的 **任何一个** 可能为真。


`a -> b`

“隐含, implies” 运算符 —— 相当于 `!a || b`。

例如，`windows -> x64` 可以被认为是 “如果使用 Windows 代理，那么该代理必须是64位的”，同时仍然允许该块在任何没有 `windows` 标签的代理上执行，而不管他们是否也有 `x64` 标签。


`a <-> b`

“当且仅当” 运算符 —— 等同于 `a && b || !a && !b`。

例如，`windows <-> dc2` 可以被认为是 “如果使用 Windows 代理，那么该代理 **必须** 在数据中心 2，但如果使用非 Windows 代理，那么他 **必须不** 在数据中心 2”。


**注意点**

- 所有运算符都是左结合的，即 `a -> b -> c` 等同于 `(a -> b) -> c`；

- 如果标签或代理的名称包含与运算符语法冲突的字符，则他们可以用引号引起来；

    比如 `"osx (10.11)" || "Windows Server"`。

- 表达式可以不写空格，但为了可读性，建议包含空格；Jenkins 在计算表达式时将忽略空格；

- 不支持用通配符或正则表达式来匹配标签或代理的名称；

- 空的表达式将始终计算为 **真，`true`**，而匹配所有代理。


**示例**

`master`

这个代码块只能在 Jenkins 的内建节点上执行。


`linux-machine-42`

此代码块只能在名为 `linux-machine-42` 的代理上执行（或在任何恰好具有名为 `linux-machine-42` 的标签的机器上执行）。


`windows && jdk9`

该代码块只能在安装了第 9 版 Java 开发包的任何 Windows 代理上执行（假设安装了 JDK 9 的代理已被赋予 `jdk9` 标签）。


`postgres && !vm && (linux || freebsd)`

这个代码块只能在 Linux 或 FreeBSD 代理上执行，只要他们不是虚拟机，并且安装了 PostgreSQL（假设每个代理都有适当的标签 -- 特别是，在虚拟机中运行的每个代理必须有 `vm` 标签，才能使这个例子按预期工作。）


## `powerhell`：Windows Powershell 脚本


- `script: String`

执行一个 Windows PowerShell 脚本（版本 3 或更高）。允许多行。

注意：请当心 [Windows PowerShell 和 PowerShell Core 之间的区别](https://docs.microsoft.com/en-us/powershell/scripting/whats-new/differences-from-windows-powershell?view=powershell-7.2)，要检查咱们代理上有着哪一个。

- `encoding: String` (可选项)

{{#include ./nodes_and_processes.md:20:36}}


## `pwsh`：PowerShell Core 脚本

- `script: String`

执行一个 PowerShell 脚本。允许多行。这个插件支持 PowerShell Core 6+。

{{#include ./nodes_and_processes.md:141}}

{{#include ./nodes_and_processes.md:20:36}}


## `sh`: Shell 脚本


- `script: String`

运行 Bourne shell 脚本，通常在 Unix 节点上。接受多行。

可以使用解释器选择器，an interpreter selector，例如： `#！/usr/bin/perl`。

否则，系统默认的 shell 将在使用 `-xe` 命令行开关下被运行（咱们可指定 `set +e` 和/或 `set +x` 来禁用这些命令行开关）。


{{#include ./nodes_and_processes.md:20:36}}


## `ws`：分配工作区

分配一个工作区。请注意，工作区是通过 `node` 步骤自动为咱们分配的。


- `dir: String`

在 `node` 步骤中，会自动为咱们分配一个工作区，或者咱们可以通过这个 `ws` 步骤获得一个备用工作空间，但默认情况下，会自动选择位置。(类似于 `AGENT_ROOT/workspace/JOB_NAME@2`。)


咱们可以在这里指定某个路径，而该工作区将被锁定。(该路径可以是相对于构建代理根的，也可以是绝对的）。

如果并发构建，concurrent builds，要求相同的工作区，那么一个后缀为 `@2` 的目录可能被锁定。目前没有等待锁定所请求的确切目录的选项；如果咱们需要强制执行这种行为，咱们既可以在 `pwd` 表明咱们得到了一个不同的目录时进行失败处理（`error`），或者咱们可以通过一些其他手段（如 `lock` 步骤）强制顺序执行构建的这一部分。

如果咱们不关心加锁的问题，那么只需使用 `dir` 步骤来改变当前目录。


（End）


