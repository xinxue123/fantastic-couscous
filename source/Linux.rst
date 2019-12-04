Linux 
==========

|  安装软件时使用：./ 或 bash
|  获得root权限
::

 sudo su root

linux分区
--------------------
| 两种分区方式 MBR,GPT,引导包括BIOS、EFI
| GRUB 主流的引导程序
| MBR只能创建四个主分区,一个扩展分区占用一个主分区位置;扩展分区不可用,只能在其中建立逻辑分区;主分区可以直接使用;

网络配置
-------------------

| ifconfig -a
| ifup eth0 # 打开接口
| ifdown etho # 关闭接口
| 使用setup配置网络接口

配置好的位置位于 /etc/sysconfig





vim 基本操作
---------------
在文本编辑器内部::

 25gg #跳行到25行 
 shift+g #跳到文本末尾
 ctrl+o  # 返回上一次修改的位置
 shift+V 可视块
 ctrl+v 垂直模式
 x or d 均是剪切操作
 y 复制
 p 粘贴
 dd 删除命令
 r 替换
 R 可一直替换
 > or  <    可视模式下缩进
 >> or <<   正常模式下缩进
 .          重复以前的操作s
 /str       查找命令
 n          向下查找
 N          向上查找

 :%s/要替换文本/替换为/g   全局查找替换
 :s/要替换文本/替换为/g    可视区查找替换
 :%s/要替换文本/替换为/gc  确认查找替换

在文本编辑器外部::

 vim +114 file_name # 定位到114行



安装图形界面
-----------------

安装步骤

1.  yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
2.  ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target 
3.  startx
4.  yum install tigervnc-server -y
5.  cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
6.  vim /etc/systemd/system/vncserver@:1.service
7.  systemctl daemon-reload
8.	设置vnc登录账号(vncpasswd)
9.  服务设置

 - systemctl enable vncserver@:1.service  #开机启动
 - systemctl start vncserver@:1.service   #启动服务
 -  systemctl stop vncserver@:1.service   #停止服务

10. 下载vnc软件并连接：IP:1


安装python
-----------------

1.  wget "https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tgz"
2.  tar -zxvf Python-3.7.1.tgz 
3.  yum install libffi-devel zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make -y
4.  ./configure --prefix=/usr/local/python3
5.  make && make install
6.  ln -s /usr/local/python3/bin/python3 /usr/bin/python3
7.  ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
8.  pip3 install --upgrade pip
9.  pip3 install ipython
10. sudo apt-get purge --auto-remove python3.4 # 卸载


修复误删除python::

 ctrl+alt+F1进入console 
 sudo apt-get install ubuntu-minimal ubuntu-standard ubuntu-desktop 
 sudo reboot

修改pip源::

 ~/.pip/pip.conf
 %HOMEPATH%\pip\pip.ini

 [global]
 index-url = http://pypi.douban.com/simple
 [install]
 trusted-host=pypi.douban.com  # 如果不添加这两行,将会出现错误提示

 阿里云 http://mirrors.aliyun.com/pypi/simple/
 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 
 豆瓣(douban) http://pypi.douban.com/simple/ 
 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/


安装thrift
------------------------------

1. 安装依赖
::
 
 yum -y install automake libtool flex bison pkgconfig gcc-c++ boost-devel libevent-devel zlib-devel python-devel ruby-devel openssl-devel

2. 安装thrift
::
 
 wget "http://mirror.bit.edu.cn/apache/thrift/0.10.0/thrift-0.10.0.tar.gz"

3. 验证是否可行
::

 thrift -version

4. 启动hbase的thrift服务
::
 
 hbase-daemon.sh start thrift

|  boost 下载安装

a. wget http://iweb.dl.sourceforge.net/project/boost/boost/1.60.0/boost_1_60_0.tar.gz
b. ./bootstrap.sh --prefix=/usr
c. ./b2 install 

|  在 make 这一步会发生一个错误 g++: error: /usr/lib64/libboost_unit_test_framework.a: No such file or directory。错误原因是：./configure 的时候是默认编译32位的,不会在 /usr/lib64/ 下产生文件。修改方法：先查找文件 find / -name libboost_unit_test_framework.a,比如在 /usr/local/lib/libboost_unit_test_framework.a,然后建立软连接
::

 sudo ln -s /usr/local/lib/libboost_unit_test_framework.a /usr/lib64/libboost_unit_test_framework.a

| 最后重新执行 make

shell相关知识
---------------------------

变量定义::
 
 temp=666 # 定义不同变量
 env 查看系统变量
 set GOROOT=/usr/local/go/src # 设置系统变量
 export GOROOT=/usr/local/go/src # 设置系统变量
 ~/.bashrc

变量类型::
 
 # 位置变量
 # 执行脚本 ./test.sh a b c 
 # a,b,c为传递的参数
 $0 执行的脚本名字
 $1
 $2
 $3

 # 特殊变量
 $# 传递参数的个数
 $@ 所有参数
 $? 脚本完成状态,0:success other:failed
 $$ 进程id

 # 取值操作
 v=$变量名
 var=$(pwd)
 var=`pwd`

