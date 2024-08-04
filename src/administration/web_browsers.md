## 浏览器兼容性

> 本页记录了 Jenkins 自动化服务器中的浏览器支持政策。他不适用于 Jenkins 网站或 Jenkins 项目托管的其他服务。


### 支持模型

Jenkins 的 web 浏览器支持分为三类：

1. 级别 1：旨在主动修复这些浏览器，并在所有浏览器上提供平等的用户体验;
2. 级别 2：接受补丁来解决问题，并尽最大努力确保至少有一种方法可以执行任何操作；
3. 级别 3：没有保证。我们将接受补丁，但前提是他们的风险较低。**对于下面未列出的浏览器/版本，这是默认浏览器支持的默认级别**。

我们（Jenkins 开发团队）不声明与浏览器的预发布（例如，alpha、beta 或 canary）版本的任何兼容性或接受错误报告和补丁。

### 浏览器兼容性汇总表

| 浏览器 | 级别 1 | 级别 2 | 级别 3 |
| :-- | :-- | :-- | :-- |
| Google Chrome | 最新的定期发布/补丁 | N-1 版本，最新补丁 | 别的版本 |
| Mozilla Firefox | 最新的定期发布/补丁；最新的 [ESR](https://www.mozilla.org/en-US/firefox/organizations/) 发布版本 | N-1 版本，最新补丁 | 别的版本 |
| Microsoft Edge | 最新的定期发布/补丁 | N-1 版本，最新补丁 | 别的版本 |
| Apple Safari | 最新的定期发布/补丁 | N-1 版本，最新补丁 | 别的版本 |

对移动浏览器（例如 iOS Safari）的支持尚未确定。

### 更改历史

- 2022-02-01 - 移除对 Internet Explorer 的支持，增加 Edge（ [开发者邮件列表中的讨论](https://groups.google.com/g/jenkinsci-dev/c/piANoeohdik) ）；
- 2019-11-19 - 政策更新（[开发者邮件列表中的讨论](https://groups.google.com/forum/#!topic/jenkinsci-dev/TV_pLEah9B4) ）；
- 2014-09-03 - Jenkins 1.579 的最初政策（ [治理会议记录](http://meetings.jenkins-ci.org/jenkins/2014/jenkins.2014-09-03-18.01.html) ）。
