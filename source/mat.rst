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







