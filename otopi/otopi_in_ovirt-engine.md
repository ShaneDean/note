# 前言
oVirt项目的几个关键组件如engine、vdsm，都是通过otopi来完成install、config、uninstall的。

本篇重点介绍ovirt-engine是如何使用otopi的。

# engine的脚本

在完成ovirt-engine开发环境构建之后，在${PREFIX}/bin/目录下面，有一些可执行的脚步，其中包括engine-setup、engine-cleanup、engine-config等会经常使用的脚步。


```
ls -al
//连接关系  ： engine-setup  ->    xxxxx/ovirt-engine-setup
```

## ovirt-engine-setup

这是一个shell脚步，它的作用是准备setup的环境(ovirt-engine-setup.env)和参数(baseenv)，我们也可以通过 --help来查看我们可以传递进去的一些选项。

命令的最终是执行 

    exec "${otopidir}/otopi" "${baseenv} ${environment} ${otopienv}"
    
在engine的packaging/setup目录下还提供了两个相关的重要内容。

    ovirt_engine_setup  // python module
    plugins             // otopi中定义的插件
    
**ovirt_engine_setup**模块的功能提供给plugins目录下面的插件所使用。

![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/ovirt_engine_packages_setup_plugins_list.png?raw=true)

对应的关系是 

    ./plugins/${groups}/[${directory}]/${plugins}/__init__.py

每个_ _ init _ _.py文件里面定义了plugin

![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/ovirt_engine_packages_setup_plugins_plugin_createPlugins.png?raw=true)

所以最后都会通过otopi的context.registerPlugin来保存全部加载到的context变量_plugins中。

## engine-setup log

执行

    ./bin/engine-setup --log=./engine-setup.log
    
无论配置过程成功或失败，我们可以查看engine-setup.log文件。其中里面包含了 ENVIRONMENT 和 SEQUENCE的dump信息。
ENOVIRONMENT包含了各类环境变量。
SEQUENCE里面就根据不同的阶段显示了所有需要执行的各类方法。

SEQUENCE 显示结构如下所示

    STAGE  ${Stages.STAGE_XXXX}
        METHOD  otopi.plugins.${groups}.[${directory}].${plugin}.Plugin.${method}
        METHOD  otopi.plugins.${groups}.[${directory}].${plugin}.Plugin.${method}
    
我们可以根据对应的关键信息在engine项目中找到对应的实现代码。

## debug 

使用调试工具可以帮助我们分析代码、理解原理。

下面介绍如何调试 engine-setup程序

使用Pycharm导入otopi项目
新建一个Run/Debug Configurations

![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi-debug-ovirt-engine-setup.png?raw=true)

图中标号：

### 1

otopi 的入口是 otopi/src/bin/otopi的shell，最终执行

    exec "${OTOPI_PYTHON}" -B -m otopi.__main__ "${extraenv} $*"
    
此处我们直接使用新建一个python 的run/debug ，入口程序就是exec python -B -m 后面对应的otopi._ _ main__
### 2
ovirt-engine项目中的engine-setup已经指定了传给otopi的参数，也可以在exec前面增加echo来查看

### 3

增加 

    OTOPI_DEBUG = 1 
    
由于engine-setup执行的过程中import了部分engine里面的python lib, 地址如下：

    ovirt-engine/packaging/setup/ovirt_engine_setup
    ovirt-engine/packaging/pythonlib/ovirt-engine

需要相应的修改 PYTHONPATH

这样就可以执行调试了


# 为engine脚本添加一个plugin来完成指定工作

首先需要明确需要完成哪些功能，明白是哪个阶段来做这个工作


    //todo





