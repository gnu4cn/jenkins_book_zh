# 流水线 CPS 方法不匹配

**Pipeline CPS Method Mismatches**


Jenkins 流水线使用了一个名为 Groovy CPS 的库，来运行流水线脚本。虽然流水线用到 Groovy 解析器和编译器，但与普通的 Groovy 环境不同，他是在一个特殊的解释器中，运行大部分程序的。他用到一种延续传递式（CPS，continuation-passing style）转换，将咱们的代码，转换成一个可以将其当前状态保存到磁盘（在咱们构建目录下，一个名为 `program.dat` 文件）的版本，并在 Jenkins 重新启动后继续运行。(在 [Pipeline： Groovy 插件](https://plugins.jenkins.io/workflow-cps) 页面和 [库页面](https://github.com/cloudbees/groovy-cps/blob/master/README.md)，就可以获取到一些技术背景信息）。

虽然 CPS 转换对用户来说通常是透明的，但对可被支持的 Groovy 语言结构，则有一定的限制，在某些情况下，CPS 转换可能会造成一些违反直觉的行为。[JENKINS-31314](https://issues.jenkins.io/browse/JENKINS-31314) 就提出了，运行时检测一些最常见错误的尝试：自非 CPS 转换代码，调用 CPS 转换后的代码。以下几种情况，就属于 CPS 转换的情形：

- 咱们所编写的几乎所有 Pipeline 脚本（包括库中的脚本）;

- 绝大多数的流水线步骤，包括那些占用代码块的全部步骤；


而以下几种情况，则不属于 CPS 转换：


+ 编译后的 Java 字节码，包括：
    - Java 平台自身；

    - Jenkins 的内核与插件；

    - Groovy 语言的运行时。


- Pipeline脚本中的构造函数主体；

- 流水线脚本中，任何标有 `@NonCPS` 注解的方法；

- 少数几个不需要代码块，且立即执行的 Pipeline 步骤，例如 `echo` 或 `properties`。



