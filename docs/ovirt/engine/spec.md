# 前言
以 ovirt-enigne-4.3为例，分析engine是如何打包成 rpm 的。

[spec-macro](https://rpm.org/user_doc/macros.html),

[rpm-package-guide](https://rpm-packaging-guide.github.io/)

[maximum-rpm](http://ftp.rpm.org/max-rpm/index.html)

# rpm

# spec

spec 文件是rpmbuild的工作手册，手册里面定义了构建系统在各个阶段中执行的命令。


spec文件由不同的条目组成，具体如下：

-   注释    comments    人类阅读使用，rpm会忽略
-   标签    tags        定义数据
-   脚本    scripts     包含了特定时间执行的命令
-   宏      macros      
-   %files列表          pkg需要包含的文件列表
-   指令    directives  处理%files列表的方式
-   条件    conditionals    执行系统、架构相关的预处理工作

注释：

    # 后面的内容

注释也可以展开 但需要使用\%\%进行转义


## 标签 tags

格式

    <something>:<something-else>

 **somethin** 大小写、距离 都不敏感

内置：
-   Preamble    变量，Package Naming Tags
    -   Name        包名
    -   Version     包版本
    -   Release     发布次数，每次发布递增，新版本产生式时重置为1，1%{?dist}
    -   Summary     
    -   License     license版权
    -   URL
    -   Source0     上游的源代码，应当是远程可靠的访问地址
    -   Patch0      上游补丁，应当是远程可靠的访问地址
    -   BuildArch   如果pkg架构无关，则为noarch，如果相关则为架构缩写
    -   BuildRequires   构建依赖
    -   Requires    运行依赖
    -   ExcludeArch 不支持架构
-   Body        指令集合， Descriptive Tags
    -   %description    包描述内容
    -   %files          会被install的文件列表
    -   %changelog      发型记录

## 脚本 script

分类

-   %prep           一系列命令来准备软件的构建环境
-   %build          一系列命令来执行软件的构建
-   %install        一系列命令来完成从%builddir到%buildroot目录的拷贝 （这里只有在创建pkg的时候，不是实际的用户环境的安装过程）
-   %check          一些列命令来测试软件，比如包括 unit test case
-   %pre            在安装pkg之前执行
-   %post           在安装pkg之后执行
-   %preun          在卸载pkg之前执行
-   %postun         在卸载pkg之后执行


## 宏 macro

spec内置2个简单的宏来执行特定的任务

-   %setup      用来解压源代码，需要SOURCE
-   %patch      用来给代码打补丁, 需要PATCH

同时用户可以自己定义宏， 包含两种宏。

-   Simple Macros : 直接的文本替换
-   Parameterized Macros ： 包含选项字段，将下一行用空格分隔的字段设为args

定义

    %define <name>[(opts)] <body>

包围body的空格都会被移除。 name 可以是字母数字和\_组成，长度最少是3个字符。

如果没有opts,macro只执行展开操作。

如果opts中有参数的话，它的处理方式是[getopt](https://linux.die.net/man/3/getopt)。宏中的可用变量是：
-   %0      被调用宏的名称
-   %*      所有参数，不包括处理标志
-   %**     所有参数，包括处理标志
-   %#      参数数量
-   %{-f}   调用出现时,标志f的本身
-   %{-f*}  调用出现时,标志f指向的参数
-   %1, %2  参数本身

无参数宏的情况下上面的变量都会废弃。

%{flag} 取flag

%{?flag:X} 如果flag出现则为falg,不出现则为X

%{!?flag:X} 和上面相反

%(shell command) 执行结果即输出 ， 如%(date +%%y%%m%%d) ，这里的%%转义为命令执行中的% 避免再次展开

下面是可用来执行内嵌宏

-   %trace          打印扩展前后的调试信息
-   %dump           打印可用宏列表
-   %getncpus       可用cpu数量
-   %dnl            放弃下一行，不展开(>=4.15.0)
-   %{echo:...}     打印到stdout
-   %{warn:...}     打印到stderr
-   %{error:...}    打印到stderr，并报错
-   %define ...     定义宏，lazy展开
-   %undefine ...   取消宏定义
-   %global ...     定义全局可用宏 ， eager展开
-   %{load:...}     加载宏文件(>=4.12.0)
-   %{expand:...}   再次展开
-   %{shrink:...}   消除环绕的空格成一个空格(>=4.14.0)
-   %{quote:...}    引用一个参数宏作为参数，传递空字符或由空格分隔的字符串
-   %{lua:...}      用lua来展开
-   %{uncompress:...}   用文件方式展开，并检查文件是否压缩，支持未压缩、gzip、bzip3种
-   %{basename:...} [basename](http://man7.org/linux/man-pages/man1/basename.1.html)
-   ${dirname:...}  [dirname](http://man7.org/linux/man-pages/man1/dirname.1.html)
-   %{suffix:...}    expand to suffix part of a file name
-   %{url2path:...} 转换url为本地路径
-   %{getenv:...}   [getenv](http://man7.org/linux/man-pages/man3/getenv.3.html)
-   %{getconfdir:...}   rpm的home目录
-   %{S:...}        expand ... to <source> file name
-   %{P:...}        expand ... to <patch> file name
-   %{F:...}        expand ... to <file> file name

默认变量地址 **/usr/lib/rpm/macros**   **/etc/rpm/**

    rpm --eval '<macro expression>'   # 查看宏

    

使用宏

    %{<name>}       #直接展开成txt
    %<name> ...     #如果有参数则进行调用

eg

    %define mymacro() (echo -n "My arg is %1" ; sleep %1 ; echo done.)
    %mymacro 5
    (echo -n "My arg is 5" ; sleep 5 ; echo done.)

autoconf中的宏变量
```
    %_prefix            /usr
    %_exec_prefix       %{_prefix}
    %_bindir            %{_exec_prefix}/bin
    %_sbindir           %{_exec_prefix}/sbin
    %_libexecdir        %{_exec_prefix}/libexec
    %_datadir           %{_prefix}/share
    %_sysconfdir        %{_prefix}/etc
    %_sharedstatedir    %{_prefix}/com
    %_localstatedir     %{_prefix}/vars
    %_libdir            %{_exec_prefix}/lib
    %_includedir        %{_prefix}/include
    %_oldincludedir     /usr/include
    %_infodir           %{_prefix}/info
    %_mandir            %{_prefix}/man
```

## %files

指向需要在构建系统中打包进rpm中的文件列表。 这些文件都会被作为参数执行2次，第一次是构建系统的路径，第二次是安装到目标系统的路径

%files包含了一些可用的指令

-   明确配置文件、文档
-   检查文件的权限
-   控制pkg验证阶段的文件需要被检查的方面
-   消除一些创建%files列表时候的繁琐工作

具体包括

-   %attr   (<mode>,<user>,<group>)
-   %defattr (<file-mode>,<user>,<group>,<dir-mode>)
-   %doc
-   %config
-   %ghost  rpm知道但不在pkg的文件，如log
-   %docdir
-   %dir    指定目录下的所有信息都属于pkg，不用衍生的打印目录下的信息
-   -f <file>   文件中的每一行都是文件路径
-   %verify     验证特殊的文件，如设备文件
    -   owner
    -   group
    -   mode
    -   md5
    -   size
    -   maj
    -   min
    -   symlink
    -   mtime

## 条件 conditionals

条件包括

-   %ifarch
-   %ifnarch
-   %ifos
-   %ifnos
-   %else
-   %endif

# engine

根据上面的一些知识点 初步整理的engine的spec文件情况如下图

[ovirt-engine-spec](https://raw.githubusercontent.com/ShaneDean/file/a66e4024f1769b5dc16dde24d04dc443fdd768d6/blog/ovirt_engine_env/ovirt-engine-spec.png)
