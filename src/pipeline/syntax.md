# 流水线语法

**Pipeline Syntax**

这一小节是建立在 [流水线入门](./get_started.md) 中介绍的信息之上的，应仅作为参考。关于如何在实际示例中使用 Pipeline 语法的更多信息，请参考本章的 [使用 `Jenkinsfile`](./jenkinsfile.md) 部分。从 Pipeline 插件的 2.5 版本开始，Pipeline 就支持了两种不同的语法，详见下文。关于两种语法各自的优点和缺点，请参阅 [语法比较](#两种语法比较)。

正如本章开头所讨论的，流水线的最基本部分是 “步骤, step”。基本上，步骤会告诉 Jenkins 要做 *什么*，并作为声明式和脚本化流水线语法的基础构建块，the basic building block for both Declarative and Scripted Pipeline syntax。



