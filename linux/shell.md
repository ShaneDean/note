参考：http://blog.csdn.net/u010861514/article/details/51028220

留作记录，方便以后查询

    shell 基础语法
    ===============================================
    推荐：http://c.biancheng.net/cpp/shell/
    ===============================================
    第一个Shell脚本
    ===============================================
    格式：
        （1）扩展名为sh（sh代表shell）
                    例如：bash.sh
        （2）文件头（文件头固定格式）
                    如：#!/bin/bash
    
    作为可执行程序：
        （1）使脚本具有执行权限
            ~]# chmod +x ./test.sh
        （2）执行脚本
                1：可以不加执行权限
                    ~]$ bash abc.sh     （直接指明文件）
                    ~]$ bash ./abc.sh    （相对路径）
                    ~]$ bash /home/mdx/abc.sh （绝对路径）
                2：必须加执行权限 （必须是绝对路径或者相对路径）
                    ~]# ./test.sh   （相对路径）
                    ~]$ /home/mdx/test.sh  （绝对路径）
    
    解释器参数（read）：
            使用 read 命令从 stdin 获取输入并赋值给 PERSON 变量，最后在 stdout 上输出：
            read   [option]  ...  A B
               -p 'PROMPT'  提示符
                    ~] read -p "提示符："     （接受其传入的参数）
               -t    TIMEOUT
    
    ===============================================
    Shell变量
    ===============================================
    定义：
        （1）只能使用字母、数字和下划线；而且不能以数字开头。
        （2）不能使用标点符号。
        （3）不能使用bash里的关键字（可用help命令查看保留关键字）。
    
    变量赋值：
        （1）NAME=VALUE
                =：赋值符号，两边不能有空格。把VALUE存储到NAME指向的内存空间中
        （2）declare  命令
                    declare [-aAfFgilrtux] [-p] [name[=value] ...]
                    +：指定变量的属性
                    -：取消变量所设的属性
                    -a：一个使名称索引的数组（如果支持的话）
                    -A：一个使名称关联数组（如果支持）
                    -i：“整数”属性
                    -l： to convert NAMEs to lower case on assignment
                    -r：名字只读
                    -t    to make NAMEs have the `trace' attribute
                    -u：to convert NAMEs to upper case on assignment
                    -x    to make NAMEs export
    
    重新赋值变量：
        （1）NAME=VALUE（再次重新赋值）
    
    变量引用：
        （1）${NAME}
        （2）$NAME
    
    只读变量：
        （1）readonly NAME（变量赋值后定义）
    
    删除变量：
        （1）unset NAME
    
    变量类型：
    （1) 局部变量
        局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
    （2) 环境变量
        所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
    （3) shell变量
        shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行
    
    ===============================================
    Shell特殊变量
    ===============================================
    （1）$0：当前脚本的文件名
                    ~]# vi test.sh
                            echo $0
                    ~]# ./test.sh
                            ./test.sh
    （2）$n：传递给脚本或函数的参数。n 是一个数字，表示第几个参数。
                    ~]# ./test.sh param1 param2 param3 ......
                    ~]# vi test.sh

