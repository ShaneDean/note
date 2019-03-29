# 前言

本文以otopi源码的文档和参考资料[Ovirt-host-deloy_3.2.pdf](https://resources.ovirt.org/old-site-files/wiki/Ovirt-host-deploy_3.2.pdf)为输入，用自己的语言来组织下对otopi的理解和使用。

# otopi
全名 oVirt Task Oriented Pluggable Installer/Implementation

基于独立插件的安装框架，可以用来设置系统环境。使用插件的性质尽可能的简化安装新功能的难度，摆脱状态和事务管理的复杂性。

    // fixme  状态和事务管理的复杂性在哪里？为什么插件管理简单？

具有如下特点

1. 模块化、面向任务的库实现 
2. 支持可插拔管理器的对话框协议，用于人机对话
3. 本地化支持
4. 支持本地和远程模式执行
5. 分布式独立实现
6. 兼容python 2.6 2.7 3.2


    // fixme 这6点分别对应的是哪些代码?

留些问题，回头来看。    

## 安装顺序


开放把提供的插件放到插件组中，每个插件需要使用@plugin.event来注册，需要提供阶段和优先级信息，每个插件可以提供用于检查条件的回调。状态管理是通过k/v的Environment来实现。

![install_sequence](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_installation_sequence.png?raw=true)

Environment中的k/v值有三个来源，默认值、初始化之、通过Config文件覆盖的值。

在plugin.py文件中定义了18个阶段，他们的进入顺序如图中箭头所示。

每个阶段定义了7个默认的优先级，数字越小优先级越高，最后执行插件方法排序的依据是优先级数字的大小，所以每个插件在传入PRIORITY_X的时候可以做+/-运算来调整执行顺序。


## 插件组

所有插件都由插件组来划分，可以通过ENVIRONMENT来传递给otopi，接下来Otopi会去加载${group_name}下的插件。

    APPEND:BASE/pluginGroups=str:${group_name}
    
文件结构如下：

![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/ovirt_engine_packages_setup_plugins_list.png?raw=true)

如其中ovirt-engine-common就是plugin groups，core文件夹下面的_ _ init__.py文件里面定义了core目录中包含了多少plugin. 如下图

![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/ovirt_engine_packages_setup_plugins_plugin_createPlugins.png?raw=true)

定义了5个plugin

- firewall_manager
- firewall_ mamager_firewalld
- firewall_ manager_human
- firewall_ manager_iptables
- hostname

所有Plugin都继承自PluginBase并使用@Plugin.event注解来声明该插件所在的流程位置，python模块可以使用createPlugins方法来加载插件。

下面是一个示例，假设写了plugin example1 在group1里面


```
//__init__.py
from otopi import util

from . import example1

@util.export
def createPlugins(context):
    example1.Plugin(context=context)
```


```
//example1.py
import platform
import gettext
_ = lambda m: gettext.dgettext(message=m, domain='otopi')


from otopi import constants
from otopi import util
from otopi import plugin
from otopi import filetransaction


@util.export
class Plugin(plugin.PluginBase):

    def __init__(self, context):
        super(Plugin, self).__init__(context=context)

    #
    # 使用默认的优先级注册到STAGE_INIT阶段
    #
    @plugin.event(
        stage=plugin.Stages.STAGE_INIT,
    )
    def _init(self):

        #
        # 使用默认值来保持现存的ENVIRONMENT
        #
        self.environment.setdefault('var1', False)

    #
    # 执行验证，修改的最后机会
    #
    @plugin.event(
        stage=plugin.Stages.STAGE_VALIDATION,
        priority=plugin.Stages.PRIORITY_LOW,
    )
    def _validate(self):
        if not self._distribution in ('redhat', 'fedora'):
            raise RuntimeError(
                _('Unsupported distribution for iptables plugin')
            )

    #
    # 执行插件的对应工作.
    #
    @plugin.event(
        stage=plugin.Stages.STAGE_MISC,
        condition=lambda self: self.environment['var1'],
    )
    def _store_iptables(self):
            self.environment[constants.CoreEnv.TRANSACTION].append(
                filetransaction.FileTransaction(
                    name='/etc/example1.conf',
                    content=(
                        'hello',
                        'world',
                    )
                )
            )

@util.export
def createPlugins(context):
    Plugin(context=context)

```

## 环境

环境包含两个部分 系统变量和安装环境变量

### 系统变量

可以用来覆盖安装环境变量，一般用来调试。

选取几个变量

```
    OTOPI_CONFIG
        配置文件
        覆盖了 安装环境的CORE/configFileName.

    OTOPI_DEBUG
        是否开启调试特性，0表示关闭
        覆盖了 安装环境的BASE/debug.
    
    OTOPI_LOGDIR
        日志目录
        覆盖了安装环境的CORE/logDir.
    
    OTOPI_LOGFILE
        指定了日志文件
        覆盖了安装环境的CORE/logFileName.
```

### 安装环境变量

在constants.py中定义了所有的插件变量

![otopi_installer_environment_constants_py_variables](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_installer_environment_constants_py_variables.png?raw=true)

其中两个注解的意思是

    @util.export    //导出该类
    @util.codegen   //将下面变量定义成class的可访问属性

上图中定义的变量都可以通过import的方式被plugin访问引用，从而影响plugin的行为，而且更进一步让ovirt-engine通过otopi提供的MachineDialogParser类来与变量交互。

![netenv_ssh](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_ovirt_engine_set_NetEnv_SSH_USER.png?raw=ture)

参考几个关键的Environment

```

    BASE/pluginGroups(str)
        通过 ":"来切分的插件组
    CORE/configFileName(str) [/etc/otopi.conf]
        默认路径是/etc/otopi.conf的配置文件
    NETWORK/sshEnable(bool) [False]
        是否存储ssh的密钥，默认不存。
    NETWORK/iptablesRules(multi-str)
        iptables的过滤规则，多条str
```

## 对话框

dialog是plugins用来和外部交互的接口。交互的对象包括Human和Machine类。

我的理解是：
    
    human的交互模式就是 command line interactive mode
    
    machine的交互模式就是answer file mode
    
支持的交互包括

- terminate
- note
- queries 
    - string
    - multi-string
    - value
- display
    - multi-string
    - value
- confirm

### 定制和结束对话框

在启动otopi之前传入DIALOG/customization=bool:Ture，可以开启comman 交互模式，这个模式下会有一些简单的命令可以使用，如下

```
abort - Abort process
env-get - Get environment variable
env-query - Query environment variable
env-query-multi - Get multi string environment variable
env-set - Set environment variable
env-show - Display environment
exception-show - show exception information
help - Display available commands
install - Install software
log - Retrieve log file
noop - No operation
quit - Quit

```
可以通过这些命令来操作environment，一般在每个stages的定制和结束前可以执行。效果如下图

![otopi_dialog_customization](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_dialog_customization_true.png?raw=true)

## bundle 
略


# 源码分析

otopi属于Ovirt下的一个子项目，开发仓库[gerrit](https://gerrit.ovirt.org/)

```
    git clone https://gerrit.ovirt.org/otopi
``` 

## 安装

项目文档中提供了INSTALL

```
    autoreconf -ivf
    ./configure
    make
    make install
```

## 目录

核心的内容在一下四个目录
- src/bin   otopi命令入口脚本
- src/java  otopi提供给外部Java程序和Environment交互的工具jar包的源码
- src/otopi otopi架构的核心代码
- src/plugins/otopi otopi内置的插件，外部在使用otopi来实现安装过程中会默认加载这些插件

### bin/otopi
这个脚本在完成必要的 环境检查（bundle\root\ptyon-version)之后进入otopi._ _ main__中

### src/otopi

梳理下otopi运行的核心代码的流程如下图

![otopi_code_process_flow](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_core_code_process_flow.png?raw=true)

可以简单分成两部分：

```

    installer =main.Otopi() //开始进行初始化工作。
    
    installer.execute() //根据传入的参数开始执行对应的业务。
    
```

**初始化部**分可以看到，otopi主要包含了4个对象:
- Dialog
  - humam
  - machineDialog
- Services
  - rhel
  - openrc
  - systemd
- Packager
  - dnfpackager
  - yumpackager
- Command

除了command外其他都属于虚类，具体实现由子项中的子类来实现。在otopi的初始化阶段，context只保存虚类，所有子类全部由插件的方式提供，该子类插件的对应阶段执行时，执行响应的context.registerXXXXX()方法。

例如：

![otopi_human_dialog_registerdialog](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_human_dialog_registerdialog.png?raw=true)

在context中，registerXXXX()都属于覆盖（registerPlugin、registerNotification除外），在完成覆盖后，由对应插件的子类来实现具体功能，如下：

```
    def registerNotification(self, notification):
        self._notifications.append(notification)

    def registerPlugin(self, p):
        self._plugins.append(p)

    def registerDialog(self, dialog):
        self._dialog = dialog

    def registerServices(self, services):
        self._services = services

    def registerPackager(self, packager):
        self._packager = packager

    def registerCommand(self, command):
        self._command = command
```

**运行阶段**分为三个步骤，加载插件、构建插件执行序列、按序列执行插件。

#### 加载插件

**加载插件**的主要工作是根据BASE/pluginGroups指定的目标插件组(needgroups)，去BASE/pluginPath路径下寻找。

在needgroups中，otopi会默认增加自己预定义的插件，即src/plugins/otopi目录。

内置的插件共22个

![default_plugin](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi_default_plugins.png?raw=true)

找到对应的Plugin中后会调用Util.loadModule来加载该python模块，并执行该模块的_ _ init__.py中的createPlugins方法，该方法中定义了模块路径下所有的plugin入口。

所有定义的plugin都会继承PluginBase类，该类的_ _ init _ _ 方法中会调用registerPlugin，从而在context._plugins[]中保存了所有注册的plugins，这是第二步骤的输入。

#### 构建插件执行顺序

**构建插件执行序列**的主要工作是根据@util.event中的before和after来构建一个有向无环图(toposort)。每个plugin都可以定义before和after两个变量，设定当前的plguin为current。

根据注释中 **before=EVENTNAMESLIST -- place this event before the events with names EVENTNAMESLIST.** 的说明，那么可以产生(current,before)和(after,current)多个向量【经过分析代码验证相同】，通过全部插件的全部向量，我们可以绘制出多个有向无环图(需要错误检查)。同时使用prioty和stage两个参数来实现分组。

下面随便寻找一个toposort group结果来示意下：


执行构建的方法是 _toposortBuildSequence ，简单的理解就是先根据

```
 (current,before)
 (after,current)
 i.stage == j.stage && i.priority < j.priority ? (i, j) : ( j , i )
 ```
 来生成对应的向量，最后的结果如下图
 
 ![deps_vector](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/otopi__toposortBuildSequence_deps.png?raw=true)
 
 挑一些标出来的向量，如
 
 红色 : (1,256) .  绿色 : (4, 73) .  蓝色 : (6,20) .

这样的就有了一堆的经过检查的向量。下面的就是将这些已定义的向量进行拓扑排序。

拓扑排序的方法是_toposort ， 算法学习详见[这篇](http://note.youdao.com/noteshare?id=fc2dfe95bd2905d91185bafc91d7b149&sub=422857DAC7294892AD99F5E9A2D62326)

#### 按序列执行插件

排序后得到的结果是一个二维list，这里的stage就是otopi预定的18个阶段的枚举值，这里的method 包含了plugin主要信息的结构体（包含了每个插件的method和condition）
```
[
    {
        stage0:[method0,method1,...]
    },
    {
        stage1:[method0,method1,...]
    },
    ...
]
```

然后就是按前面说的安装顺序依次执行method

这里注意2点：
1. 每次执行method的时候需要先执行condition，condition由定义plugin时候指定
2. 每次执行method前需要保存oldEnvironment，因为Plugin在执行的过程中有可能会修改Environment，需改的部分都会被dump到日志中。

### src/plugins/otopi

下面针对otopi提供的几个内置的plugin来进行分析。内容也会涉及部分ovirt-engine的setup脚本，下面内容不做区分。

Stage_boot:

    根据environment定义dialog的模式（human/machine）
    注册一些信息到environment中，如包名、版本
    确定packager管理方式dnf/yum，默认推荐dnf
    //todo dnf和yum的代码逻辑 otopi.plugins.otopi.packagers.[dnfpackager/yumpackager]
    把密码设置为敏感字，在日志打印时候就会屏蔽不输出
    
Stage_init：

    找到所有需要解析的配置文件
    otopi的初始化过程由每个Plugin执行对应的初始化来完成，如packager\command\transaction
    //todo transaction的代码逻辑 otopi.core.transaction
    初始化部分插件相关的environment，如cli/firewalld/iptables/ssh/clock/reboot/answerfile/hostile_service/selinux/fence_kdump_listener/db/java/....
    
Stage_setup

    检测关键命令是否存在，如ip/firewall-cmd/reboot/openssl.....
    计算系统的一些信息，如最大内存
    连接数据库
    
Stage_internal _packages

    //todo  trascation相关
    
Stage_programs

    在PATH路径下找对应的命令
    
Stage_late _setup

    nfs/websocket
    
Stage_customization

    解析conf文件定义的变量，并覆盖现有的变量
    使用command进行customization
    用户通过输入plugin接收的参数来控制产品的形态，并影响对应的environment，engine的产品configure阶段就是在这个阶段来整合。

Stage_validation

    检查写到Environment 里面的value的格式
    检查当前依赖的服务运行状态、系统信息、db权属、需要的文件和目录
    执行预写的脚本完成特定的任务，如taskcleaner
    
Stage_transaction _begin

    fence_kdump_listener/websocket/notifier service
    //todo ?? 

Stage_early _misc

    使用firewall-cmd重启服务
    //Fixing Engine database inconsistences


Stage_packages

    这里可以用来安装依赖的外部packages
    //yum/dnf -- processTransaction()  ??
    
Stage_misc

    使用filetransaction来完成文件操作
    生成CA证书、数据库初始化或升级
    把其他plugin的更新信息加入到数据中
    
Stage_transaction _end

    transaction.commit()

Stage_closeup

    启动或重启服务
    打印总结信息
    
Stage_cleanup

    显示日志路径、生成answer file

Stage_pre _terminate

    dump environment

Stage_terminate

    log/human/machine

Stage_reboot

    或许重启系统


