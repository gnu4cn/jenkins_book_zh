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


CPS 转换了的代码可调用非 CPS 转换代码，或其他 CPS 转换代码，非 CPS 转换代码可调用其他非 CPS 转换代码，但非 CPS 转换代码 **不得** 调用 CPS 转换代码。如果咱们尝试从非 CPS 转换代码调用 CPS 转换代码，CPS 解释器将无法正确运行，从而造成不正确及经常混乱的结果。


## 常见问题及解决方案

**Common problems and solutions**


### 在 `@NonCPS` 中使用 Pipeline 步骤


有时，用户会对方法定义应用 `@NonCPS` 注解，以绕过该方法内部的 CPS 转换。这样做的目的，可能是绕过 Groovy 语言覆盖范围的限制（因为方法的主体将用到原生 Groovy 语义执行），或者是为了获得更好的性能（解释器会带来很大的开销）。不过，此类方法就不得调用 CPS 转换的代码了，比如如 Pipeline 步骤。例如，以下代码将无法运行：

```groovy
@NonCPS
def compileOnPlatforms() {
  ['linux', 'windows'].each { arch ->
    node(arch) {
      sh 'make'
    }
  }
}
compileOnPlatforms()
```

在这个方法中，使用 `node` 或 `sh` 步骤是非法的，会导致行为异常。运行此脚本时，日志中的警告如下：


> *expected to call WorkflowScript.compileOnPlatforms but wound up catching node*


要修复这种情况，只需删除那个注解即可，因为不需要他。(在修正 [JENKINS-26481](https://issues.jenkins.io/browse/JENKINS-26481) 之前，流水线的长期用户，可能认为其是必要的）。



### 以 CPS 转换了的参数调用非 CPS 转换方法

**Calling non-CPS-transformed methods with CPS-transformed arguments**


一些 Groovy 和 Java 方法，会将复杂类型作为参数，以支持动态的行为。一个常见的例子，便是是排序方法，其允许调用者指定用于比较对象的方法 ( [JENKINS-44924](https://issues.jenkins.io/browse/JENKINS-44924) )。在修正 [JENKINS-26481](https://issues.jenkins.io/browse/JENKINS-26481) 之后，Groovy 标准库中的许多类似方法就都能正确工作了，但仍有些方法未被修正。例如，以下方法将无法工作：


```groovy
def sortByLength(List<String> list) {
  list.toSorted { a, b -> Integer.valueOf(a.length()).compareTo(b.length()) }
}
def sorted = sortByLength(['333', '1', '4444', '22'])
echo(sorted.toString())
```

传递给 `Iterable.toSorted` 的闭包是经过 CPS 转换的，但 `Iterable.toSorted` 本身在内部却并没有经过 CPS 转换，因此这不会按预期的方式工作。当前的行为是，调用 `toSorted` 的返回值，将是第一次调用闭包的返回值。在示例中，这导致了 `sorted` 被设置为 `-1`，日志中的警告信息如下：


> *expected to call java.util.ArrayList.toSorted but wound up catching org.jenkinsci.plugins.workflow.cps.CpsClosure2.call*


要修复这种情况，传递给这些方法的任何参数，都不得进行 CPS 转换。而要做到这一点，可以将有问题的方法（示例中的 `Iterable.toSorted`）封装在另一个方法中，并用 `@NonCPS` 注解外部方法，或者为闭包创建一个显式的类定义，并用 `@NonCPS` 注解该类中的全部方法。



### 构造器

**Constructors**

有时，用户可能会尝试在 Pipeline 脚本的构造函数中，使用 CPS 转换了的代码，如 Pipeline 步骤。遗憾的是，通过 Groovy 中的 `new` 运算符构建对象，并不能进行 CPS 转换（ [JENKINS-26313](https://issues.jenkins.io/browse/JENKINS-26313) ），因此这种做法不会管用。下面是一个在构造函数中，调用 CPS 转换方法的示例：


```groovy
class Test {
  def x
  public Test() {
    setX()
  }
  private void setX() {
    this.x = 1;
  }
}
def x = new Test().x
echo "${x}"
```

`Test` 的构造，将在调用 `Test.setX` 时失败，因为 `setX` 是一个 CPS 转换方法。运行此脚本时日志中的警告如下：


> *expected to call Test.<init> but wound up catching Test.setX*


要解决这种情况，就要确保在 Pipeline 脚本中定义的、从构造器内部调用到的任何方法，都以 `@NonCPS` 进行了注解，并且构造器没有调用任何 Pipeline 步骤。如果必须在构造函数中调用 CPS 转换代码（如流水线步骤），则需要将与那些 CPS 转换方法相关的逻辑，移出构造函数，例如移入调用了 CPS 转换代码的静态工程方法中，然后将结果传递给构造函数。


### 非 CPS 转换方法的重写

**Overrides of non-CPS-transformed methods**


用户可在 Pipeline 脚本中，创建出某个对该 Pipeline 脚本外部定义的，例如 Java 或 Groovy 标准库中的已有类，进行扩展的类。这样做时，子类就必须确保，任何重载方法都以 `@NonCPS` 进行了注解，并且这些重载方法内部，都不得使用任何 CPS 转换代码。否则，如果从非 CPS 上下文中调用，这些重写方法就将失效。例如，以下方法将不起作用：


```groovy
class Test {
  @Override
  public String toString() {
    return "Test"
  }
}
def builder = new StringBuilder()
builder.append(new Test())
echo(builder.toString())
```

从比如 `StringBuilder.append` 等非 CPS 转换代码中，调用 `toString` 的 CPS 转换重写，是不允许的，而且在大多数情况下，都不会按预期运行。运行此脚本时日志中的警告如下：


> *expected to call java.lang.StringBuilder.append but wound up catching Test.toString*


要解决这种情况，可在覆盖方法中添加 `@NonCPS` 注解，并从方法中移除任何 CPS 转换代码的使用，如 Pipeline 步骤。


### `GString` 中的闭包

**Closures inside `GString`**


在 Groovy 里， 使用位处 `GString` 中的某个闭包是可行的，这样每次将 `GString` 作为 `String` 使用时，都会对这个闭包进行计算。然而，在 Pipeline 脚本中，这不会像预期的那样起作用，因为 `GString` 中的闭包将被 CPS 转换。下面是一个示例：


```groovy
def x = 1
def s = "x = ${-> x}"
x = 2
echo(s)
```

在本例中，使用 `GString` 内的闭包将不起作用。运行此脚本时日志中的警告如下：


> *expected to call WorkflowScript.echo but wound up catching org.jenkinsci.plugins.workflow.cps.CpsClosure2.call*


要解决这种情况，可以一个用到普通表达式，而不是闭包，返回 `GString` 的闭包，替换原来的 `GString`，然后在用到原来的 `GString` 的地方，调用这个闭包，如下所示：


```groovy
def x = 1
def s = { -> x = "${x}" }
x = 2
echo(s())
```


## 误报问题（假阳性）

**False Positives**


不幸的是，某些表达式即使执行正确，也可能错误地触发此类警告。如果遇到这种情况，请为 `workflow-cps-plugin` [提交一个新问题](https://www.jenkins.io/participate/report-issue/redirect/#21713)（请首先检查是否有重复）。


（End）


