redis
=========================

1.数据操作
--------------------------

string(字符串)
^^^^^^^^^^^^^^^^^^^^^^^^^

  - 设置键值

	* ``set key value`` # 仅键值
	* ``set key value seconds`` # 键值+过期时间，需要指定EX或PX
	* ``mset key1 value1 key2 value2`` # 设置多个

  - 追加值
	``append key value``

  - 获取值
	* ``get key``
	* ``mget key1 key2`` # 获取多个值

  - 通用命令
	* ``keys *`` # 查找所有键(支持正则)
	* ``exist key`` # 判断键是否存在
	* ``type key`` # 查看键的类型
	* ``del key`` # 删除键
	* ``expire key seconds`` # 设置过期时间
	* ``ttl key`` # 查看有效时间

hash类型(哈希)
^^^^^^^^^^^^^^^^^^^^^^^^^

  - 设置属性
	* ``hset key field value``,例如：``hset user name li``
	* ``hmset key field1 value1 field2 value2``

  - 获取属性
	* ``hkeys key``
	* ``hmget key field1 field2``

  - 删除属性
	``hdel key field1 field2``

list类型(列表)
^^^^^^^^^^^^^^^^^^^^^^^^^

  - 插入数据
	* ``lpush key value1 value2`` # 左侧插入
	* ``rpush key value1 value2`` # 右侧插入
	* ``linsert key ... ... ...``

  - 查看数据
	``lrange key start end``

  - 根据索引设置值
	``lset key index value``

  - 将前count次值为value的元素删除
	``lrem key count value``

set类型(集合)
^^^^^^^^^^^^^^^^^^^^^^^^^

	* ``sadd key members1 members2`` # 添加元素
	* ``smembers key`` # 查看数据
	* ``srem key member`` # 删除数据

zset(有序集合,按权重排序)
^^^^^^^^^^^^^^^^^^^^^^^^^^^

	* ``zadd key weight1 value1 weight2 value2`` # 添加权重及元素
	* ``zrange key start end`` # 查看元素
	* ``zscore key member`` # 查看元素权重
	* ``zrem key member1 member2`` # 删除元素

2.python交互
--------------------------

安装及调用
^^^^^^^^^^^^^^^^

::

  pip3 install redis
  from redis import *
  sr = StrictRedis(host="localhost",port=6379,db=0)


| 使用方法与redis客户端一致

3.配置主从
---------------------------------

| redis 服务启动 ``redis-server conf``

| redis 关闭服务 `` redis-cli -h 192.168.0.134 shutdown``

| redis 客户端启动 ``redis-cli -h 192.168.0.134``

| redis的配置文件如果用yum安装位于/etc/redis/下，源码安装是在安装目录下的redis.conf文件


| 配置从服务器(位于同一机器)

 * 复制redis文件到从服务器,修改文件名为slave.conf
 * 修改文件

 ::

 	bind 192.168.0.134 # 服务器ip地址
 	slaveof 192.168.0.134 6379 # 设置主服务器IP和PORT
 	port 6378 #如果非同一台电脑也可以将port设置为6379
 	daemonize yes #设置以守护进程启动

 * 启动redis服务

 ``sudo redis-server slave.conf``

 * 查看主从关系 

 ``redis-cli -h 192.168.0.134 info Replication``

4.python连接集群
---------------------------------

``pip install redis-py-cluster`` # 安装

``from rediscluster import *`` # 调用

需要注意的是创建类时需要将主节点的IP、PORT以字典的形式传入

| 命令参考 http://doc.redisfans.com/

5.ray与redis的使用

``ray start --head --redis-port=6379`` # 启动master

``ray start --address="192.168.0.134:6379`` # 启动从节点

``ray stop`` # 停止节点

::

	import ray
	ray.init(address="192.168.0.134:6379") # 初始化ray,指定IP地址