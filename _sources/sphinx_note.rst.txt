Sphinx Note
===========

通过sphinx创建web note

*install*
-------------

前置条件：安装python,建议使用Anaconda安装：

在window下的安装sphinx的命令::

 pip install sphinx

*create project*
--------------------

安装完成后可以使用如下命令创建工程::

 sphinx-quickstart

创建完工程之后会有如下的文件结构::

 project name
 |--
 |--source
    |--_static
    |--_templates
    |--index.rst
    |--conf.py
 |--make.bat
 |Makefile

``index.rst``: 索引页，所有的文档都需要添加到index.rst

``conf.py``: 工程的配置文件，可以更改工程名，更改sphinx主题等

``_templates``: 存放模板的文件夹

``_static``: 存放静态文件的文件夹，例如图片之类

创建完工程之后就可以通过sphinx生成html文件，命令如下::

 make html

如果多次使用make html 页面没有发生变化可使用如下命令清除原先内容::

 make clean

生成html之后通过如下命令启动服务::

 python -m http.server

默认的网页地址是http://localhost:8000

若要添加自己的文档可在source文件夹下生产rst文件如support.rst,并添加内容如下::

 note
 ====

 simple note,
 you can write content here
 end

保存文件之后你的文件夹结构如下::

  project name
 |--built
 |--source
    |--_static
    |--_templates
    |--index.rst
    |--conf.py
    |--support.rst
 |--make.bat
 |Makefile

然后将你的support.rst文件添加到index.rst中,添加后的内容如下::

 .. toctree::
   :maxdepth: 2
   :caption: Contents:
   
   support.rst

最后重新make html 后就可以查看到自己的文档

*some operations*
---------------------

添加图片::

 .. image:: test.jpg  # 图片的位置
    :height: 100px
    :width: 200 px
    :scale: 50 %
    :alt: alternate text
    :align: center

上传到github::
 
 git add . // 添加所有
 git commit -m "sphinx start" // 上传到本地
 // git remote add origin https:......(your repository)
 git push origin master






