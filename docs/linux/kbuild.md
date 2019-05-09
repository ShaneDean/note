# 前言
参考[KBUILD系统原理分析](https://jin-yang.github.io/reference/linux/kernel/KBUILD_system.pdf)

# make 总结

1. 依次读取变量"MAKEFILES"定义的makefile文件列表
2. 读取工作目录下的Makefile文件
3. 依次读取工作目录makefile文件中使用指示符"include"包含的文件
4. 查找重建所有以读取的makefile文件的规则
5. 初始化变量值并展开那些需要立即展开的变量和函数并根据预设条件确定执行分支
6. 根据“最终目标”以及其他目标的依赖关系建立依赖关系链表
7. 执行除“最终目标”以外的所有的目标的规则
8. 执行最终目标所在的规则

# KBUILD MAKE

文件名 | 说明
---|---
Makefile | 顶层 Makefile
.config | 内核配置文件
arch/$(ARCH)/Makefile | 具体架构的 Makefile
scripts/Makefile.* | 通用的规则等，面向所有的 Kbuild Makefiles。
kbuild Makefiles | 内核源代码中大约有 500 个这样的文件

## 目标定义

    obj-$(CONFIG_FOO)+=foo.o
    
$(CONFIG_FOO) 可以为 y (编译进内核) 或 m (编译成模块)。否则的话  foo.o就不会被编译连接了

### obj-y

    KBuild Makefile规定编译进内核的文件都存在$(obj-y)的列表中。
    编译所有的$(obj-y)文件，然后调用$(LD) -r将它们合并到一个build-in.o文件中，然后该文件会被其父Makefile连接到vmlinux中
    $(obj-y)中的文件有顺序，可以重复项。连接过程有顺序，有些函数module_init()/__initcall将会在启动时按照他们出现的顺序进行调用。
    注意顺序会改变SCSI控制器的检测顺序，从而导致硬盘数据损害
    
### obj-m

    列举了哪些文件要编译成可装载模块
    一个模块可以由一个文件或多个文件编译而成。
    一个文件直接加$(obj-m)
    多个文件，需要声明编译的模块。通过变量$(<module_name>-objs)声明哪些文件在
    让 KbuildMakefile 可以通过使用 CONFIG_符号来判断该对象是否是用来组合对象的
    
### lib-y

    其中所列的文件来组成目录下的一个库文件。
    在obj-y和lib-y中同时出现的文件，因为都是可以访问的，所以该文件不会被包含在库文件中。
    相同情况的lib-m中的文件就会包含在lib.a库文件中
    build-in.o和lib.a可以同时出现在一个目录里面
    
### 访问子目录
    
    一个Makefile只对编译所在的目录对象负责。在子目录中的文件的编译要由其所在的子目录的MakeFile来管理。
    通过obj-$(CONFIG_MODULE_NAME) += module_dirctory/
    
### 编辑标志

    EXTRA_变量只在所定义的Kbuild Makefile中起作用
    
    $(EXTRA_CFLAGS) 是用 $(CC) 编译 C 源文件时的选项。
    $(EXTRA_AFLAGS)也是一个针对每个目录的选项，只不过它是用来编译汇编源代码的。
    $(EXTRA_LDFLAGS) 和 $(EXTRA_ARFLAGS)分别与 $(LD)和 $(AR)类似，只不过，他们是针对每个目录的。
    
    $(CFLAGS_$@) 是 $(CC) 针对每个文件的选项。$@ 表明了具体操作的文件。
    $(AFLAGS_$@) 也类似，只不是是针对汇编语言的。
    
### 其他

    跟踪依赖
        
        1、所有要参与编译的文件（.c和.h)文件
        2、在参与编译文件中所要使用的CONFIG_选项
        3、用于编译目标的命令行
        
        
    特殊依赖
        
        $(src)表明 Makefile所在目录的相对路径。经常在定位源代码树中的文件时，使用该变量。
        $(obj) 表明目标文件所要存储目录的相对路径。经常在定位所生成的文件时，使用该变量。
        
    支持的函数
    as-option，当编译汇编文件(*.S)时，用来检查 $(CC) 是否支持特定选项。如果第一个选项不支持的话，可选的第二个选项可以用来指定。
    ld-option，当联接目标文件时，用来检查 $(CC) 是否支持特定选项。如果第一个选项不支持的话，可选的第二个选项可以用来指定。
    cc-option，用来检查 $(CC) 是否支持特定选项,并且不支持使用可选的第二项。
    cc-option-yn,用来检查 gcc 是否支持特定选项，返回'y'支持，否则为'n'。
    
    cc-option-align  : gcc 版本大于 3.0时，改变了函数，循环等用来声明内存对齐的选项。当用到对齐选项时，$(cc-option-align) 用来选择正确的前缀
    cc-version 以数学形式返回 $(CC)编译器的版本号。其格式是：<major><minor>，二者都是数学。
    cc-ifversion 测试 $(CC) 的版本号，如果版本表达式为真，就赋值为最后的参数。    
    
