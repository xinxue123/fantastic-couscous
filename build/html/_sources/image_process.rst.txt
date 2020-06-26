遥感影像处理
====================================

1. 辐射定标
------------------------------------

| 辐射定标就是将图像的数字量化值（DN）转化为辐射亮度值或者反射率或者表面温度等物理量的处理过程。将未定标的DN图像转成辐射亮度值图像，通常只用一个线性转换公式及偏移和增益就可以完成，偏移和增益参数一般在元数据文件中可获取。
|	L = Gain * Pixel value + Offset
| 发射率包含大气表观反射率、地表反射率，其中地表反射率可以通过大气校正得到，大气表观反射率通过辐射亮度定标参数，太阳辐照度，太阳高度角和成像时间等几个参数计算得到。
| 当一个物体的辐射亮度与某一黑体的辐射亮度相等时，该黑体的物理温度就被称之为该物体的“亮度温度”，通过普朗克方程，将热红外辐射亮度（定标后的热红外数据）图像转成亮温图像。

2. 大气校正
------------------------------------

大气校正主要由两部分组成，分别是大气参数的估算和地表反射率的反演。造成大气效应的主要吸收体包括水汽、臭氧、氧气和气溶胶，臭氧、氧气等其他气体引起的吸收比较容易纠正，因为这些要素的浓度在时间和空间上都比较稳定，困难的是气溶胶和水汽参量的估算，因此去除气溶胶与水汽的影响成为大气校正的主要内容。

| a. 消除气溶胶的大气校正

	- 基于光谱特征的方法有长波近红外暗目标法、短波近红外暗目标法。

	- 基于时间序列影像的方法包括线性回归法。

| b. 消除水汽的大气校正

常用的大气校正模型有MODTRAN模型、6S模型。

3. 几何校正
------------------------------------
| 影像几何纠正模型分为两类：物理模型、经验模型。

| 经验模型包括一般多项式模型、直接线性变换模型、仿射变换模型、有理函数模型。

下面介绍基于GDAL进行几何校正的实现

::

	from osgeo import gdal, osr
	import os
	src_fold = r"F:\ENVI\Data\02transform\bldr_tm.img" # 待校正影像
	destination_fold = r"F:\ENVI\Data\result"
	out_name = "transform01.tif" # 输出影像名
	dst = gdal.Open(src_fold)
	## 地面控制点
	gcps = [gdal.GCP(470876.749, 4438937.779, 0, 61.887, 113.014),
	        gdal.GCP(469085.001,4433872.524,0,29.418,299.515),
	        gdal.GCP(477597.708,4433648.605,0,328.101,254.338),
	        gdal.GCP(477579.376, 4440075.697,0,287.793,33.022),
	        ]
	src_arr = dst.ReadAsArray()[0]
	dst_srs = gdal.Open(r"F:\ENVI\Data\02transform\bldr_sp.img")
	dst_srs = dst_srs.GetProjection()
	dst.SetGCPs(gcps, dst_srs)
	gdal.Warp(os.path.join(destination_fold, out_name), dst, format="Gtiff", tps=True)
	# 若使用RPC校正可在gdal.Warp(rpc=True,transformerOptions=*)这样设置

4. 图像配准
-----------------------------------

学习中

5. 图像融合
-----------------------------------

| 图像融合分为三个层次像素级融合、特征级融合、决策级融合。特征级融合包括基于变量替换技术融合方法、基于调制融合方法、基于多尺度分析融合方法。
| 下面介绍基于GDAL进行特征级图像融合的实现

基于变量替换技术融合方法01

