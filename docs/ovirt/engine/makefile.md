# 前言
make 命令是GNU的工程化的编译工具，用于编译众多相互关联的源代码，通过工程化的管理来提供开发效率。ovirt-engine项目中有一个Makefile，这是理解项目代码的入口。

# make

make本身的资料很多，[官网](https://www.gnu.org/software/make/manual/make.html),[参考1](http://www.ruanyifeng.com/blog/2015/02/make.html),[参考2](https://docs.huihoo.com/gnu/linux/gmake.html)。

个人理解，可以把make看做一个将先定义任务再将任务拆分成小任务的工具，它有自己的简单语法和规则帮助你完成拆分和组织工作。它和bash配合紧密，我们可以通过bash和其他的工具、命令、语言来统筹配合完成更多的事情。

就以ovirt-engine为例， engine使用了make命令来完成各类任务，如配置开发环境(install-dev)、打包(rpm)、开启gwt调试(gwt-debug)等。但后面实际的工作可能是mvn、rpmbuild、linux command、linux shell、python来完成的，make提供了入口，它定义并拆分了任务，再交由其他工具来完成具体的小任务。

在工程组织方面，make只提供了最基础的功能，诸如[cmake](https://cmake.org/)、[automake](https://www.gnu.org/software/automake/)等工具则更加强大，但不是本文覆盖的范围，感兴趣的同志请自行阅读。

# Makefile

## 控制参数

engine在makefile 的开始 定义了很多控制参数，这些控制参数可以来影响make的流程。  

- BUILD_UT  是否开启单元测试
- BUILD_VALIDATION 是否进有效性检查。(shell-check.sh\image-check.sh\dbscripts-duplicate _upgrade _stripts.sh)
- DEV_BUILD _GWT _SUPER _DEV _MODE  是否开启[GWT super Dev mode](http://www.gwtproject.org/articles/superdevmode.html)
- BUILD _ALL _USER _AGENTS  是否编译全部的浏览器代理类型
- 等等

## 目录层级定义

//略

## 变量

//略 

## 目标

engine项目汇总的makefile里面定义了很多目标，开发者用的最多的应该是make install-dev 、make dist 、 make clean

- make install-dev 负责安装ovirt-engine到一个PREFIX目录，如果是prod部署状态 这个PREFIX是 / 根目录。相应的，在install-dev的dev部署模式中，*.in文件中的变量都会替换成PREFIX的相对路径，这样就可以直接通过 $PREFIX/bin/engine-setup 来启动配置初始化工作 或 $PREFIX/xxxx/ovirt-engine.py start来启动系统服务等。

- make dist 则是修改ovirt-engine.spec.in文件,具备rpmbuild的条件
 
- make clean 是执行mvn clean 和 rm $(BUILD _FILE) ${GENERATED _FILE)， 完成清理工作

除此之花还包括很多其他任务目标，如：

- .in: 替换变量的占位符成实际计算出来的值
- maven ： 传入 对应的otps或flags ，执行mvn install
- copy-recursive: 地柜地将源目录文件复制到目的位置
- install-packaging-files ： 将项目中的packaging目录 下为文件放到各自的指定目录 ，如  etc/pki/bin/services/dbscripts/pythonlib等
- install-layout: 建立目录结构、文件结构
- 等

# engine makefile

整理后的xmind

![ovirt-engine-4.3-makefile-xmind](https://github.com/ShaneDean/file/raw/7ca6fbc6e895ea99af34e66b945e257dc3ae8888/blog/ovirt_engine_env/ovirt-engine-Makefile.png)