## 本地程序支持
Kbuild将编译分成了2个阶段
1. 告诉Kbuild存在哪些可执行文件。通过变量hostprogs-y来完成的。
2. 添加一个对可执行文件的显性依赖。（通过增加依赖关系到一个规则中，或是利用$(always)）

$(<executeable>-objs) 列出了联接成最后的可执行文件所需的所有目标文件。

### 定义共享库
扩展名为 so 的文件称为共享库，被编译成位置无关对象。

共享库文件经常要求一个相应的 -objs，

程序经常是利用$(HOSTCC)编译,其选项在 $(HOSTCFLAGS)变量中。可通过使用变量 HOST＿EXTRACFLAGS，影响所有在 Makefile 文件中要创建的主机程序。

## 构建Makefile

Kbuild 执行的几个步骤
1. 根据内核配置生产文件.config
2. 将内核的版本号存储在include/linux/version.h
3. 生产指向include/asm-$(ARCH)的符号链接
4. 更新所有编译所需的文件：附加的文件由arch/$(ARCH)/Makefile指定
5. 递归向下访问所有在下列变量中列出的目录 init-* core* drivers-* net-* libs-*,并编译生成目标文件。这些变量的值可以再arch/$(ARCH)/Makefile中扩充。
6. 链接所有的目标文件，在源代码树顶层目录中生成vmlinux。最先链接是在head-y中列出的文件，该变量由arch/$(ARCH)/Makefile赋值
7. 最后完成具体架构的特殊要求，并生成最终的启动镜像（ 包含生成启动指令， 准备 initrd 镜像或类似文件）

### 针对具体架构的调整

    LDFLAGS 一般是$(LD)选项
    LDFLAGS_Module 链接模块时的链接器选项，在链接模块.ko文件使用
    LDFLAGS_vmlinux 用来链接vmlinux时使用
    
    OBJCOMPYFLAGS objcopy选项
    $(call if_changed,objcopy) 经常被用来为 vmlinux 生成原始的二进制代码
    
    AFLAGS $(AS) 汇编编译器选项
    
    CFLAGS $(CC) 编译器选项
    
    $(CFLAGS_KERNEL) 包含了用于编译常驻内核代码的附加编译器选项。
    
    $(CFLAGS_MODULE) 包含了用于编译可装载模块的附加编译器选项。
    
    archprepare 规则在递归访问子目录之前，列出编译目标文件所需文件。
    
    
### 自定义kbuild命令
    
        当 Kbuild 的变量 KBUILD_VERBOSE 为 0 时，只会显示命令的简写。
        如果要为自定义命令使用这一功能，需要设置 2 个变量：
        quiet_cmd_<command> - 要显示的命令
        cmd_<command> - 要执行的命令
