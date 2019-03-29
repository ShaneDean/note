# 开发环境

## 基础环境

### 安装Centos7
![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/version.png?raw=true)


### 安装jdk

下载最新[jdk8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html/) 

选择jdk-xxxx-linux-x64.rpm版本

    sudo rpm -ivh jdk-xxxx-linux-x64.rpm

修改JAVA环境变量

```
vim /etc/profile

//在最后加上，然后wq!
export JAVA_HOME=/usr/java/jdk1.8.0_144
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

//启用配置
source /etc/profile

//此时替换掉原有的centos7 java版本 选择你下载的版本
sudo alternatives --config java

//最后
java -version
```
    
### 配置maven3.5.0

[参考](http://www.cnblogs.com/HendSame-JMZ/p/6122188.html) 
    
### 安装git

    sudo yum install git

速度太慢 替换使用阿里源 [参考](http://mirrors.aliyun.com/help/centos)

ht内部源[参考](http://mirrors.ht.com/repo/help/centos.html)

### 克隆代码
代码仓库有2个可选
一个是[github](https://github.com/oVirt/ovirt-engine)，另一个是ovirt项目开发用的[gerrit](https://gerrit.ovirt.org/#/admin/projects/ovirt-engine)

    cd /the/path/you/want/
    git clone https://github.com/oVirt/ovirt-engine.git
    cd ovirt-engine
    //切换分支，我们这里使用ovirt-engine-4.0
    git branch -a  //查看全部分支
    git checkout -t origin/ovirt-engine-4.0 //切换分支
    
### 导入项目

下载安装intellij idea安装；完成后打开。

    Import Project -> 
        选择克隆代码路径 -> 
    import project from external model ->
        选择<maven> ->
    Environment settings... ->
        选择3.5.0的MAVEN_HOME->
        选择Override ->
    ->next ... 
    ->选择新的jdk 
    ->next ...

全部完成后ide会自动去下载依赖

觉得国外下载速度慢可以[参考](http://blog.csdn.net/u010717403/article/details/52188496)替换成阿里源或者内部的源

依赖导入完成后就可以阅读代码了

## 开发环境

阅读代码的下一步就是把项目跑起来

### 编译
    //关闭单元测试，不然时间太久
    make install-dev PREFIX="/the/path/you/want/" BUILD_UT=0
    
    //建议时间戳来设置不同的安装目录，不然清理和重新配置的过程会有很多麻烦
    //第一次时间还是会比较久，因为会有一些临时下载的包
    //存在因为网络原因导致失败的情况，请耐心等待
    
    ...
    
编译的时候我们可以先去做其他事情

成功的效果：
![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/make_install_dev.png?raw=true)




### 依赖服务

参考ovirt-engine的readme文件
```
- jdk-8 / openjdk-8-jdk / icedtea-8
- mime-types or mailcap
- unzip
- openssl
- postgresql-server postgresql
- python-m2crypto / m2crypto
- python-psycopg2 / psycopg
- python-jinja2 / Jinja2
- libxml2-python / libxml2[python]
- python-daemon
- otopi >= 1.1.0
- ovirt-host-deploy >= 1.1.0
- ovirt-setup-lib
- maven-3 (optional)
- pyflakes (optional)
- python-pep8 / pep8 (optional)
- patternfly1 (optional)
- python-docker-py (optional)
```
首先

    sudo yum install http://plain.resources.ovirt.org/pub/yum-repo/ovirt-release40.rpm
    
然后执行

```
sudo yum install mailcap unzip m2crypto python-psycopg2 python-jinja2 libxml2-python python-daemon otopi ovirt-host-deploy ovirt-setup-lib postgresql-server ovirt-engine-wildfly ovirt-engine-wildfly-overlay patternfly1 pyflakes


//
```

找不到的包去[rpmfind](https://rpmfind.net/)里面去下


#### 配置postgresql


```
sudo -i
cd /var/lib/pgsql/data
postgresql-setup initdb
vim pg_hba.conf  #增加允许的IP
```
![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/postgresql1.png?raw=true)



```
vim postgresql.conf 
```
![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/postgresql2.png?raw=true)


#### 增加数据库用户



    systemctl restart postgresql.service
    
    su postgres
    
    psql
    
    #create user
    create user engine password 'engine';
    #create database
    create database engine owner engine template template0 encoding 'UTF8' lc_collate 'en_US.UTF-8' lc_ctype 'en_US.UTF-8';
    #exit
    \q  

#### 增加jboss环境

    sudo vim /etc/profile
    
    #增加一段    
    export JBOSS_HOME=/usr/share/ovirt-engine-wildfly

### engine 服务启动
#### engine-setup

在编译完成后，如果是第一次启动服务我们需要调用ovirt提供的setup脚步来配置初始化服务


    cd /the/path/you/make/install-dev/
    
    ./bin/engine-setup
    
    #后面根据个人需求配置
    
#### start

```
./share/ovirt-engine/services/ovirt-engine/ovirt-engine.py start

```

访问  ++http://127.0.0.1:8080/ovirt-engine/++



#### 可能出现的错误的解决办法

[错误1](http://blog.csdn.net/wangjun_pfc/article/details/5528893)

登录效果![image](https://github.com/ShaneDean/file/blob/master/blog/ovirt_engine_env/webadmin1.png?raw=true)
    
## 调试环境

服务起来后的下一步就是启用调试环境，可以断点调试



