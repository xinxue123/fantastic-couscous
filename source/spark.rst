Spark 
=================
创建sparkcontext::

 val conf = spark.SparkConf().setMaster("local[2]").setAppName("dataframe")  // 创建配置
 val sc = new spark.SparkContext(conf)  // 
 SparkSession.Builder().master("local[2]").appName("data").getOrCreate()

spark 特性及其配置：
----------------------------
rdd是由一系列的partition组成；
算子作用在partition上；
rdd的容错性，partition可多可少；
rdd的tuple是kv格式；
partition提供计算最佳位置，使运算本地化；

::

 driver,worker,master 是节点上的jvm进程
 checkpoint:很少对RDD持久化;惰性
 1.当job完成后,spark回溯找到checkpoint
 2.回溯完启动job重新计算并放到checkpoint目录，
 3.放到目录后会切断RDD的依赖关系
 4.注意可以在checkpoint前进行cache
 5.指定目录sc.setCheckpointDir("")

 cache:对下次的action算子优化
 uncache() unpersit()：取消持久化
 vim /etc/profile java_home;hadoop_home;spark_home;pyspark_python; export ~
 PATH=$PATH:$JAVA_HOME/bin 若配置过java_home 则改为PATH=$JAVA_HOME/bin：$PATH

 spark-env.sh
 export SPARK_MASTER_HOST=hao
 export SPARK_MASTER_PORT=7077
 export SPARK_WORKER_CORES=2
 export SPARK_WORKER_MEMORY=3g
 export HADOOP_CONF_DIR=/

spark 相关端口及配置：
---------------------------------
web页面port:8080

历史服务器port:18080

spark-default.sh 文件中配置以下几项::

 spark.eventLog.enabled           true
 spark.eventLog.dir               hdfs://hao:9000/directory
 spark.history.fs.logDirectory    hdfs://hao:9000/directory

spark-env.sh 文件中配置::

 export SPARK_MASTER_WEBUI_PORT=18080

hbase:60010#16010  hbase端口

hbase shell        启动hbase

date -s ""         查看时间

ntpdate ntp1.aliyun.com 同步时间

spark 执行流程：
--------------------------------
nm中的executor是执行者

spark 基于yarn(:8088):: 

 1.客户端启动driver
 2.driver向rs申请启动am
 3.rs找到一台nm启动am
 4.am向rs申请资源用于启动executor
 5.rs找到资源后返回给am节点信息
 6.am找到节点启动executor()
 7.executor反向注册给driver（client:driver在客户端，cluster:driver为am）
 8.driver发送任务



spark 优化：
---------------------------
Kryo 类库进行序列化::

 --conf "spark.executor.extraJavaOption=-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps"
 rdd的缓存空间调整（降低缓存，增加任务的内存）
 sparkconf.set("spark storage memoryFraction","0,5") 0.2 都可以
 分配给task的内存就是jvm堆空间大小
 -XX:SurvivorRatio=4 如果值为4 代表两个Survivor 跟Eden 的比例市2：4
 -XX:NewRatio=4 调节新生代和老年的比例
 conf.set("spark.serializer","org.apache.spark.serializer.KryoSerializer")
 conf.registerKryoClasses(classof[])

windows下运行idea连接MySQL
-----------------------------------
::
 Exception in thread "main" java.sql.SQLException: No suitable driver
 需要下载mysql-connector-java,只要在meven repository 搜索然后添加到sbt中
 例: libraryDependencies += "mysql" % "mysql-connector-java" % "8.0.17"
 val prop = new java.util.Properties()
 prop.put("user","root")
 prop.put("password","123456")
 prop.put("driver","com.mysql.cj.jdbc.Driver")
 val table = "userinfo"
 val url = "jdbc:mysql://192.168.0.132/db"
 val sparksession = SparkSession.builder().master("local[*]").appName("interpolation").getOrCreate()
 val df = sparksession.read.jdbc(url,table,prop) // 读取整个表
 df.createOrReplaceTempView("t") // 将表注册为临时表以便使用sql语句
 val s = sparksession.sql("select id from t") // 使用sql语句
 s.show() // 将表展示出来