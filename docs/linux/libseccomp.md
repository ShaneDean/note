# 说明
seccomp(全称securecomputing mode) 是linuxkernel从2.6.23版本开始所支持的一种安全机制。

在Linux系统里,大量的系统调用(systemcall)直接暴露给用户态程序。

但是,并不是所有的系统调用都被需要,而且不安全的代码滥用系统调用会对系统造成安全威胁。

通过seccomp,我们限制程序使用某些系统调用,这样可以减少系统的暴露面,同时是程序进入一种“安全”的状态。

# 编译

    git clone https://github.com/seccomp/libseccomp.git
    //自己选择 tag 或 branch 默认最新
    ./autogen.sh            //需要autoconf automake
    ./configure
    make V=1
    make install
    make check
    
    在构建完成之后，doc/man目录下面有可以参考的手册，通过man xxxx 来查询详细的使用用法

    其他的一些参考资料
    Documentation/prctl/seccomp_filter.txt
    https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt
    http://man7.org/linux/man-pages/man2/seccomp.2.html



--------



# 项目构建

libseccomp是由 automake 组织构建的。

先看 configure.ac文件

```

AC_INIT([libseccomp], [2.3.3])      //定义了包名和版本号

AC_CONFIG_AUX_DIR([build-aux])      //辅助工具的存放路径

AC_CONFIG_HEADERS([configure.h])    //宏定义变量的变量集合

AC_CONFIG_MACRO_DIR([m4])           //指定本地autoconf宏的位置,m4文件夹下定义了check-code-coverage


//m4 中  define (name, [expansion])
m4_define([serial_tests], [
    //m4 中  esyscmd(shell-command)
    m4_esyscmd([automake --version |
                head -1 |
                awk '{split ($NF,a,"."); if (a[1] == 1 && a[2] >= 12) { print "serial-tests" }}'
    ])
])
//根据automake的版本去


AM_INIT_AUTOMAKE(-Wall foreign subdir-objects tar-pax serial_tests)     //增加make  的 options


AC_PROG_CC              //检查要使用的c编译器
AM_PROG_CC_C_O          //测试上面的次编译器的 -c 和 -o
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])

LT_INIT([shared pic-only])      //启用libtool

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])       //减少构建输出


AM_CPPFLAGS="-I\${top_srcdir}/include"      //cpp flags
AM_CFLAGS="-Wall"                           //c   flags
AM_LDFLAGS="-Wl,-z -Wl,relro"               //ld  flags
AC_SUBST([AM_CPPFLAGS])             //加入变量
AC_SUBST([AM_CFLAGS])
AC_SUBST([AM_LDFLAGS])


AC_CHECK_HEADERS_ONCE([linux/seccomp.h])        //检查下这个文件是否存在


//获取版本信息
VERSION_MAJOR=$(echo ${VERSION} | cut -d'.' -f 1)
VERSION_MINOR=$(echo ${VERSION} | cut -d'.' -f 2)
VERSION_MICRO=$(echo ${VERSION} | cut -d'.' -f 3)
AC_SUBST([VERSION_MAJOR])
AC_SUBST([VERSION_MINOR])
AC_SUBST([VERSION_MICRO])

// cython checks   略
// python binding checks   略
// coverity checks 略
// code coverage checks

//指明需要版本信息来生成的文件
AC_CONFIG_FILES([
	libseccomp.pc
	include/seccomp.h
])

//指明需要生成的各级Makefile
AC_CONFIG_FILES([
	Makefile
	include/Makefile
	src/Makefile
	src/python/Makefile
	tools/Makefile
	tests/Makefile
	doc/Makefile
])

```


再看 makefile.am文件

顶层目录的makefile