条件判断和循环::

 if [条件判断];then
 逻辑处理
 fi

  var1=1 # 不能有空格
 var2=2
 if [ $var1 -gt $var2] # 注意空格
 then
 echo "$var1 大于 $var2 "
 elif [ $var1 -lt $var2 ]
 then
 echo "$var1 小于 $var2 "
 else
 echo "$var1 等于 $var2 "

 #-eq 等于,如:if [ "$a" -eq "$b" ] 
 #-ne 不等于,如:if [ "$a" -ne "$b" ] 
 #-gt 大于,如:if [ "$a" -gt "$b" ] 
 #-ge 大于等于,如:if [ "$a" -ge "$b" ] 
 #-lt 小于,如:if [ "$a" -lt "$b" ] 
 #-le 小于等于,如:if [ "$a" -le "$b" ] 
 #<   小于(需要双括号),如:(("$a" < "$b")) 
 #<=  小于等于(需要双括号),如:(("$a" <= "$b")) 
 #>   大于(需要双括号),如:(("$a" > "$b")) 
 #>=  大于等于(需要双括号),如:(("$a" >= "$b"))

 list=`ls`
 for var in $list;do
  echo "$var"
 done

 funcName(){
 函数体(逻辑循环判断)
 }
 funcName $1 传参


linux 时间处理
-----------------------
::
 
 date +%F # full date; same as %Y-%m-%d
 date +%j # day of year (001..366)
 date +%m # month (01..12)
 date +%d # day of month (e.g., 01)

 date --date="2019-09-01 1:1:1" # 自定义时间字符串
 date +%F -d "+2hour" # 未来俩小时
 date +%F -d "+2day" # 未来两天

crontab
----------------------

at 的排程编辑
::

 # at 是仅执行一次的工作
 at now + 5 minutes
 at>echo "hello" > /root/test.txt
 ctrl+d # 按下ctrl+d结束编辑
 atq # 查询排程与 at -l 功能相同
 atrm # 删除创建的排程与 at -d 功能相同

1. crontab -e # 进入编辑排程,可增加、删除某个排程

守护进程crond

定时任务在任务末尾要加 >/dev/null 2>&1,去除不必要的信息

 - *(星号)      代表任何时刻都接受的意思！举例来说,范例一内那个日、月、周都是 * , 就代表着『不论何月、何 日的礼拜几的 12:00 都执行后续指令』的意思！   

 - ,(逗号)      代表分隔时段的意思。举例来说,如果要下达的工作是 3:00 与 6:00 时,就会是:0 3,6 * * * command 时间参数还是有五栏,不过第二栏是 3,6, 代表 3 与 6 都适用                                                   

 - -(减号)      代表一段时间范围内,举例来说, 8 点到 12 点之间的每小时的 20 分都进行一项工作, 20 8-12 * * * command 仔细看到第二栏变成 8-12  代表 8,9,10,11,12 都适用的意思！                                           

 - /n(斜线)     n 代表数字,亦即是『每隔 n 单位间隔』的意思,例如每五分钟进行一次,则：*/5 * * * * command 很简单吧！用 * 与 /5 来搭配

| 0   12  *   *   *  mail -s "at 12:00" dmtsai < /home/dmtsai/.bashrc 


2. 系统的配置文件： /etc/crontab, /etc/cron.d/* 
  
|   在/etc/crontab 可是一个『文本』,你可以 root 的身份编辑这个文件,例子如下图
 
 .. image:: crontab1.PNG 
  :height: 500px
  :width: 1000 px
  :scale: 50 %
  :alt: alternate text
  :align: center


anacron
----------------

定时任务,适合与非二十四小时的任务,以天为周期或每次开机执行



samba
------------

|  windows访问samba共享输入正确的密码时提示密码不正确,问题原因是网络安全：LAN管理器身份验证级别问题

1. 本地安全策略,本地策略-安全选项,需要修改成默认的值的修改方式:查找注册表浏览到 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA,直接删除 LMCompatibilityLevel 键。确定删除后。
2. 运行secpol.msc命令。打开本地安全策略。
3. 查看 网络安全:LAN管理器身份验证级别,安全设置已经变为默认“没有定义”
修改后发现输入账户密码就可以直接访问了
::

 yum install -y samba # 安装samba
 service smb start # 启动服务
 testparm # 测试共享配置是否正确
 smbpasswd -a root # 增加用户


FTP
-------------
::

 yum install -y vsftpd # 安装vsftpd
 /etc/vsftpd # 文件的保存位置
 /etc/vsftpd/ftpusers # 黑名单
 service vsftpd start # 启动服务


其他命令
--------------------

::

 sed -n '2,3p' /etc/passwd # 选定第2、3行
 sed -i 's/ss/ff/g' file #用ff修改ss 
 sed -r 's/(.*)/\1/' files # 正则匹配
 awk -F ":" '{print $1}' files # -F为分隔符,默认为空格,$NF为最后一列
 awk -F ":" '{if(NR<31 && NR >1) print }' /etc/passwd # 
 NR指代行数,&&、||
 awk -F "[, ]" '{print $1 $2}' files
 grep root -B 2 /etc/passwd # 查找root的前两行(B、A、C)
 grep -E 'python|go' files # -E 可同时过滤多个值
 find / -type f -name "*.txt" # 查找文件-o、-a 或者、同时成立,!取反
 ntpstat # 查看时间同步状态


重定向

1. 标准输入:代码为0
2. 正常输出:代码为1
3. 错误输出:代码为2 
1>/dev/null 2>&1


