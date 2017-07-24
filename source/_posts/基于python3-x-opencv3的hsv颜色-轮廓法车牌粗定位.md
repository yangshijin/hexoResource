---
title: 基于python3.x+opencv3的hsv颜色+轮廓法车牌粗定位
date: 2017-04-15 15:45:39
tags:
- python
- opencv
- 车牌识别
- 机器学习
---

##### 前言
　　上一篇文章将了基于图像学sobel算子等图像处理的边缘检测法通过轮廓检测进行车牌粗定位，结尾也说了，这种方法有可能出现错误定位，或者定位出多个区域，今天我们来试试通过将图片转换成hsv模式，看看是否能通过色调区域方法找到车牌区域，从而完成车牌粗定位。
<!-- more -->

###### 1.加载图片，转换成hsv模式
　　HSV(Hue, Saturation, Value)是根据颜色的直观特性由A. R. Smith在1978年创建的一种颜色空间, 也称六角锥体模型(Hexcone Model)。这个模型中颜色的参数分别是：色调（H），饱和度（S），明度（V）。

　　对图形图像学没有太多了解的朋友可能比较少听过HSV模型，更多听到的是RGB模型，或者RGBA模型，和RGB模型这种三维坐标的颜色模型不同的是，HSV模型，是针对用户观感的一种颜色模型，侧重于色彩表示，什么颜色、深浅如何、明暗如何。H是色彩，S是深浅， S = 0时，只有灰度，V是明暗，表示色彩的明亮程度，但与光强无直接联系。

　　opencv提供了这样一个方法，可以直接将rgb模式的图片转换成hsv模式的图片，方法为cv2.cvtColor(img, cv2.COLOR_BGR2HSV),也就是转换成灰度图的方法，只是参数不一样。
###### 2. 定义车牌的hsv区间
　　这里不对HSV模型进行深入讲解，有兴趣的同学可以直接google一下，资料很多，这里我们只需要找到车牌的hsv颜色区间，假设只对蓝底白字车牌进行定位，那么颜色空间就是：H区间：[100,140] , S区间：[50,255] , V区间：[50,255] ，S和V之所以跨度这么大，是因为不同的时间、光照、摄像机拍出来的效果值可能差距会比较大。

###### 3.找到图中包含这个hsv区间的所有像素
　　遍历经过转换成hsv模式图片的所有像素点，将所有处于这个颜色区间的像素点标识出来，置为1（255）,不处于这个颜色区间的像素点置为0,从而形成了一张二值化的图片。
　　opencv提供了这样的一个方法，因此我们不需要手动编码去遍历图片的所有像素，方法为：cv2.inRange(hsv_img, lower_color, higher_color).第一个参数为hsv模式的图片，第二个参数为颜色区间的最低值，第三个参数为颜色区间的最大值。效果如下：

![](/img/hsv2binary.jpg)

###### 4.矩形链接、噪点处理
　　由步骤三得到的二值化图可知，绝大部分图片具有蓝色区间的区域不仅仅是在车牌处，其他的地方也有可能具有这个颜色的像素点，甚至蓝色的标识牌、蓝色的车身会极大的干扰到车牌的定位。
　　由于车牌字体肯定不属于蓝色区间，这里我们首先要做的就是将蓝色的车牌矩形处中间由于车牌号而镂空的地方链接起来，因此首先进行一轮闭操作，闭操作原理上一篇文章有讲过。效果如下：

![](/img/hsv-closed.jpg)

　　可以看到车牌区域中间镂空的预期已经补齐，接下来去除细小噪点，原理同样在上一篇文章有讲过，即开操作，先对图片进行腐蚀，再进膨胀，去除掉细小的像素点。效果如下：

![](/img/hsv-opened.jpg)

###### 5.进行边缘检测，寻找图中所有矩形
　　对二值化图像进行链接、去燥处理后，图片已经很干净，这里直接用上一篇文章用到的形态学方法，寻找图中矩形，然后进行过滤判断，最终定位到车牌区域，效果如下：

![](/img/hsv-result.jpg)

PS：同样的，这里只是粗定位，而且是只针对蓝底车牌的粗定位，像其他黄底、黑底、白底的车牌，在这里没有任何可能定位出来，另外，即使是蓝底车牌还是有可能定位到多个矩形区域，或者干脆定位不到车牌区域，因此，要保持较高的识别率，仅仅使用hsv模式定位是不行的，可能通过结合上一篇文章讲到的边缘检测的识别方法，来对车牌进行一个定位，这样能保持比较高的识别率。

###### 6.实现代码：

```python
import cv2
import numpy as np

# 定义蓝底车牌的hsv颜色区间
lower_blue = np.array([100, 50, 50])
higher_blue = np.array([140, 255, 255])

# 找到符合车牌形状的矩形
def findPlateNumberRegion(img):
    region = []
    # 查找外框轮廓
    contours_img, contours, hierarchy = cv2.findContours(img, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_NONE)
    print("contours lenth is :%s" % (len(contours)))
    # 筛选面积小的
    for i in range(len(contours)):
        cnt = contours[i]
        # 计算轮廓面积
        area = cv2.contourArea(cnt)

        # 面积小的忽略
        if area < 2000:
            continue

        # 转换成对应的矩形（最小）
        rect = cv2.minAreaRect(cnt)
        # print("rect is:%s" % {rect})

        # 根据矩形转成box类型，并int化
        box = np.int32(cv2.boxPoints(rect))

        # 计算高和宽
        height = abs(box[0][1] - box[2][1])
        width = abs(box[0][0] - box[2][0])
        # 正常情况车牌长高比在2.7-5之间,那种两行的有可能小于2.5，这里不考虑
        ratio = float(width) / float(height)
        if ratio > maxPlateRatio or ratio < minPlateRatio:
            continue
        # 符合条件，加入到轮廓集合
        region.append(box)
    return region

if __name__ == '__main__':
    img = cv2.imread("d:\\carImage\\car_02.jpg")
    # 转换成hsv模式图片
    hsv_img = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

    # 找到hsv图片下的所有符合蓝底颜色区间的像素点，转换成二值化图像
    in_range_array = cv2.inRange(hsv_img, lower_blue, higher_blue)

    # 进行闭操作，链接车牌区域
    element = cv2.getStructuringElement(cv2.MORPH_RECT, (17, 3))
    closed = cv2.morphologyEx(in_range_array, cv2.MORPH_CLOSE, element)

    # 进行开操作，去除细小噪点
    eroded = cv2.erode(closed, None, iterations=2)
    dilation = cv2.dilate(eroded, None, iterations=2)

    # 查找并筛选符合条件的矩形区域
    region = findPlateNumberRegion(dilation)

    # 标识出所有符合条件的矩形区域
    if len(region) != 0:
        for i in range(len(region)):
            box = region[i]
            rect_img = findRectImg(box, tmp_img)
            rect_imgs.append(rect_img)
            cv2.drawContours(img, [box], 0, (0, 255, 0), 1)

    cv2.imshow("img", img)

```

　　