```
    ACLOCAL_AMFLAGS =-I m4      //需要引用的宏的地址
    SUBDIRS = include src tools tests doc   //下级 makefile.am地址
    
    pkgconfdir = ${libdir}/pkgconfig    
    pkgconf_DATA = libseccomp.pc
    
    EXTRA_DIST = CHANGELOG CREDITS LICENSE README.md SUBMITTING_PATCHES     //规范中没有定义，但是选上的文件

    
    //配置静默安装
    AM_MAKEFLAGS_0 = --quiet --no-print-directory
    AM_MAKEFLAGS_1 =
    AM_MAKEFLAGS_ = ${AM_MAKEFLAGS_0}
    AM_MAKEFLAGS = ${AM_MAKEFLAGS_@AM_V@}
    
    
    //切到src和tests中构建
    check-build: all
	${MAKE} ${AM_MAKEFLAGS} -C src check-build      
	${MAKE} ${AM_MAKEFLAGS} -C tests check-build
    
    check-syntax:
	@./tools/check-syntax       //使用astyle来优化代码格式
	
    check-code-coverage: check-build
	${MAKE} ${AM_MAKEFLAGS} -C tests check-code-coverage    
	
```


按照SUBDIRS的顺序依次进入子目录， 首先是include。


```
    include_HEADERS = seccomp.h
```


src目录


```
    //看是否开启Python，开启的话则增加PYTHON到子目录中
    SUBDIRS = .
    if ENABLE_PYTHON
    SUBDIRS += python
    endif  
    
    SOURCE_ALL = ...    //所有的源文件
    
    EXTRA_DIST = arch-syscall-validate

    TESTS = arch-syscall-check
    
    check_PROGRAMS = arch-syscall-check arch-syscall-dump   //指定系统的测试程序
    
    lib_LTLIBRARIES = libseccomp.la
    
    arch_syscall_dump_SOURCES = arch-syscall-dump.c ${SOURCES_ALL}

    arch_syscall_check_SOURCES = arch-syscall-check.c ${SOURCES_ALL}

    libseccomp_la_SOURCES = ${SOURCES_ALL}
    libseccomp_la_CPPFLAGS = ${AM_CPPFLAGS} ${CODE_COVERAGE_CPPFLAGS} \
    	-I${top_builddir}/include
    libseccomp_la_CFLAGS = ${AM_CFLAGS} ${CODE_COVERAGE_CFLAGS} ${CFLAGS} \
    	-fPIC -DPIC -fvisibility=hidden
    libseccomp_la_LDFLAGS = ${AM_LDFLAGS} ${CODE_COVERAGE_LDFLAGS} ${LDFLAGS} \
    	-version-number ${VERSION_MAJOR}:${VERSION_MINOR}:${VERSION_MICRO}
    
    check-build:
    	${MAKE} ${AM_MAKEFLAGS} ${check_PROGRAMS}       
    
```


tools的makefile.am
 

```
AM_CPPFLAGS = -I${top_builddir}/include

noinst_LTLIBRARIES = util.la        //lib_LTLIBRARIES
util_la_SOURCES = util.c util.h bpf.h
util_la_LDFLAGS = -module

    //安装的程序
bin_PROGRAMS = \
	scmp_sys_resolver
	
	//没有安装的程序
noinst_PROGRAMS = \
	scmp_arch_detect \
	scmp_bpf_disasm \
	scmp_bpf_sim
	
EXTRA_DIST = check-syntax scmp_app_inspector

scmp_bpf_disasm_SOURCES = scmp_bpf_disasm.c bpf.h util.h
scmp_bpf_sim_SOURCES = scmp_bpf_sim.c bpf.h util.h

scmp_sys_resolver_LDADD = ../src/libseccomp.la
scmp_sys_resolver_LDFLAGS = -static
scmp_arch_detect_LDADD = ../src/libseccomp.la
scmp_arch_detect_LDFLAGS = -static
scmp_bpf_disasm_LDADD = util.la
scmp_bpf_sim_LDADD = util.la


```


test中的makefile.am



