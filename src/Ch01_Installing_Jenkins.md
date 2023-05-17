# 安装 Jenkins

本章中的过程适用于 Jenkins 的新安装。

Jenkins 通常在其自己的进程中作为独立应用程序运行。 Jenkins WAR 文件捆绑了 [Winstone](https://github.com/jenkinsci/winstone)，一个 [Jetty](https://www.eclipse.org/jetty/) servlet 容器包装器，并且可以在任何操作系统或平台上启动，只要有 Jenkins 支持的 Java 版本。

理论上，Jenkins 也可以在 [Apache Tomcat](https://tomcat.apache.org/) 或 [WildFly](https://www.wildfly.org/) 这样的传统 servlet 容器中作为 servlet 运行，但在实践中，这基本上没有经过测试，而且有很多注意事项。特别是，对 WebSocket 代理的支持只在 Jetty servlet 容器中实现。详见 [Servlet 容器支持政策](https://www.jenkins.io/doc/administration/requirements/servlet-containers) 页面。
