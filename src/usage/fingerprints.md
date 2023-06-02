# 指纹

## 使用指纹跟踪整个 Jenkins 作业的文件使用情况

当咱们在 Jenkins 上有相互依赖的项目时，往往很难跟踪某个文件的哪个版本被哪个版本的对该文件的某个依赖所使用。Jenkins 支持 **文件指纹，file fingerprinting** 来记录依赖关系。

例如，假设咱们有一个 TOP 项目，他依赖于一个 MIDDLE 项目，而 MIDDLE 又依赖于一个 BOTTOM 项目。咱们正在进行 BOTTOM 项目的工作。TOP 团队报告说他们正在使用的 `bottom.jar` 导致了一个空指针异常（Null-Pointer Exception, NPE），而咱们（BOTTOM 团队的成员）认为咱们在 BOTTOM #32 中修复了这个问题。Jenkins 可以告诉咱们哪些 MIDDLE 构建和 TOP 构建正在使用（或不使用）咱们的 `bottom.jar` #32。


## 怎样设置文件指纹？