```

//code_coverage_rules  略

LDADD = util.la ../src/libseccomp.la 

check_LTLIBRARIES = util.la
util_la_SOURCES = util.c util.h
util_la_LDFLAGS = -module

TESTS = regression    //主体的测试程序

check_PROGRAMS = ...

EXTRA_DIST_TESTPYTHON = ...

EXTRA_DIST_TESTCFGS = ...

EXTRA_DIST_TESTSCRIPTS = regression testdiff testgen    // 3个测试脚本

EXTRA_DIST_TESTVALGRIND = valgrind_test.supp

nodist_00_test_SOURCES = 00-test.c
EXTRA_PROGRAMS = 00-test

check-build:
	${MAKE} ${AM_MAKEFLAGS} ${check_PROGRAMS}

clean-local:
	${RM} -f 00-test *.pyc

```

doc的就略过了，  筛选出关键的外部程序或脚本

```
    regression 
    
    testdiff \ testgen   // compare test output from different test runs  

    check-syntax   //  a basic C style/format checking tool
    
    scmp_app_inspector  //a simple syscall inspector based on strace

    scmp_sys_resolver \ scmp_arch_detect \   scmp_bpf_disasm \  scmp_bpf_sim
    
    arch-syscall-validate       //a simple syscall validation tool

    
```




    
进入tests目录 

发现   [数字]-[名称].[c\py\tests] 

这是   regression脚本对上面的文件进行libseccomp的测试

其中  .c和 .py是测试的主题逻辑

.test文件则是参与测试的样例数据
 
 
其中主体的测试控制程序是 regression ， 可以通过 -h 来查看使用方法，默认不增加参数即可开启测试

regression是一个shell程序，分析其代码：

```
		run_tests
		|-->	run_test $batch_name $testnum $line $test_type
		|		//其中举例 01-sim-allow.tests
		|		//		   01-sim-allow  01	"01-sim-allow 	all	 0-350	N	N	N	N	N	N	ALLOW"  bpf-sim
		    |-->
		    		|		run_test_basic
		    		|		run_test_bpf_sim_fuzz
		    		|		run_test_bpf_valgrind
		    		|		run_test_live
		    		|-->	run_test_bpf_sim	 01-sim-allow  01	"01-sim-allow	 all	0-350	N	N	N	N	N	N	ALLOW" 
	    					|--> run_test_command "$testnumstr" "./$testname" "-b" 4 ""
				 					|-->  01-sim-allow

``` 

注意:由于参数较多，组合起来的测试方案多种多样，所以*xxxx.tests*文件中定义 range的模式，比0-300表示 0 到 300，regression会将这些表示拆开，然后分别执行，在对比日志的时候可以得到佐证。

