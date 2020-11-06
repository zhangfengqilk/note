## 1 编译通过，运行时提示找不到库文件

	libglog.so.0: cannot open shared object file: No such file or directory

dconfig是一个动态链接库管理命令，其目的为了让动态链接库为系统所共享。
ldconfig的主要用途：
默认搜寻/lilb和/usr/lib，以及配置文件/etc/ld.so.conf内所列的目录下的库文件。
搜索出可共享的动态链接库，库文件的格式为：lib***.so.**，进而创建出动态装入程序(ld.so)所需的连接和缓存文件。
缓存文件默认为/etc/ld.so.cache，该文件保存已排好序的动态链接库名字列表。
ldconfig通常在系统启动时运行，而当用户安装了一个新的动态链接库时，就需要手工运行这个命令。

# 一 安装配置mysql数据库
## 1.安装MySQL

```shell
sudo apt-get install mysql-server
```


## 2.配置MySQL
查看运行状态
```shell
systemctl status mysql.service
```
配置root用户的用户名和密码，设置本体访问权限
``` 
update mysql.user 
set authentication_string=password('root') 
set plugin="mysql_native_password"
where host  ='localhost' and user='root';
```
##  MySql远程访问

### 1  改表法。

可能是你的帐号不允许从远程登陆，只能在localhost。这个时候只要在localhost的那台电脑，登入mysql后，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，从"localhost"改称"%"
```
mysql -u root -pvmwaremysql>use mysql;

mysql> update user set host = '%' where user = 'root';

mysql> select host, user from user;
```
### 2 授权法。

例如，你想myuser使用mypassword从任何主机连接到mysql服务器的话。
```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

FLUSH   PRIVILEGES;
```
如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器，并使用mypassword作为密码
```
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

FLUSH   PRIVILEGES;
```
如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器的dk数据库，并使用mypassword作为密码
```
GRANT ALL PRIVILEGES ON dk.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;

FLUSH   PRIVILEGES;
```
我用的第一个方法,刚开始发现不行,在网上查了一下,少执行一个语句 mysql>FLUSH RIVILEGES 使修改生效.就可以了

另外一种方法,不过我没有亲自试过的,在csdn.net上找的,可以看一下.

在安装mysql的机器上运行：

1、d:\mysql\bin\>mysql   -h   localhost   -u   root //这样应该可以进入MySQL服务器

2、mysql>GRANT   ALL   PRIVILEGES   ON   *.*   TO   'root'@'%'   WITH   GRANT   OPTION //赋予任何主机访问数据的权限

3、mysql>FLUSH   PRIVILEGES //修改生效

4、mysql>EXIT //退出MySQL服务器

这样就可以在其它任何的主机上以root身份登录啦！

# 二 安装ftp

## 1、首先用命令检查是否安装了vsftpd

```
vsftpd -version
```

