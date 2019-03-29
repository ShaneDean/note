只使用autoconf来构建

    your source files --> [autoscan*] --> [configure.scan] --> configure.ac
     
     configure.ac --.
                    |   .------> autoconf* -----> configure
     [aclocal.m4] --+---+
                    |   `-----> [autoheader*] --> [config.h.in]
     [acsite.m4] ---'
     
     Makefile.in
     
如果增加使用automake来构建

     [acinclude.m4] --.
                      |
     [local macros] --+--> aclocal* --> aclocal.m4
                      |
     configure.ac ----'
     
     configure.ac --.
                    +--> automake* --> Makefile.in
     Makefile.am ---'
     
     
 用到配置软件包中的文件
 
    图：
                            .-------------> [config.cache]
     configure* ------------+-------------> config.log
                            |
     [config.h.in] -.       v            .-> [config.h] -.
                    +--> config.status* -+               +--> make*
     Makefile.in ---'                    `-> Makefile ---'



# automake

## 标准的makefile目标

    all         默认执行的目标，编译文件
    install     安装文件，兵拷贝包内文件
    install-trip    没有调试信息的install
    uninstall   卸载文件
    clean       清空 make all的文件
    distclean   在clean的基础上再清除 ./configure创建的文件
    check       跑测试套件
    installcheck     检查已安装的程序或库文件
    dist        用源码重新创建package-version.tar.gz文件
    
## 标准的安装步骤

    ./configure 
    make
    make install
    
## 标准目录变量

    prefix          /usr/local
    bindir          ${prefix}/bin
    libdir          ${prefix}/lib
    includedir      ${prefix}/include
    datarootdir     ${prefix}/share
    mandir          ${datarootdir}/man
    infodir         ${datarootdir}/info
    docdir          ${datarootdir}/doc/${PACKAGE}
    
## 标准配置变量

    CC              c编译器的命令
    CFLAGS          c编译器的参数
    CXX             c++编译器命令
    CXXFLAGS        c++编译器参数
    LDFLAGS         链接器的参数
    CPPFLAGS        c/c++预处理器参数