regression的最后通过调用 run_test_command 最后进入了 c的测试程序，在进入c的测试程序之前，我们需要熟悉下libseccomp 的一些api。    

	
```	
		//初始化seccomp filter state
		scmp_filter_ctx seccomp_init(uint32_t def_action);
		int seccomp_reset(scmp_filter_ctx ctx, unit32_t def_action);
			def_action：
				SCMP_ACT_KILL , SCMP_ACT_TRAP , ACMP_ACT_ERRNO , SCMP_ACT_TRACE , SCMP_ACT_ALLOW
				
		//释放seccomp filter state  ,已经loaded into kernel不受影响
		void seccomp_release(scmp_filter_ctx ctx);
		//合并两个seccomp filter,src会被释放，不需要在调用 seccomp_release
		//filter值需要一致，架构需要重叠
		int seccomp_merge(scmp_filter_ctx dst, scmp_filter_ctx src);
		
		
		//架构管理
       uint32_t seccomp_arch_resolve_name(const char *arch_name);
       uint32_t seccomp_arch_native();
       int seccomp_arch_exist(const scmp_filter_ctx ctx, uint32_t arch_token);
       int seccomp_arch_add(scmp_filter_ctx ctx, uint32_t arch_token);
       int seccomp_arch_remove(scmp_filter_ctx ctx, uint32_t arch_token);
		
			uint32_t arch_token 由 SCMP_ARCH_* 定义的常量
			SCMP_ARCH_NATIVE 常量总是指向本地编译的架构
			当一个新的架构加进来的时候，老	的filter和它没关系，但是后面新增的filter都跟他相关。
			
		//属性管理
		int seccomp_attr_set(scmp_filter_ctx ctx,
								enum scmp_filter_attr attr, uint32_t value)
		int seccomp_attr_get(scmp_filter_ctx ctx,
								enum scmp_filter_attr attr, uint32_t *value)
		
			可选的scmp_filter_attr为
			
				SCMP_FLTATR_ACT_DEFAULT
					只读属性
				SCMP_FLTATR_ACT_BADARCH  	//def_action
					如果架构不匹配，那么默认 SCMP_ACT_KILL
				SCMP_FLTATR_CTL_NNP			//boolean
					定义NO_NEW_PRIVS在filter加载到内核之前就应该被启动。如果这个为0，那么会去检查 CAP_SYS_ADMIN，不然失败。默认1。
				SCMP_FLTATR_CTL_TSYNC		//boolean
					设置表示seccomp_load调用的时候需要全部同步filter
				SCMP_FLTATR_ATL_TSKIP		//boolean
					设置表示可以创建 -1的syscall	
		//导出seccomp filter
		int seccomp_export_bpf(const scmp_filter_ctx, int fd);		//bpf	--> Berkley Packet Filter
		int seccomp_export_pfc(const scmp_filter_ctx, int fd);		//pfc  --> Pseudo Filter Code
			
		//装载filter到kernel中
		int seccomp_load(scmp_filter_ctx ctx);  //成功的加载
		
		//增加 seccomp filter rule
		int SCMP_SYS(syscall_name);
		struct scmp_arg_cmp SCMP_CMP(unsigned int arg, enum scmp_compare op, ...);
		struct scmp_arg_cmp	SCMP_A0(enum scmp_compare op, ...);
		...
		struct scmp_arg_cmp SCMP_A5(enum scmp_compare op, ...);
		int seccomp_rule_add(scmp_filter_ctx ctx, uint32_t action ,	int syscall, unsigned int arg_cnt, ...);
		int seccomp_rule_add_exact(scmp_filter_ctx ctx, uint32_t action,	int syscall, unsigned int arg_cnt, ...);
		
		int seccomp_rule_add_array(scmp_filter_ctx ctx, uint32_t action, int syscall, unsigned int arg_cnt, const struct scmp_arg_cmp *arg_array);
		int seccomp_rule_add_exact_array(scmp_filter_ctx ctx, uint32_t action, int syscal, unsigned int arg_cnt, const struct scmp_arg_cmp *arg_array);
		
			新加入的filter rule需要load进 kernel才会生效
			SCMP_CMP（） 和 SCMP_A{0-5}()宏 生成一个 scmp_arg_cmp结构用到上面的函数中。
	
		
			//区分 seccomp filter 中的 syscall
			int seccomp_syscall_priority(scmp_filter_ctx ctx, int syscall, uint8_t priority);
			//解析syscall名称
			int seccomp_syscall_resolve_name(const char *name);
			int seccomp_syscall_resolve_name_arch(uint32_t arch_token, const char *name);
			int seccomp_syscall_resolve_name_rewrite(uint32_t arch_token, const char *name);
			char *seccomp_syscall_resolve_num_arch(uint32_t arch_torken, int num);
```		
		
				
				 					
进入 01-sim-alloc.c 代码	

```
	...
	ctx = seccomp_init(SCMP_ACT_ALLOW);
	...
	rc = util_filter_output(&opts,ctx);
	     |--> _ctx_valid(ctx)
	     |--> program = gen_bpf_generate((struct db_filter_col *) ctx);
	            |--> _gen_bpf_build_bpf(stcut bpf_state *state,const struct db_filter_col *col)
	            			|--> ???
	
``` 


