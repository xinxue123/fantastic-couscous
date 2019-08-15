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

requests库的应用
------------------

发送http请求::

 url = 'https://www.baidu.com'
 headers = {'user-agent':""}
 proxy = {"http":"89.17.37.218:52957"} # 使用代理
 response = requests.get(url,params={"wd":"中国"},proxies=proxy)
 # response.content 是原始页面的byte类型，没有解码
 # response.text 是requests 解码后的数据，解码方式是推测的，可能出错

 response = request.post(url,data) # post 请求