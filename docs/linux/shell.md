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
                    +:  指定变量的属性
                    -:  取消变量所设的属性
                    -a:  一个使名称索引的数组（如果支持的话）
                    -A:  一个使名称关联数组（如果支持）
                    -i:  “整数”属性
                    -l:  to convert NAMEs to lower case on assignment
                    -r:  名字只读
                    -t:   to make NAMEs have the 'trace' attribute
                    -u:  to convert NAMEs to upper case on assignment
                    -x:   to make NAMEs export
    
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
                            echo $1 （param1：第1个参数）
                            echo $2 （param2：第2个参数）
                            echo $3 （param3：第3个参数）
                            ......
    （3）$#：传递给脚本或函数的参数个数。
                    ~]# vi test.sh
                            echo $#
                    ~]# ./test.sh param1 param2 param3 ......
                            3
    （4）$*：传递给脚本或函数的所有参数。横向显示。
                    ~]# vi test.sh
                           for var in "$*"
                           do
                               echo "$var"
                           done
                    ~]# ./test.sh param1 param2 param3 ......
                            param1 param2 param3 ....
    （5）$@：传递给脚本或函数的所有参数。竖向显示。
                    ~]# vi test.sh
                           for var in "$@"
                           do
                               echo "$var"
                           done
                    ~]# ./test.sh param1 param2 param3 ......
                            param1
                            param2
                            param3
                            ...
    （6）$?：上个命令的执行状态，或函数的返回值。
                    成功：0
                    失败：1-255
    （7）$$：当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。
                    ~]# vi test.sh
                           echo $$
                    ~]# ./test.sh 
                            14781
    
    ===============================================
    Shell替换
    ===============================================
    转义字符：
        （1）\\  反斜杠
        （2） \a  警报，响铃
        （3） \b  退格（删除键）
        （4） \f  换页(FF)，将当前位置移到下页开头
        （5） \n  换行
        （6） \r  回车
        （7） \t  水平制表符（tab键）
        （8） \v  垂直制表符
    
    命令替换：
        （1）：`command` 反引号
        （2）：$(command)
    
    变量替换：
        （1）${var}  变量本来的值
        （2）${var:-word}    如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。
        （3）${var:=word}    如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。
        （4）${var:?message} 如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。若此替换出现在Shell脚本中，那么脚本将停止运行。
        （5）${var:+word}    如果变量 var 被定义，那么返回 word，但不改变 var 的值。
    
    ===============================================
    Shell运算符
    ===============================================
     条件测试：
        （1） test  EXPRESSION 
        （2）[ EXPRESSION ]
        （3）[[ EXPRESSION ]]                       
    
    算术运算符
        （1）+：加法
        （2）-：减法
        （3）*：乘法
        （4）/：除法
        （5）%：取余
        （6）=：赋值
        （7）==：相等
        （8）!=：不相等
    
        计算方式：
            （1）expr
                    expr $a + $b
                    $(expr $a + $b)
                    `expr $a + $b`   （注意：这是反引号）
                    注意：符号号两边有空格
            （2）$[$a+$b]
            （3）let
                    let "c = $a + $b"  或者  let "c=$a+$b"  （注意：这是双引号）
                    let c=$a+$b （ 注意：符号号两边没有空格）
            （4）$((expression))
                    c=$(( $a + $b ))
                    备注：空格可有可无
    
    数字运算符：
        （1）-eq   是否等于  
        （2）-ne   是否不等于
        （3）-gt   是否大于
        （4）-lt   小于    
        （5）-ge  大于等于
        （6）-le   是否小于等于 
    
    布尔运算符：
            （1）-not  非运算。
            （2）-o  或运算。
            （3）-a  与运算。
            （4）! 非运算。
            （5）&& 与运算。
            （6）||  或运算。
    
    字符串运算符：
        （1）=  相等。
        （2）!= 不相等。
        （3）-z   判断字符串是否为空
        （4）-n   判断字符串是否不为空
        （5）str： 检测字符串是否为空。
    
    文件测试运算符：
            -b file： 检测文件是否是块设备文件，
            -c file： 检测文件是否是字符设备文件，
            -d file： 检测文件是否是目录，
            -f file： 检测文件是否是普通文件（既不是目录，也不是设备文件），
            -g file： 检测文件是否设置了 SGID 位，
            -k file： 检测文件是否设置了粘着位(Sticky Bit)，
            -p file： 检测文件是否是具名管道，
            -u file： 检测文件是否设置了 SUID 位，
            -r file： 检测文件是否可读，
            -w file： 检测文件是否可写，
            -x file： 检测文件是否可执行，
            -s file： 检测文件是否为空（文件大小是否大于0），不为空返回 true。
            -a file： 检测文件（包括目录）是否存在，（-a 等同于 -e）
            -e file： 检测文件（包括目录）是否存在，
            -N  file   文件自从上一次读操作之后，是否被改过 
            -O  file 当前用户是否为文件的属主
            -G  file  当前用户是否为文件的属组
    
          双目测试
            FILE1 -ef FILE2  是否指向同一个文件系统的相同inode的硬链接
            FILE1  -nt FILE2  FILE1文件，是否新于FILE2
            FILE1 -ot  FILE2  FILE1文件，是否旧于FILE2
    
    其他运算符：
        ?: 三元运算符
        =~  左侧字符串是否能够被右侧的PATTERN所匹配 （说人话  包含）
    
    ---------------------------------------------------------------------------------------
    let 与 expr 语法详解
    ---------------------------------------------------------------------------------------
    let：评估算术表达式
            id++, id--  variable post-increment, post-decrement
            ++id, --id  variable pre-increment, pre-decrement
            -, +        unary minus, plus
            !, ~        logical and bitwise negation
            **      exponentiation
            *, /, %     multiplication, division, remainder
            +, -        addition, subtraction
            <<, >>      left and right bitwise shifts
            <=, >=, <, >    comparison
            ==, !=      equality, inequality
            &       bitwise AND
            ^       bitwise XOR
            |       bitwise OR
            &&      logical AND
            ||      logical OR
            expr ? expr : expr
                    conditional operator
            =, *=, /=, %=,
            +=, -=, <<=, >>=,
            &=, ^=, |=  assignment
    ---------------------------------------------------------------------------------------
    expr：评价表达式 
           ARG1 | ARG2       ARG1 if it is neither null nor 0, otherwise ARG2
           ARG1 & ARG2  ARG1 if neither argument is null or 0, otherwise 0
           ARG1 < ARG2    ARG1 is less than ARG2
           ARG1 <= ARG2  ARG1 is less than or equal to ARG2
           ARG1 = ARG2   ARG1 is equal to ARG2
           ARG1 != ARG2   ARG1 is unequal to ARG2
           ARG1 >= ARG2  ARG1 is greater than or equal to ARG2
           ARG1 > ARG2  ARG1 is greater than ARG2
           ARG1 + ARG2  arithmetic sum of ARG1 and ARG2
           ARG1 - ARG2  arithmetic difference of ARG1 and ARG2
           ARG1 * ARG2  arithmetic product of ARG1 and ARG2
           ARG1 / ARG2  arithmetic quotient of ARG1 divided by ARG2
           ARG1 % ARG2  arithmetic remainder of ARG1 divided by ARG2
           STRING : REGEXP  anchored pattern match of REGEXP in STRING
           match STRING REGEXP     same as STRING : REGEXP
           substr STRING POS LENGTH  substring of STRING, POS counted from 1
           index STRING CHARS   index in STRING where any CHARS is found, or 0
           length STRING   length of STRING
           + TOKEN
                  interpret TOKEN as a string, even if it is a
                  keyword like 'match' or an operator like '/'
           ( EXPRESSION )
                  value of EXPRESSION
    ---------------------------------------------------------------------------------------
    
    
    
    ===============================================
    Shell注释
    ===============================================
    以“#”开头的行就是注释，会被解释器忽略。
    
    ===============================================
    Shell字符串
    ===============================================
    单引号：str='this is a string'
        注意：
        单引号里的任何字符都会原样输出。
    双引号：your_name="qinj"
        注意：
            双引号里变量正常输出
    ---------------------------------------------------------------------------------------
    字符串切片：${var:offset:number}
        示例：
            1：取字符串的子串
                ~]# vi bash.sh
                    var='abcdefg'
                    echo ${var:3}
                ~]# ./bash.sh
                ~]# defg
            2：${var:  -length}：取字符的最右侧的几个字符。
                ~]# vi bash.sh 
                    var='abcdefg'
                    echo ${var: -3}   #注意：冒号后必须有一个空白字符
                ~]# ./bash.sh
                ~]# efg
            3：从左向右截取某字符后几位
                ~]# vi bash.sh
                    var='abcdefg'
                    echo ${var:2:2}
                ~]# ./bash.sh
                ~]# cd
            4：从右向左截取某字符后几位
                ~]# vi bash.sh
                    var='abcdefg'
                    echo ${var: -4:2}
                ~]# ./bash.sh
                ~]# de
    ---------------------------------------------------------------------------------------
    基于模式取子串：
        1：${var#*word}：删除字符串开头至此分隔符之间的所有字符。
                示例：
                    ~]# vi bash.sh
                        var='abc/de/fg'
                        echo ${var#*/}
                    ~]# ./bash.sh 
                        de/fg
    
        2：${var##*word}：删除字符串开头至此分隔符之间的所有字符；
            示例：
                ~]# vi bash.sh 
                    var='abc/de/fg'
                    echo ${var##*/}
                ~]# ./bash.sh 
                    fg
    
        3：${var%word*}：删除此分隔符至字符串尾部之间的所有字符；
            示例：
                ~]# vi bash.sh 
                    var='abc/de/fg'
                    echo ${var%/*}
                ~]# ./bash.sh 
                    abc/de
    
        4：${var%%word*}：删除此分隔符至字符串尾部之间的所有字符；
                示例：
                    ~]# cat bash.sh 
                            var='abc/de/fg'
                            echo ${var%%/*}
                    ~]# ./bash.sh 
                            abc
    ---------------------------------------------------------------------------------------
    查找替换：（PATTERN中使用glob风格和通配符）
        1：${var/PATTERN/SUBSTI}：查找var所表示的字符串中，第一次被PATTERN所匹配到的字符串，将其替换为SUBSTI所表示的字符串；
                示例：
                    ~]# vi ./bash.sh 
                        var='aaabbbcccaaabbbccc'
                        echo ${var/bbb/字符串}
                    ~]# ./bash.sh 
                        aaa字符串cccaaabbbccc
    
        :2：${var//PATTERN/SUBSTI}：查找var所表示的字符串中，所有被PATTERN所匹配到的字符串，并将其全部替换为SUBSTI所表示的字符串；
            示例：
                ~]# vi bash.sh 
                    var='aaabbbcccaaabbbccc'
                    echo ${var//bbb/字符串}
                ~]# ./bash.sh 
                    aaa字符串cccaaa字符串ccc
    
        3：${var/#PATTERN/SUBSTI}：查找var所表示的字符串中，行首被PATTERN所匹配到的字符串，将其替换为SUBSTI所表示的字符串；
            示例：
                ~]# vi bash.sh 
                    var='aaabbbcccaaabbbccc'
                    echo ${var/#aa/字符串}
                ~]# ./bash.sh 
                    字符串abbbcccaaabbbccc
    
        4：${var/%PATTERN/SUBSTI}：查找var所表示的字符串中，行尾被PATTERN所匹配到的字符串，将其替换为SUBSTI所表示的字符串；
            示例：
                ~]# vi bash.sh 
                    var='aaabbbcccaaabbbccc'
                    echo ${var/%cc/字符串}
                ~]# ./bash.sh 
                    aaabbbcccaaabbbc字符串
    ---------------------------------------------------------------------------------------
    查找删除：
        1：${var/PATTERN}：删除第一次的匹配；
            示例：
                ~]# vi bash.sh 
                    var='aaabbbcccaaabbbccc'
                    echo ${var/cc/字符串}
                ~]# ./bash.sh 
                    aaabbb字符串caaabbbccc
    
        2：${var//PATERN}：删除最后一次匹配
            示例：
            ~]# vi bash.sh 
                var='aaabbbcccaaabbbccc'
                echo ${var//cc/字符串}
            ~]# ./bash.sh 
                aaabbb字符串caaabbb字符串c
    
        3：${var/#PATTERN}：删除行首匹配
            示例：
                ~]# cat bash.sh 
                    var='aaabbbcccaaabbbccc'
                    echo ${var/#aa/字符串}
                ~]# ./bash.sh 
                    字符串abbbcccaaabbbccc
    
        4：${var/%PATTERN}：删除行尾匹配
            示例：
                ~]# cat bash.sh 
                    var='aaabbbcccaaabbbccc'
                    echo ${var/%cc/字符串}
                ~]# ./bash.sh 
                    aaabbbcccaaabbbc字符串
    ---------------------------------------------------------------------------------------
    字符大小写转换：
        1：${var^^}：所有字符转换为大写；
            示例：
                ~]# cat bash.sh 
                    var='aaabbbcccAAABBBCCC'
                    echo ${var^^}
                ~]# ./bash.sh 
                    AAABBBCCCAAABBBCCC
    
        :2：${var,,}：所有字符转换为小写；
            示例：
                ~]# cat bash.sh 
                    var='aaabbbcccAAABBBCCC'
                    echo ${var,,}
                ~]# ./bash.sh 
                    aaabbbcccaaabbbccc
    ---------------------------------------------------------------------------------------
    变量赋值：
        1：${var:-VALUE}：如果var变量为空，或未设置，那么返回VALUE；否则，则返回var变量的值； 
            示例：
                ~]# cat bash.sh 
                    var='字符串'
                    echo ${var:-string}
                    echo ${ar:-string}
                ~]# ./bash.sh 
                    字符串
                    string
    
        2：${var:=VALUE}：如果var变量为空，或未设置，那么返回VALUE，并将VALUE赋值给var变量；否则，则返回var变量的值； 
            示例：
                ~]# cat bash.sh 
                    var='字符串'
                    echo ${var:=string}
                    echo ${ar:=string}
                    echo $ar
                ~]# ./bash.sh 
                    字符串
                    string
                    string
    
        3：${var:+VALUE}：如果var变量不为空，则返回VALUE；
            示例：
                ~]# cat bash.sh 
                    var='字符串'
                    echo ${var:+string}
                ~]# ./bash.sh 
                    string
    
        4：${var:?ERROR_INFO}：如果var为空，或未设置，那么返回ERROR_INFO为错误提示；否则，返回var值； 
            示例：
                ~]# cat bash.sh 
                    var='字符串'
                    echo ${var:?string}
                    echo ${ar:?错误信息}
                ~]# ./bash.sh 
                    字符串
                    ./bash.sh: line 4: ar: 错误信息
    ---------------------------------------------------------------------------------------
    拼接字符串
        示例：
            ~]# vi bash.sh
                your_name="qinjx"
                greeting="hello, "$your_name" !"
                greeting_1="hello, ${your_name} !"
                echo $greeting $greeting_1
            ~]# ./bash.sh
                hello, qinjx ! hello, qinjx !
    ---------------------------------------------------------------------------------------
    获取字符串长度：${#string}
        示例：
            ~]# vi bash.sh 
                string='abcd'
                echo ${#string}
                echo $(expr length $string)
            ~]# ./bash.sh 
                4
                4
    ===============================================
    Shell if else语句
    ===============================================
    （1）单分支1
            if CONDITION ; then
                分支
            fi
    （2）单分支2
            if CONDITION ; then
                分支1
            else
                分支2
            fi
    （3）多分支1
            if CONDITION1; then
                    分支1
            elif  CONDITION2; then
                    分支2
                    ...
            elif CONDITION; then
                    分支n
            fi
    （4）多分支2
            if CONDITION1; then
                    分支1
            elif  CONDITION2; then
                    分支2
                    ...
            else CONDITION; then
                        分支n
            fi
    
    语法：then可换行写，condition后面就不用加分号。
            if CONDITION
            then
                分支
            fi
    ===============================================
    Shell case esac语句
    ===============================================
    （1）语法结构
            case  $VARAIBLE  in  
            PAT1)
                分支1
                ;;
            PAT2)
                分支2
                ;;
            ...
            *)
                分支n
                ;;
            esac
    
    示例：
        value='c'
        case $value in
        a)
            echo '这是a' # 可以是单引号或者双引号
            ;;
        b)
            if [0 -le 18];then  # 可以是一段代码
                echo '一个判断'
            fi
            ;;
        c)
            echo 这是c
            ;; # 必须以;;为结束
        *)
            echo "未匹配到上面的结果,在此可以执行一段代码或者不写 *) 这一部分"
            ;;
        esac
        
        结果：
            这是c
    ===============================================
    Shell for循环
    ===============================================
    （1）语法结构
        for 变量 in 列表; do
            command
        done
    
    示例：
        1：详细输出，依次输出1,2,3,4,5 
            for File in 1 2 3 4 5 
            do 
                echo $File 
            done
        2：输出某个目录下所有的文件或者匹配到的文件
            for file in $HOME/.bash*; do        # 也可以写 $HOME/*
                echo $file
            done
        3：{开始正整数..结束正整数}：注意：中间是两个点
            for num in {1..5}; do
                echo $num           # 输出数字 1-5 的正整数
            done
        4：((i=1; i<=5; i++ ))：每次循环加1
            for((i=1; i<=5; i++ )); do
                echo $i          # 输出数字 1-5 的正整数
            done
        5：执行命令 seq：起始从1开始
            for i in $(seq 5); do
                echo $i          # 输出数字 1-5 的正整数
            done
        6：执行命令
            for i in $(ls /); do
                echo $i
            done
    
    语法：do可以换行写，那里就不需要分号了。
    ===============================================
    Shell while循环
    ===============================================
    （1）语法结构
            while  CONDITION; do
                代码....
            done
    
            注意：
            进入条件： CONDITION 测试为”真“
            退出条件： CONDITION 测试为”假“
    
    示例：
    （1）
        declare -i i=1
        while [ $i -le 5 ]; do
            echo $i
            let i++
        done
        或者
        declare -i i=1
        while (( $i <= 5 )); do
            echo $i
            let i++
        done
    
    ===============================================
    Shell until循环
    ===============================================
    （1）语法结构
            until  CONDITION; do
                循环体
                循环控制变量修正表达式
            done
    
            注意：
            进入条件：CONDITION测试为”假“
            退出条件：CONDITION测试为”真“    
    
    示例：
        declare -i i=1
        until (( $i > 5 )); do
            echo $i
            let i++
        done
        或者
        declare -i i=1
        until [ $i -gt 5 ]; do
            echo $i
            let i++
        done
    ===============================================
    Shell跳出循环
    ===============================================
    （1）continue：跳过本次循环，执行下一次循环
    （2）break：退出循环
    
    ===============================================
    Shell函数
    ===============================================
    （1）语法一：
            function  f_name  {
                ...函数体...
            }
                
    （2）语法二：
            f_name()  {
                ...函数体...
            }
    
    执行函数：
        f_name
    
        示例：
            1：    #函数的声明
                    function fname {
                        echo '我是语法一函数'
                    }
                    # 执行函数
                    fname
    
            2：   # 函数的声明
                fname(){
                    echo '我是语法二函数'
                }
                # 执行函数
                fname
    
    参数：
        （1）$1，$2, ...
        （2）$#   $@ $*
            
            示例：
                    function fname {
                        echo "第一个参数：$1"
                        echo "第二个参数：$2"
                        # echo "第N个参数：$n"
                        echo "参数总数：$#"
                        echo "参数字符串: $@"
                        echo "参数字符串: $*"
                    }
                    #执行函数，并传入参数
                    fname 1 2
                结果：
                    第一个参数：1
                    第二个参数：2
                    参数总数：2
                    参数字符串: 1 2
                    参数字符串: 1 2
    
    return：返回退出状态码，shell不退出
        示例：
            function fname {
                return 1
            }
            fname
            echo $?
        结果：
            1
    
    exit：返回状态码并推出
    
    作用域：
        （1）全局作用域：没有使用 local 关键字；语法：VARIABLE=VALUE
        （2）局部作用域：函数内部声明并且使用 local 关键字，仅在函数内部有效：语法：local VARIABLE=VALUE
    
            示例：
                name=tom    # 全局作用域
                setname() {
                    local name=jerry        # 局部作用域,仅在函数内部有效。
                        echo "局部: $name"
                }
                setname         #执行函数
                echo "全局: $name"
    ===============================================
    Shell数组
    ===============================================
    声明数组：
            declare  -a  NAME：声明索引数组；
            declare  -A  NAME：声明关联数组；
    
    数组中元素的赋值方式：
                (1) 一次只赋值一个元素；
                    ARRAY_NAME[INDEX]=value
                    示例：索引数组
                        declare -a index_array;
                        index_array[0]=0
                        index_array[1]=1
                        echo ${index_array[0]}
                    示例：关联数组
                        declare -A array_name
                        array_name[aa]=aaaa
                        array_name[bb]=bbbb
                        echo ${array_name[aa]}
                (2) 一次赋值全部元素；
                    ARRAY_NAME=("VAL1"  "VAL2"  "VAL3"  ...)
                    示例：索引数组
                        declare -a index_array;
                        index_array=('val0' 'val1' 'val2')
                        echo ${index_array[0]}
    
                (3) 只赋值特定元素；
                    ARRAY_NAME=([0]="VAL1"  [3]="VAL4" ...)
                    示例：索引数组
                        declare -a index_array;
                        index_array=([0]='val0' [3]='val1' [6]='val2')
                        echo ${index_array[6]}
                    示例：关联数组
                        declare -A array_name
                        array_name=([aa]='aaaa' [bb]='bbbb')
                        echo ${array_name[aa]}
    
    
                    注意：bash支持稀疏格式的数组；
    
                (4) read  -a  ARRAY_NAME
                    示例：
                        read -p "输入参数: " -a array_name
                        echo ${array_name[0]}
    
    
           （5） 引用数组中的元素：${ARRAY_NAME[INDEX]}
            注意：引用时，只给数组名，表示引用下标为0的元素；
    
            （6）数组的长度（数组中元素的个数）:
                ${#ARRAY_NAME[*]}
                ${#ARRAY_NAME[@]}
                示例：
                    declare -a array_name
                    array_name[0]=00000
                    array_name[1]=1111
                    array_name[2]=2222
                    echo ${#array_name[@]}
                    echo ${#array_name[*]}
                    结果：
                        3
                        3
    
            （7）数组的参数（数组中所有的参数）：
                ${ARRAY_NAME[*]}
                ${ARRAY_NAME[@]}
                示例：
                    declare -a array_name
                    array_name[0]=00000
                    array_name[1]=1111
                    array_name[2]=2222
                    echo ${array_name[@]}
                    echo ${array_name[*]}
                结果：
                    00000 1111 2222
                    00000 1111 2222
    ===============================================
    Shell输入输出重定向
    ===============================================
    推荐：http://www.cnblogs.com/chengmo/archive/2010/10/20/1855805.html
    
    linux shell下常用输入输出操作符是：
    1.  标准输入   (stdin) ：代码为 0 ，使用 < 或 << ； /dev/stdin -> /proc/self/fd/0   0代表：/dev/stdin 
    2.  标准输出   (stdout)：代码为 1 ，使用 > 或 >> ； /dev/stdout -> /proc/self/fd/1  1代表：/dev/stdout
    3.  标准错误输出(stderr)：代码为 2 ，使用 2> 或 2>> ； /dev/stderr -> /proc/self/fd/2 2代表：/dev/stderr
    
    输出重定向：
        1：重定向程序正常执行的结果
        覆盖重定向：覆盖目标文件中的原有内容；
            COMMAND >  /PATH/TO/SOMEFILE
        追加重定向：追加新产生的内容至目标文件尾部；
            COMMAND >> /PATH/TO/SOMEFILE
    
        shell的一个功能开关：
            # set -C
                禁止覆盖输出重定向至已存在的文件；
                注意：此时仍然可以使用“>|”至目标文件； 
            # set +C
                关闭上述特性；
                            
    错误重定向：
                    重定向错误的执行结果；
                    
                        COMMAND 2>  /PATH/TO/SOMEFILE
                            错误输出覆盖重定向；
                        COMMAND 2>> /PATH/TO/SOMEFILE
                            错误输出追加重定向；
                            
    合并标准输出与错误输出流：
                    (1) &>, &>>
                    (2) COMMAND > /PATH/TO/SOMEFILE 2>&1
                          COMMAND >> /PATH/TO/SOMEFILE 2>&1
                          
                特殊输出目标：/dev/null
                    位桶：bit bucket
                特殊的输入文件：/dev/zero
    
    输入重定向：
                COMMAND < /PATH/FROM/SOMEFILE
                COMMAND << ：
                Here Document
    
                用法：
                    COMMAND << EOF
                    COMMAND > /PATH/TO/SOMEFILE << EOF
    ===============================================
    Shell文件包含  . 与 source
    ===============================================
    文件包含：. （点） 与 source 都可以引入文件
    注意：
        1：被引入的文件不需要执行权限
        2：可以没有 #!/bin/bash
        3：被引入程序当做一个可执行的脚本运行。
    
    示例：
    ~]# vi bash.sh 
        #!/bin/bash
        # 引入 config.sh 文件
        . ./config.sh
        # 引入 cfg 文件
        source ./cfg
        echo $string
        echo $cfg
    ~]# vi cfg
        cfg='cfg文件'
    ~]# vi config.sh 
        #!/bin/bash
        string='config文件'
    ~]# ./bash.sh
        config文件
        cfg文件