::

	# 基于变量替换技术融合方法 PCA
	from osgeo import gdal, osr
	import os
	import cv2
	import numpy as np
	src_fold = r"F:\ENVI\Data\04merge"
	sp_name = "qb_boulder_msi.img"
	pan_name = "qb_boulder_pan.img"
	out_path = r"F:\ENVI\Data\result\merge01.tif"
	def write_raster(filename, im_proj, im_geotrans, im_data):
	    # gdal.GDT_Byte,
	    # gdal .GDT_UInt16, gdal.GDT_Int16, gdal.GDT_UInt32, gdal.GDT_Int32,
	    # gdal.GDT_Float32, gdal.GDT_Float64
	    if 'int8' in im_data.dtype.name:
	        datatype = gdal.GDT_Byte
	    elif 'int16' in im_data.dtype.name:
	        datatype = gdal.GDT_UInt16
	    else:
	        datatype = gdal.GDT_Float32

	    if len(im_data.shape) == 3:
	        im_bands, im_height, im_width = im_data.shape
	    else:
	        im_bands, (im_height, im_width) = 1, im_data.shape

	    driver = gdal.GetDriverByName("Gtiff")
	    dataset = driver.Create(filename, im_width, im_height, im_bands, datatype)

	    dataset.SetGeoTransform(im_geotrans)
	    dataset.SetProjection(im_proj)

	    if im_bands == 1:
	        dataset.GetRasterBand(1).WriteArray(im_data)
	    else:
	        for i in range(im_bands):
	            dataset.GetRasterBand(i + 1).WriteArray(im_data[i])
	    del dataset
	dst01 = gdal.Open(os.path.join(src_fold, sp_name))
	dst02 = gdal.Open(os.path.join(src_fold, pan_name))
	sp_ras = dst01.ReadAsArray()
	sp_ras_upper = sp_ras.copy()
	sp_ras_upper_s = np.zeros((sp_ras_upper.shape[0], sp_ras_upper.shape[1]*4, sp_ras_upper.shape[2]*4))
	for i in range(sp_ras_upper.shape[0]):
	    sp_ras_upper_s[i] = cv2.resize(sp_ras_upper[i], dsize=None, fx=4, fy=4)

	sp_ras_upper_s = sp_ras_upper_s.reshape(sp_ras_upper_s.shape[0], -1)
	sp_ras = sp_ras.reshape(sp_ras.shape[0], -1)
	pan_ras = dst02.ReadAsArray()
	pan_ras = pan_ras.ravel()
	sp_cov = np.cov(sp_ras)
	sp_eig_val = np.linalg.eig(sp_cov)
	sp_val = sp_eig_val[0]
	sp_eig = sp_eig_val[1]
	sp_order = np.argsort(sp_val)[::-1]
	sp_eig_order = sp_eig[sp_order]
	pcs = np.dot(sp_eig_order, sp_ras_upper_s)
	eig_inv = np.linalg.inv(sp_eig_order)
	print("***"*10)
	for i in range(sp_ras_upper.shape[0]):
	    sp_ras_upper_s[i] = sp_ras_upper_s[i] + eig_inv[i][0] * (pan_ras - pcs[0])

| 基于变量替换技术融合方法02-IHS
::

	from osgeo import gdal, osr
	import os
	import cv2
	import glob
	import numpy as np
	src_fold = r"F:\ENVI\Data\04merge"
	sp_name = "qb_boulder_msi.img"
	pan_name = "qb_boulder_pan.img"
	out_path = r"F:\ENVI\Data\result\merge03.tif"
	dst01 = gdal.Open(os.path.join(src_fold, sp_name))
	dst02 = gdal.Open(os.path.join(src_fold, pan_name))
	sp_ras = dst01.ReadAsArray()
	print("orign_1", sp_ras.shape)
	sp_ras_upper = sp_ras.copy()
	sp_ras_upper_s = np.zeros((sp_ras_upper.shape[0], sp_ras_upper.shape[1]*4, sp_ras_upper.shape[2]*4))
	for i in range(sp_ras_upper.shape[0]):
	    sp_ras_upper_s[i] = cv2.resize(sp_ras_upper[i], dsize=None, fx=4, fy=4)

	sp_ras_upper_s = sp_ras_upper_s.reshape(sp_ras_upper_s.shape[0], -1)
	sp_ras = sp_ras.reshape(sp_ras.shape[0], -1)
	pan_ras = dst02.ReadAsArray()
	pan_ras = pan_ras.ravel()

	vec = np.array([[1/3,1/3,1/3],[-np.sqrt(2)/6, -np.sqrt(2)/2, np.sqrt(2)/3],[1/np.sqrt(2),-1/np.sqrt(2),0]])

	sp_ras_upper_s[:3] = vec.dot(sp_ras_upper_s[:3])
	pan_ras = pan_ras - sp_ras_upper_s[0]


	for i in range(sp_ras_upper.shape[0]):
	    sp_ras_upper_s[i] = sp_ras_upper_s[i] + pan_ras

	print(sp_ras_upper_s.shape)
	write_raster(out_path, dst02.GetProjection(), dst02.GetGeoTransform(), sp_ras_upper_s.reshape(4, 4096, 4096))

| 基于调制融合方法block-regression