![这里写图片描述](https://www.linuxidc.com/upload/2016_12/161220083241911.png) 
如果未安装用一下命令安装

```
sudo apt-get install vsftpd
```

安装完成后，再次输入vsftpd -version命令查看是否安装成功

## 2、新建一个文件夹用于FTP的工作目录

```
mkdir /home/ftp
```

![这里写图片描述](https://www.linuxidc.com/upload/2016_12/161220083241912.png)

## 3、新建FTP用户并设置密码以及工作目录 
**ftpname为你为该ftp创建的用户名**

```
sudo useradd -d /home/ftp -s /bin/bash ftpname
```

![这里写图片描述](https://www.linuxidc.com/upload/2016_12/161220083241913.png)

为新建的用户设置密码

```
passwd ftpname
```

【注释：用cat etc/passwd可以查看当前系统用户】

## 4、修改vsftpd配置文件 
用命令打开vsftpd.conf

```
vi vsftpd.conf
```

![这里写图片描述](https://www.linuxidc.com/upload/2016_12/161220083241914.png) 
设置属性值 
anonymous_enable=NO #禁止匿名访问 
local_enable=YES 
write_enable =YES 
保存返回 

## 5、启动vsftpd服务

```
 service vsftpd start
```

# 三 安装依赖库

## 1 安装grpc库

1）首先安装pkg-config：

```
sudo apt-get install pkg-config
```
2）然后安装依赖文件autoconf、automake、libtool、make、g++、unzip、libgflags-dev、libgtest-dev、clang、libc++-dev：       
````
sudo apt-get install autoconf automake libtool make g++ unzip
sudo apt-get install libgflags-dev libgtest-dev
sudo apt-get install clang libc++-dev
````
3)、下载GRPC
  注意，github速度很慢，有时会下不下来，可以用以前下载完的源代码直接使用

```
git clone https://github.com/grpc/grpc.git
cd grpc
git submodule update --init  //更新第三方源码
```
4)、安装protobuf源码：    

    cd third_party/protobuf/
    git submodule update --init --recursive   //确保克隆子模块，更新第三方源码
    ./autogen.sh       //生成配置脚本
    ./configure        //生成Makefile文件，为下一步的编译做准备，可以加上安装路径：--prefix=path
    make               //从Makefile读取指令，然后编译
    make check         //可能会报错，但是不影响
    sudo make install  //从Makefile读取指令，安装到指定位置，默认为/usr/local/，也可以指定安装目录：--prefix=path。
        卸载的命令为make uninstall。make clean：清除编译产生的可执行文件及目标文件(object file，*.o)。make distclean：
        除了清除可执行文件和目标文件外，把configure所产生的Makefile也清除掉。//
    sudo ldconfig       // 更新共享库缓存
    which protoc       // 查看软件的安装位置
    protoc --version   //检查是否安装成功

5)、安装GRPC   

    cd ../..
    make                    //从Makefile读取指令，然后编译
    sudo make install       //从Makefile读取指令，安装到指定位置，默认为/usr/local/，这里可以加上安装路径：prefix=/usr/local/

6)、测试用例

    cd examples/cpp/helloworld/
    make                 //编译
    ./greeter_server     //服务器
    ./greeter_client     //客户端（重新开一个服务器连接）
## 2 安装zeromq
```shell
sudo apt-get install libzmq3-dev
```

## 3 安装boost

```shell
apt-cache search boost
```
  搜到所有的boost库

```shell
apt-get install libboost-all-dev
```

# 四 操作Ubuntu系统

## 1 远程ssh连接

在程序中搜索 desktop 或者remmina 出来一个remmina软件，这个软件支持ssh协议，这个协议可以远程ssh连接linux系统

## 2 ubuntu下使用ftp连接服务器

在程序中搜索 desktop 或者remmina 出来一个remmina软件，这个软件支持sftp协议，可以用于连接ftp

## 3 ubuntu 下 远程windows系统

在程序中搜索 desktop 或者remmina 出来一个remmina软件，这个软件支持RDP远程桌面连接，连接时勾选下面的共享目录即可传文件

## 4 ubuntu 下使用mysql

在ubuntu下使用mysql，可以在命令行中用

```shell
mysql -uroot -p
```

然后输入密码，进入mysql的工作环境，首先使用use 数据库名，更换以下数据库，然后就可以查看数据库了，

```mysql
show tables  #显示所有表
show databases  #显示所有数据库
create database ccs # 创建数据库
source *.sql   #导入数据库文件
```

 

# 五 安装glog

## 1 安装

```CQL
1 Git clone https://github.com/google/glog.git
2 cd glog
3 ./autogen.sh
4 ./configure  --prefix=path(install)
5 make
6 make install
```

从安装目录得到lib 和include

## 2 使用

sample code:

```c++
#include <string>
#include <iostream>
#include "glog/logging.h"   // glog 头文件
#include "glog/raw_logging.h"

int main(int argc, char** argv){
    // FLAGS_log_dir=".";   //设置log目录  没有指定则输出到控制台
    FLAGS_logtostderr = 1;  //输出到控制台
    google::InitGoogleLogging(argv[0]);    // 初始化
std::string test = "this is test";
int i = 2, number = 8;

LOG(INFO) << "it is info";     // 打印log：“hello glog.  类似于C++ stream。
LOG_IF(INFO, number > 10) << "number >  10"; 
LOG_IF(INFO, number < 10) << "number <  10";
for(i=0; i<20 ;i++){
    LOG_EVERY_N(INFO, 5) << "log i = " << i;
}

LOG(WARNING) << "It is error info"; 
LOG(ERROR) << "It is error info"; 

DLOG(INFO) << "it is debug mode";
DLOG_IF(INFO, number > 10) << "debug number > 10";  
// DLOG_EVERY_N(INFO, 10) << "log i = " << i;
RAW_LOG(INFO, "it is pthread log");
return 0;
}
```

