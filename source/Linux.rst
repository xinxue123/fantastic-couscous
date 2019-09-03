Linux 
==========

获得root权限::

 sudo su root

安装软件时：./ 或 bash


vim 基本操作
---------------
在文本编辑器内部::

 25gg #跳行到25行 
 shift+g #跳到文本末尾
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

 vim +114 file_name



安装图形界面
-----------------

安装步骤::

 1.  yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
 2.  ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target 
 3.  startx
 4.  yum install tigervnc-server -y
 5.  cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
 6.  vim /etc/systemd/system/vncserver@:1.service
 7.  systemctl daemon-reload
 8.  systemctl enable vncserver@:1.service  #开机启动
     systemctl start vncserver@:1.service   #启动服务
     systemctl stop vncserver@:1.service   #停止服务
 9. 下载vnc软件并连接：IP:1


安装python
-----------------

安装步骤::

 1.  wget "https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tgz"
 2.  tar -zxvf Python-3.7.1.tgz 
 3.  yum install libffi-devel zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make -y
 4.  ./configure --prefix=/usr/local/python3
 5.  make && make install
 6.  ln -s /usr/local/python3/bin/python3 /usr/bin/python3
 7.  ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
 8.  pip3 install --upgrade pip
 9.  pip3 install ipython

 卸载：
 10. sudo apt-get purge --auto-remove python3.4


 修复误删除python
 ctrl+alt+F1进入console 
 sudo apt-get install ubuntu-minimal ubuntu-standard ubuntu-desktop 
 sudo reboot

 修改pip源
 ~/.pip/pip.conf
 %HOMEPATH%\pip\pip.ini

 [global]
 index-url = http://pypi.douban.com/simple
 [install]
 trusted-host=pypi.douban.com  # 如果不添加这两行，将会出现错误提示


 阿里云 http://mirrors.aliyun.com/pypi/simple/
 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 
 豆瓣(douban) http://pypi.douban.com/simple/ 
 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/


安装thrift
------------------------------
安装步骤::

 1>安装依赖
 yum -y install automake libtool flex bison pkgconfig gcc-c++ boost-devel libevent-devel zlib-devel python-devel ruby-devel openssl-devel
 2>安装thrift
 wget "http://mirror.bit.edu.cn/apache/thrift/0.10.0/thrift-0.10.0.tar.gz"
 3>验证是否可行
 thrift -version
 4>启动hbase的thrift服务
 hbase-daemon.sh start thrift

 wget http://iweb.dl.sourceforge.net/project/boost/boost/1.60.0/boost_1_60_0.tar.gz
 ./bootstrap.sh --prefix=/usr
 ./b2 install 

 在 make 这一步会发生一个错误 g++: error: /usr/lib64/libboost_unit_test_framework.a: No such file or directory,

 错误原因是：./configure 的时候是默认编译32位的，不会在 /usr/lib64/ 下产生文件
 修改方法：先查找文件 find / -name libboost_unit_test_framework.a,
 比如在 /usr/local/lib/libboost_unit_test_framework.a，
 就可以做如下操作，sudo ln -s /usr/local/lib/libboost_unit_test_framework.a /usr/lib64/libboost_unit_test_framework.a,
 然后重新执行 make

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

 list=`ls`
 for var in $list;do
  echo "$var"
 done

 funcName(){
 函数体(逻辑循环判断)
 }
 funcName $1 传参
