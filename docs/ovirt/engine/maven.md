# 前言

ovirt-engine是一个由maven组织起来java项目，其中也包括makefile、package或otopi plugin等其他手段来协助完成打包、验证、开发环境准备、初始化等工作，但主要的是maven，在分析梳理项目或导入项目的时候都可以以pom.xml为入口。一个大型项目通常都是由几个小项目或者模块组成的，通过pom.xml定义的关系和包含的mvn plugin可以快速的了解每个子项目的作用从而理解整个项目。

例如来make install-dev中出现错误的时候，一般应当找到对应报错的maven项目，切换到该项目目录，分析出错的maven命令的原因，这样就可以了解深层次的错误原因而不是看到报错一筹莫展。

# maven

maven的学习资料比较多，可以通过阅读[参考1](https://segmentfault.com/a/1190000014136187)、[参考2](https://juejin.im/entry/5b0fa70af265da090e3df499)、[参考3](http://jolestar.com/dependency-management-tools-maven-gradle/)、[参考4](http://www.cnblogs.com/davenkin/p/advanced-maven-multi-module-vs-inheritance.html)、[官网](http://maven.apache.org/guides/getting-started/index.html)和[书](https://item.jd.com/10476794.html)来学习。

个人感觉可以和npm类比的学习[参考5](https://codeday.me/bug/20181204/432991.html)

maven本只提供了一个执行环境，真正的操作项目的是由插件来执行的，例如编译java的compile插件、打包的jar插件等等。在安装完maven后会有内置的默认插件，全部定义在所有maven都需要集成的Super Pom里面。

除了maven默认插件之外，熟练使用丰富的外部 plugin是学习maven的另一个重点。

每个plugin都可以通过mvn [plugin]:[action]的方式使用其提供的服务。

例如tomcat-maven-plugin：
```
mvn tomcat:deploy   --部署一个web war包
mvn tomcat:reload   --重新加载web war包
mvn tomcat:start    --启动tomcat
mvn tomcat:stop    --停止tomcat
mvn tomcat:undeploy --停止一个war包
mvn tomcat:run  --启动嵌入式tomcat ，并运行当前项目
```

# pom.xml

在engine项目中的root pom.xml中定义了5个module，
```
build-tools-root： 包含静态检查的一些通用规则配置文件,如checkstyle\findbugs
backend：   后台业务逻辑代码
frontend:   前台业务逻辑代码
ear：          定义jboss运行时的内外部依赖和业务逻辑代码中不同子项目的打包方式、URL映射关系
mavenmake:      执行make install-dev
```

详细分析见下图:

![ovirt-engine-maven-pom](https://github.com/ShaneDean/file/raw/eb68d3da19b6bb1d293900288228650ee4b43f9b/blog/ovirt_engine_env/ovirt-engine-maven-pom.png)

# plugin

engine中各个项目中用到的插件汇总如下：

-   maven-resources-plugin
-   maven-surefire-plugin
-   maven-ejb-plugin
-   maven-source-plugin
-   maven-checkstyle-plugin
-   maven-enforcer-plugin
-   maven-war-plugin
-   maven-deploy-plugin
-   maven-javadoc-plugin
-   ovirt-jboss-modules-maven-plugin
-   maven-compiler-plugin
-   findbugs-maven-plugin
-   maven-shade-plugin
-   maven-dependency-plugin
-   exec-maven-plugin
-   maven-jaxb22-plugin
-   build-helper-maven-plugin
-   maven-antrun-plugin
-   libsass-maven-plugin
-   gwt-maven-plugin
-   maven-processor-plugin
-   maven-clean-plugin
-   lifecycle-mapping
-   maven-ear-plugin
-   taglist-maven-plugin
-   maven-jar-plugin
-   maven-assembly-plugin
-   animal-sniffer-maven-plugin

## ovirt-jboss-modules-maven-plugin

这是ovirt项目团队写的一个maven插件，This plugin is intended to simplify the creation of JBoss Modules in Maven projects.

ovirt-engine使用是jdk1.8，java的原生module是在9才出现。JBOSS自己实现了一套module的机制，需要通过module.xml的文件来声明模块的依赖和资源。[jboss-module-参考资料](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html-single/development_guide/index#class_loading_and_modules)，在engine里面的