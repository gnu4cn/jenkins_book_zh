# 备忘录


## `Scripts not permitted to use staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods...`


```console
+ rm -rf scm2010_watcher.wise-release_nightly_20230902.tar.gz scm2010_watcher.wise-build_20230902.log *.html
Scripts not permitted to use staticMethod org.codehaus.groovy.runtime.DefaultGroovyMethods contains java.lang.CharSequence java.lang.CharSequence. Administrators can decide whether to approve or reject this signature.
```

原因是这个 `sh` 步骤中，包含了多个 `.` 符合，违反了 Jenkins 的相关限制。


（End）


