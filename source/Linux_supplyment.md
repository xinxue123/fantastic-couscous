echo
------------------------
`echo -n "helo\nhhh"` # 可以接转义字符\t tab \n换行 \b回退


history
------------------
- `export TMOUT=5` # 终端超时
- `export HISTSIZE=5` # 历史记录保存时间
- `export HISTFILESIZE`　＃ ~/.bash_history 历史记录
- `history -c` # 清空历史记录
- `history -d num` # 删除某一条记录

ulimit
----------------------
- `/etc/security/limit.conf` # 配置文件
- `ulimit -a` # 显示文件描述符

issue
-------------------
- `/etc/issue` # linux 版本信息(可清空)
- `/etc/motd` # 修改linux版本信息

chattr
-------------------
`chattr +i /etc/passwd` # 锁定文件,不可修改文件

kernel 相关
--------------------
- `/etc/sysctl.conf` # 内核修改 `sysctl -p` 生效
- `/proc/sys/net` # 内核的可配置参数

wget 
----------------------------
- `wget http://url`
	`--spider` 不下载，仅检查
	`-T --timeout=10` 设置超时时间
	`--tries=2` 重试次数
	`-q --quiet` 安静下载,无输出结果

alias 起别名

less 按屏或按行查看文件与more类似

DNS配置文件 /etc/resolv.conf(优先级低于网卡配置/network-scripts/下的文件)


fstab
--------------------
`dd if=/dev/zero of=/dev/sdd1 bs=4096 count=10` 虚拟盘

设备      挂载点   文件类型  挂载参数  备份 检查项
/dev/sda1 /data  xfs  defaults  0 0

`mount /dev/sda1 /mnt/` # 挂载盘命令

fsck 磁盘检查,不要检查好磁盘,磁盘必须卸载

/etc/rc.local
-----------------------
服务器档案文件,可以把开机启动项放到rc.local文件中,linux启动流程最后一项

/etc/inittab
------------------------
运行级别

/var/log/message 系统日志
/var/log/secure  ssh日志

grep
----------------------
- `grep -n "." files` # 显示行号
- `cat -n files` # 显示行号
- `less -N files` # 显示行号

tar
--------------------------
tar zxvf a.tar.gz
tar zcvf a.tar.gz *
tar jxvf b.tar.bz
tar jcvf b.tar.bz *
find / -type f |xargs tar ......

cut
--------------------------
`echo 1 2 h j >> t5`
`cut -d ' ' -f3 t5` #-d 分隔符 -f 取第几个 

inode
-------------------
索引节点:文件或目录的唯一标识,linux读取文件时首先要读取索引节点;
inode记录了文件属性,但不包含文件名;内容存放在block中;
查看inode占用情况:`df -i`,小文件增多时很容易将inode用尽;

硬链接
----------
通过ln命令创建,多个具有相同inode节点号的多个文件互为硬链接文件;
删除所有硬链接文件和源文件后,文件会被删除;

regular
----------------------
export LC_ALL=C

tr 字符替换, tr asdf 4567 将asdf替换为4567

查找文件命令
--------------------
- find 
- which 从全局环境变量里查
- whereis -b
- locate 从数据库里查

ctrl + a # 切换到命令行首
ctrl + e # 切换到命令末尾
ctrl + u # 剪切光标之前的
ctrl + k # 剪切光标之后的

umask
---------------------
创建文件的默认最大权限时666,然后文件权限经过umask后会减权限

DNS解析
-------------------
根服务器,全球有十三台
解析命令dig、nslookup、host、ping

修改主机名
------------------
- 修改/etc/hosts
- hostnamectl name

抓包工具
-------------------
tcpdump
nmap

lsof
------------------
查看文件被删除,进程仍然占用
lsof | grep del
lsof -i :80


无法上网问题排查
---------------------------
- 检查链路
- ping www.baidu.com
- ping 外网ip
- ping 网关
- 检查DNS

网站慢排查
---------------

- 本地局域网速度
- 用户带宽
- 本地带宽
- 服务器CPU利用率过高


nfs
---------------
- 性能优化
- 安全优化
- 内核优化: socket连接,读写缓冲适量调大
问题: mount.nfs access denied by server

umount
-------------------
umount -lf /mnt # 卸载磁盘

http协议
-------------------------------


