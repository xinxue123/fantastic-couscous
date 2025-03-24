Scala Note
=============


*scala 读取图片生成二进制流*
------------------------------

代码如下::

 import java.io.FileInputStream
 val path = "path"
 val f = new FileInputStream(path)
 val i = f.available()
 val data = new Array[Byte](i)
 f.read(data)
 //生成字符串
 val strings = new String(org.apache.commons.codec.binary.Base64.encodeBase64(data)) 
 //生成字节数组，用base64编码
 val bytes = org.apache.commons.codec.binary.Base64.encodeBase64(data)

*breeze 的简单操作*
-----------------------

breeze的创建函数::

 import breeze.linalg._
 val s1 = DenseVector.tabulate(20){i=>i*2} 
 val s2 = DenseMatrix.tabulate(2,3){(i,j)=> i+j}
 // 创建随机数(0,1)
 val s3 = DenseVector.rand(20)
 val s3 = DenseMatrix.rand(20)
 // 创建0,1向量
 val s4 = DenseVector.zeros[Double](4)
 val s5 = DenseVector.ones[Int](5)

 
breeze切片::

 println(s1(0)) // 第一个数据
 println(s1(0 until  4 by 1)) // 第一个到第四个数据
 println(s1(-1)) // 最后一个数据
 println(s2(2,3)) // 
 println(s2(::,3)) // 所有行的第四列数据
 println(s2(-1,::)) //最后一行的所有列数据
 // example
 val rand_seq = IndexedSeq(1,3,5)
 val target = breeze.linalg.DenseVector.zeros[Double](10) // 构造一个double型的向量
 val values = breeze.linalg.DenseVector(3.0,2.4,2.5)
 target(rand_seq) := value

breeze计算::

 // 常规计算
 val s3 = s1 + s2
 val s4 = s1 * 2
 println(min(s1))

 // 矩阵之间的运算
 import breeze.linalg.operators.OpEq // 判断是否相等
 import breeze.linalg.operators.OpGT // 判断前者是否大于后者
 import breeze.linalg.operators.OpLT // 判断前者是否小于后者
 val s2 = DenseMatrix.tabulate(4,5){(i,j)=> i+j}
 val s3 = DenseVector.range(10,30)
 val s4 = DenseMatrix(s3).reshape(4,5) - 14
 println(OpLT(s4,s2))

 // 四舍五入等操作
 import breeze.numerics._
 val s5 = DenseVector(1.24,2.35,1.8)
 println(round(s5))
 println(ceil(s5))
 println(floor(s5))

 //常规统计值
 import breeze.linalg._
 import breeze.stats._
 val s5 = DenseVector(1.24,2.35,1.8,3.5,4.6,4.3,3,1,6)
 println(sum(s5))

 mean(s5)
 println(mean(s5))
 println(stddev(s5))
 println(median(s5))
 hist(s5,2)

*scala关于时间的函数*
------------------------

关于操作scala时间的函数如下::

 import java.text.SimpleDateFormat
 import java.util.Calendar

 val cal = Calendar.getInstance()
 cal.clear()
 // calendar 的日历月份默认是从第0月开始的，12月即第0月
 cal.set(2019,1,1)
 val day_of__year = cal.get(Calendar.DAY_OF_YEAR) // 获取一年中的第几天
 
 // 获取当前时间并获取当前是该年的第几天
 val t = DateTime.now().toString() // 获取当前时间并转为字符串 2019-08-06T17:12:27.370+08:00
 val patten = "(\\d\\d\\d\\d)-(\\d\\d)-(\\d\\d)".r // 构建正则
 val time_piece = patten.findAllIn(t).next().toString.split("-") // 获取年、月、日
 // 将时间设置到calendar
 cal.set(time_piece3(0).toInt,time_piece3(1).toInt-1,time_piece3(2).toInt) 
 val day_of__year = cal.get(Calendar.DAY_OF_YEAR) // 获取一年中的第几天

map转json::

 import org.json4s.DefaultFormats
 import org.json4s.jackson.Json
 import scala.collection.immutable.{ListMap, Map}

 var map1 = Map[String,String]()
 for (i<- -1 to 3){map1 += ((-i).toString -> 0.2.toString)}
 val m = Json(DefaultFormats).write(map1)
 println(m)

map排序::

 import scala.collection.immutable.{ListMap, Map}

 var map1 = Map[String,String]()
 for (i<- -1 to 3){map1 += ((-i).toString -> 0.2.toString)}
 // 根据key值的数值大小排序
 println(ListMap(map1.toSeq.toList.sortBy(_._1.toDouble):_*)) 