编译:

```CQL
g++ test_main.cpp ./lib/libglog.a -I./include  -std=c++11 -DDEBUG -lpthread -o sample
```

控制台直接输出结果:

```
I0228 20:03:53.824311  1721 test_main.cpp:15] it is info
I0228 20:03:53.824481  1721 test_main.cpp:17] number <  10
I0228 20:03:53.824519  1721 test_main.cpp:19] log i = 0
I0228 20:03:53.824538  1721 test_main.cpp:19] log i = 5
I0228 20:03:53.824558  1721 test_main.cpp:19] log i = 10
I0228 20:03:53.824579  1721 test_main.cpp:19] log i = 15
W0228 20:03:53.824596  1721 test_main.cpp:23] It is error info
E0228 20:03:53.824612  1721 test_main.cpp:24] It is error info
I0228 20:03:53.824628  1721 test_main.cpp:26] it is debug mode
```



## 3 说明：
3.1 log输出说明

I0228 20:03:53.824311  1721 test_main.cpp:15] it is info

输出信息对应
I日期 时：分：秒.微秒 线程号 源文件名：行数] 信息
I是log等级首字母
3.2 log文件名说明

sample： sample.sa02.jw.li.log.INFO.20180228-194622.995 
<program name>.<hostname>.<user name>.log.<severity level>.<date>.<time>.<pid>

    1
    2

其实对应google::InitGoogleLogging(argv[0])；中的argv[0]，即通过改变google::InitGoogleLogging的参数可以修改日志文件的名称。
3.3 log等级（severity level）

INFO(=0)
3.4 条件输出

LOG_IF(INFO, number < 10) << "number <  10";

    1

当number > 10条件成立时，“ “number < 10”才会输出
LOG_EVERY_N(INFO, 5) << “log i = ” << i;
当程序中周期性的记录日志信息，在该语句第1、6、11……次被执行的时候，才会打印该log
3.5 debug模式下使用的宏

glog提供特定的宏只在debug模式下生效。以下分别对应LOG、LOG_IF、DLOG_EVERY_N操作的专用宏。

   DLOG(INFO) << "Found cookies";
   DLOG_IF(INFO, num_cookies > 10) << "Got lots of cookies"; 
   DLOG_EVERY_N(INFO, 10) << "Got the " << COUNTER << "th cookie";

    1
    2
    3

g++ test_main.cpp ./lib/libglog.a -I./include -DDEBUG -lpthread -o sample
加上-DDEBUG 编译参数
3.6 线程安全log宏

glog提供了线程安全的日志记录方式。在<glog/raw_logging.h>文件中提供了相关的宏，如，RAW_CHECK，RAW_LOG等。这些宏的功能与CHECK，LOG等一致，除此以外支持线程安全，不需要为其分配任何内存和提供额外的锁（lock）机制。

RAW_LOG(INFO, "it is pthread log");

    1

3.7 通过符号变量配置glog

FLAGS_log_dir=".";   //设置log目录  没有指定则输出到控制台
FLAGS_logtostderr = 1;  //输出到控制台

在程序中，通过修改全局变量(使用前缀”FLAGS_”)来设置符号变量
大多数符号变量修改后会立即生效
与输出位置有关(如FLAGS_log_dir)，如果要生效需要在google::InitGoogleLogging()之前设置

符号变量包括：

logtostderr(bool,default=false),                                 只输出到STDERR而不写入日志文件
stderrthreshold(int,default=2,which is ERROR)，高于该级别的日志除写入日志文件还输出到STDERR
minloglevel(int,default=0,which is INFO)，          低于该级别的日志消息不输出
log_dir(string,default="")，                                      日志输出目录
v(int,default=0)，                                                       小于等于该值的VLOG(m)会被输出，否则不会输出
vmodule(string,default="")，                                    可为源文件定制VLOG日志输出级别
max_log_size(int,default=1800)，                            日志文件最大值(单位MB)
log_link(string,default="")，                                     日志文件的连接所在的文件夹
stop_logging_if_full_disk(bool,default=false)，     如果磁盘写满是否停止记录日志
alsologtoemail(string,default="")，                          是否将日志额外发送邮件到指定地址
logemaillevel(int,default=999)，                               设置发送邮件的日志等级
logmailer(string,default="/bin/mail")，                  发送邮件程序



