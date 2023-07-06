# 流水线开发工具

**Pipeline Development Tools**

Jenkins 的流水线包含了 [内置文档](#内建文档) 和 [代码片段生成器](#代码片段生成器)，他们是开发时的关键资源。他们提供了根据当前安装的 Jenkins 与相关插件的版本进行定制的详细帮助和信息。在本节中，我们将讨论可能有助于开发 Jenkins 流水线的其他工具和资源。


## 命令行的流水线静态分析程序

**Command-line Pipeline Linter**

Jenkins 可以在实际运行某个声明式流水线之前从命令行中验证，或者说 “[静态分析，lint](https://en.wikipedia.org/wiki/Lint_(software))”。这可以通过使用 Jenkins CLI 命令或带适当参数的 HTTP POST 请求来完成。我们建议使用 [SSH 接口](../managing/jenkins_cli.md#通过-ssh-使用命令行界面) 来运行静态分析程序。关于如何正确配置 Jenkins 以实现安全命令行访问的细节，请参见 [Jenkins CLI 文档](../managing/jenkins_cli.md)。


```bash
# 通过 SSH 下的 CLI 进行静态检查
# ssh (Jenkins CLI)
# JENKINS_PORT=[sshd port on controller]
# JENKINS_HOST=[Jenkins controller hostname]
ssh -p $JENKINS_PORT $JENKINS_HOST declarative-linter < Jenkinsfile
```

> 在执行了 `alias jenkins-cli='ssh -l ci_cd_scm -p 32222 ci.senscomm.com'` 时，执行如下命令：


```bash
jenkins-cli declarative-linter < Jenkinsfile
```

```bash
# 使用 `curl` 通过 HTTP POST 进行静态检查
# curl (REST API)
# Assuming "anonymous read access" has been enabled on your Jenkins instance.
# JENKINS_URL=[root URL of Jenkins controller]
# JENKINS_CRUMB is needed if your Jenkins controller has CRSF protection enabled as it should
JENKINS_CRUMB=`curl "$JENKINS_URL/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)"`
curl -X POST -H $JENKINS_CRUMB -F "jenkinsfile=<Jenkinsfile" $JENKINS_URL/pipeline-model-converter/validate
```


### 示例


下面是流水线静态检查器的两个实际例子。第一个示例显示了当传递一个无效的 `Jenkinsfile` 给静态检查器时的输出，这个文件缺少 `agent` 声明的部分。


```groovy
// Jenkinsfile
pipeline {
  agent
  stages {
    stage ('Initialize') {
      steps {
        echo 'Placeholder.'
      }
    }
  }
}
```

```console
# 无效 Jenkinsfile 的静态检查器输出
# 传递一个未包含 `agent` 小节的 Jenkinsfile
$ jenkins-cli declarative-linter < ./Jenkinsfile
Jenkinsfile content '#!/usr/bin/env groovy

libnl-3-depeline {
    agent
        stages {
            stage ('Initialize') {
                steps {
                    echo 'Placeholder.'
                }
            }
        }
}
' did not contain the 'pipeline' step
```


