机器学习算法
===========================

回归拟合
-----------------
利用torch实现一元线性回归,代码如下:

::

	import torch
	from torch import nn
	import matplotlib.pyplot as plt


	def get_fake_data(batch_size=10):
	    x = torch.rand(batch_size, 1) * 20
	    y = 2 * x + (1 + torch.randn(batch_size, 1)) * 3
	    return x, y


	if __name__ == '__main__':
	    criterion = nn.MSELoss() 
	    w = torch.rand(1, 1)  # 初始化权重
	    b = torch.rand(1, 1)  # 初始化截距
	    lr = 0.001  # 设置学习率 
	    plt.ion()
	    plt.figure(figsize=(8, 12))
	    for epoch in range(100):
	        plt.cla()
	        x, y = get_fake_data()
	        w = w.clone().detach().requires_grad_(True)
	        b = b.clone().detach().requires_grad_(True)
	        y_pre = x * w + b
	        loss = criterion(y_pre, y)
	        loss.backward()
	        # print(w.grad)
	        # print(b.grad)
	        w = w - lr * w.grad  # 更新权重
	        b = b - lr * b.grad  # 更新截距
	        print("weight is {}, intercept is {}".format(w, b))
	        plt.scatter(x.data, y.data)
	        plt.plot(x.data, y_pre.data)
	        plt.show()
	        plt.pause(1)

torch可视化
-------------------------
torch 可以采用visdom

1. 安装及启动

>>> pip3 install visdom
python -m visdom.server

2. 使用

EM算法
----------------------------------
EM 算法的手动实现

1. 首先计算E步,假设分布的均值和方差已知,分布的概率pi已知,此时可以计算每个样本在不同分布上的概率,计算公式如下：

 .. image:: gumma.png 
  :height: 171px
  :width:  447px
  :scale: 50 %
  :alt: alternate text
  :align: center

2. 计算M步,根据计算后的概率重新调整分布的均值和方差，以及pi,计算公式如下：

 .. image:: M_step.png 
  :height: 390px
  :width: 503 px
  :scale: 50 %
  :alt: alternate text
  :align: center

3. 具体实现代码如下
::

	import numpy as np
	from scipy.stats import multivariate_normal
	import matplotlib.pyplot as plt
	from sklearn.metrics.pairwise import pairwise_distances_argmin

	if __name__ == '__main__':
	    np.set_printoptions(suppress=True)
	    x1 = multivariate_normal([160, 55],[[18,12],[12,31]])  # 模拟一个高斯分布
	    x2 = multivariate_normal([173,65],[[22,28],[28,105]])
	    x1_data = x1.rvs(400)
	    x2_data = x2.rvs(400)
	    x = np.vstack((x1_data, x2_data))
	    u1 = np.min(x, axis=0)  # 初始化第一个分布的均值
	    u2 = np.max(x, axis=0)  # 初始化第二个分布的均值
	    uu1 = u1
	    uu2 = u2
	    var1 = np.diag(np.var(x, axis=0))  # 方差
	    var2 = np.diag(np.var(x, axis=0))  # 方差
	    pi = 0.5
	    for i in range(500):
	        n1 = multivariate_normal(mean=u1, cov=var1)  # 构造第一个分布
	        n2 = multivariate_normal(mean=u2, cov=var2)  # 构造第二个分布
	        n1_p = n1.pdf(x)
	        n2_p = n2.pdf(x)
	        gumma_1 = pi * n1_p
	        gumma_2 = (1-pi) * n2_p
	        g = pi * n1_p + (1-pi) * n2_p
	        gumma_1 = gumma_1 / g
	        gumma_2 = gumma_2 / g

	        # M step
	        u1 = np.dot(x.T, gumma_1) / np.sum(gumma_1)
	        u2 = np.dot(x.T, gumma_2) / np.sum(gumma_2)
	        var1 = np.dot((x - u1).T * gumma_1.T, x-u1) / np.sum(gumma_1)
	        var2 = np.dot((x - u2).T * gumma_2.T, x-u2) / np.sum(gumma_2)
	        pi = np.sum(gumma_1) / len(gumma_1)
	        print("第{}次：u1:".format(i), u1, " u2:", u2)

	    print(pi)
	    print(u1, u2)
	    print(var1)
	    print(var2)
	    plt.scatter(x1_data[:, 0], x1_data[:, 1])
	    plt.scatter(x2_data[:, 0], x2_data[:, 1])
	    plt.show()