::

	from osgeo import gdal, osr
	import os
	import cv2
	import glob
	import numpy as np
	from sklearn.linear_model import LinearRegression
	src_fold = r"F:\ENVI\Data\04merge"
	sp_name = "qb_boulder_msi.img"
	pan_name = "qb_boulder_pan.img"
	out_path = r"F:\ENVI\Data\result\merge02.tif"
	dst01 = gdal.Open(os.path.join(src_fold, sp_name))
	dst02 = gdal.Open(os.path.join(src_fold, pan_name))
	sp_ras = dst01.ReadAsArray()
	sp_ras_upper = sp_ras.copy()
	sp_ras_upper_s = np.zeros((sp_ras_upper.shape[0], sp_ras_upper.shape[1]*4, sp_ras_upper.shape[2]*4))
	for i in range(sp_ras_upper.shape[0]):
	    sp_ras_upper_s[i] = cv2.resize(sp_ras_upper[i], dsize=None, fx=4, fy=4)

	sp_ras = sp_ras.reshape(sp_ras.shape[0], -1)
	pan_ras = dst02.ReadAsArray()
	pan_ras_copy = pan_ras.copy()
	lr = LinearRegression()
	block = 8
	for i in range(int(pan_ras.shape[0]/block)):
	    for j in range(int(pan_ras.shape[1]/block)):
	        x_train = np.array(list([sp_ras_upper_s[band][j*block+ii][i*block+jj]]
	                                for band in range(sp_ras_upper_s.shape[0])
	                                for ii in range(block)
	                                for jj in range(block)
	                                )).reshape(-1, sp_ras_upper_s.shape[0])
	        y_train = np.array(list([pan_ras[j*block+ii][i*block+jj]] for ii in range(block) for jj in range(block)))
	        # print(x_train)
	        # print(y_train)
	        lr.fit(x_train, y_train)
	        y_predict = lr.predict(x_train)
	        # print(pan_ras_copy[i*block:i*block + block,j*block:j*block+block].shape)
	        # print(i*block,i*block + block,j*block,j*block+block)
	        pan_ras_copy[i*block:i*block + block,j*block:j*block+block] = (y_train / y_train-lr.intercept_).reshape(block, block)
	print(pan_ras_copy)
	for i in range(sp_ras_upper.shape[0]):
	    sp_ras_upper_s[i] = sp_ras_upper_s[i] * pan_ras_copy

	write_raster(out_path, dst02.GetProjection(), dst02.GetGeoTransform(),sp_ras_upper_s)

6. 图像镶嵌
-----------------------------------

在遥感影像获取过程中，受内部和外部环境因素干扰，多幅遥感影像的色调、亮度、反差等存在不同程度的差异。因此，为了建立无缝的正射影像镶嵌图、需要在单幅影像上和多幅影像之间进行匀光、匀色处理以达到色彩平衡。

| 下面是线性变换和直接拼接两种方法的python实现
::
	
	# 直接拼接
	from osgeo import gdal, osr
	import os
	import glob
	src_fold = r"F:\ENVI\Data\03mosaic"
	files = glob.glob(os.path.join(src_fold, "*.img"))
	destination_fold = r"F:\ENVI\Data\result"
	out_name = "mosaic.tif"
	print(files)
	gdal.Warp(os.path.join(destination_fold, out_name), files, format="Gtiff")

	# 使用线性变换拼接
	from osgeo import gdal, osr
	import os
	import cv2
	import glob
	from sklearn.linear_model import LinearRegression
	src_fold = r"F:\ENVI\Data\03mosaic"
	files = glob.glob(os.path.join(src_fold, "*.img"))
	destination_fold = r"F:\ENVI\Data\result"
	out_name = "mosaic2.tif"
	dst01 = gdal.Open(files[0])
	dst02 = gdal.Open(files[1])
	ras01 = dst01.ReadAsArray()
	geo1 = dst01.GetGeoTransform()
	xl1,yu1,xr1,yb1 = geo1[0],geo1[3],geo1[0] + geo1[1] * dst01.RasterXSize, geo1[3] + geo1[5] * dst01.RasterYSize

	ras02 = dst02.ReadAsArray()
	geo2 = dst02.GetGeoTransform()
	xl2,yu2,xr2,yb2 = geo2[0],geo2[3],geo2[0] + geo2[1] * dst02.RasterXSize, geo2[3] + geo2[5] * dst02.RasterYSize
	xl,yu,xr,yb = max(xl1,xl2), min(yu1,yu2), min(xr1, xr2), max(yb1,yb2)
	ras01_y_train = ras01[:, int((yu1 - yu)/-geo1[5]):int((yu1 - yb)/-geo1[5]), int((xl - xl1)/geo1[1]):int((xr - xl1)/geo1[1])]
	lr = LinearRegression()
	ras02_x_train = ras02[:, int((yu2 - yu)/-geo1[5]):int((yu2 - yb)/-geo1[5]),int((xl - xl2)/geo1[1]):int((xr - xl2)/geo1[1])]
	for i in range(ras02_x_train.shape[0]):
	    lr.fit(ras02_x_train[i].reshape(-1,1), ras01_y_train[i].reshape(-1,1))
	    ras02[i] = lr.predict(ras02[i].reshape(-1,1)).reshape(*ras02[i].shape)
	dst02_ = write_raster("", dst02.GetProjection(), geo2, ras02)
	gdal.Warp(os.path.join(destination_fold, out_name), [dst01, dst02_], format="Gtiff")