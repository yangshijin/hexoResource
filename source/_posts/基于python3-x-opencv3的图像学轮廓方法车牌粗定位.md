---
title: 基于python3.x + opencv3的图像学轮廓方法车牌粗定位
date: 2017-04-11 22:22:59
tags:
  - python
  - opencv
  - 车牌识别
  - 机器学习
---

##### 前言
　　最近准备迈入机器学习的大门，有经验的同事推荐我先从简单的小项目做起，于是乎他推荐了车牌识别的项目，然后就开始了相关的研究，车牌首先是从一个大图中先定位出来，然后才能识别其中的车牌号码，车牌怎么从一张大的图片就定位出来呢，看了基于c++的开源项目easyPR的作者博客，按着他的思路，可以先从图像学的角度来定位车牌。
<!-- more -->
##### 环境准备
1. 由于在选型的时候，选了最近很火的谷歌的开源深度学习框架“TensorFlow”,又由于本人是苦逼的windows党，TensorFlow目前最新的版本，需要制定python版本3.5，于是第一个需要安装的就是python3.5啦。
2. 另外一个就是基于python安装的opencv3

##### 定位思路（基于轮廓定位）
###### 1.图片灰度化（将三元色的图片转换为只有一个灰度通道的图片，处理起来更容易）
###### 2.图片高斯模糊（去除干扰的噪声，easyPR的作者思路是先高斯模糊再灰度化）
###### 3.Sobel算子处理
###### 4.图像二值化
###### 5.闭操作
###### 6.腐蚀->膨胀
###### 7.取轮廓列表
###### 8.基于车牌形状特性筛选

上面七个步骤大致就是我初步实现的思路，借鉴了easyPR的作者的前部分思路，当然这只是粗定位，后面还需要很多需要优化，但作为初学者，第一步肯定是先从简单的来实现啦。

###### 1. 图片灰度化
  这一步使用python+opencv实现起来非常简单，有两种方法，第一种是加载图片的时候加一个参数，直接灰度化，第二种是加载后另外调用opencv的API->cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)来进行灰度化，居然方法细节可以直接看方法定义
###### 2. 高斯模糊
　　高斯模糊，也称高斯平滑，根据高斯曲线调节像素色值，它是有选择地模糊图像。说得直白一点，就是高斯模糊能够把某一点周围的像素色值按高斯曲线统计起来，采用数学上加权平均的计算方法得到这条曲线的色值，最后能够留下人物的轮廓，即曲线．具体的原理介绍各位可以自己google一下。
　　opencv也提供了高斯模糊的API->cv2.GaussianBlur(),具体参数含义可以看一看API手册，这里只讲第二个参数，即模糊范围，这个范围分为x、y方向，我选择了3-3作为模糊的计算范围。
###### 3. sobel算子处理
　　sobel算子是一种图像处理算法，主要用作边缘检测.车牌的特征非常适合用sobel算子来做处理，通过sobel算子能清晰的识别出车牌的边缘，字母的边缘，得出的灰度图如下所示：
![](/img/sobel.jpg)

###### 4. 二值化
 　二值化的原理很简单，就是对一张灰度图像，基于某个阈值，当大于这个阈值的时候，置为1，小于这个阈值的时候置为0，比如我设置一个阈值为150，那么当这张灰度图的图片的某个像素点的值为170，那么二值化操作后的值就是1，也就是颜色通道的最大值255。为什么要进行二值化呢，和灰度化一样，进一步简化图像的表达，让之后的形态学图像处理更加简单。二值化后的图像如下：
 ![](/img/binary.jpg)

###### 5. 闭操作
　　要了解闭操作，首先要了解图像学的腐蚀、膨胀，什么是图像的膨胀处理呢？通俗点来讲，就是有一个二维的矩阵定义，比如3*3,然后对一个图像进行像素点遍历，基于每一个像素点作为矩阵中心点，然后对这个像素点的周边被矩阵包含的像素点做一个局部最大值的替换，腐蚀操作则反之，取局部最小值。

　　那么闭操作的原理其实就是对图像先进行膨胀操作，再进行腐蚀操作，这样有什么作用呢，相信大家也看到二值化后的图像，车牌的字符边缘很清晰的展现出来，但是我们这是要根据车牌轮廓来做粗定位的，那么我们要让车牌区域变成一个白色的矩形，闭操作第一步的膨胀操作，将车牌字符与字符之间的空白连接起来，想成一个基于车牌区域往外扩展一点的矩形区域，第二步闭操作，将由于膨胀操作使得车牌矩形往外扩展的区域腐蚀掉。

　　这个时候需要讲一下，这里的闭操作的参数之一，闭操作的像素范围，即进行局部极值计算的二维矩阵定义，由于车牌的特性，我们定义的矩阵最好是按照车牌的长宽比例来进行处理，大约是2.5-5.5这个比例内的参数调优，我自己试了一下，4左右的比例应该效果是最好的，另外就是范围的大小，由于要定位的图片不一，因此最好是根据你当前的图片库的平均大小来做确定，我自己网上找图片试了一下，大致是20左右的x像素点，4-5左右的y像素点。处理结果如下：
![](/img/closed.jpg)

###### 6. 腐蚀->膨胀
　　细心的朋友应该发现了，上面闭操作之后的图片，还是有一些小的白点细长的白点存留，这些俗称为噪点，也就是干扰项，这个时候需要进行腐蚀操作，将细小的点给腐蚀掉，然后腐蚀掉肯定车牌区域的大小会受影响，因此后面要膨胀处理，来补偿因为处理掉细节而腐蚀掉的区域。其实腐蚀操作也就是开操作，这个操作一般也就是用来处理掉细小的噪点的。那这里我为什么要分开呢，因为要进行车牌定位的图片的噪点往往进行一轮腐蚀没办法完全腐蚀掉，而闭操作只是进行一轮腐蚀后就进行膨胀了，这样没有完全腐蚀掉的细节又重新膨胀回来了。因此分开操作，opencv的腐蚀、膨胀方法，有个参数可以定义进行几轮处理，因此这里可以根据图片的情况，具体调优。调优后的图片如下：
![](/img/dilation.jpg)

###### 7. 取轮廓列表
　　找轮廓列表其实就是讲上面处理后的二值化图像的白色部分筛选出来，opencv有现成的API，cv2. cv2.findContours()

###### 8. 筛选
　　找到所有的轮廓之后，就要基于车牌的特性进行筛选了，首先将轮廓转为一个矩形，然后将面积小的过滤掉，然后根据车牌的形态特性，即长宽比例筛选出正确的车牌矩形，当然，这里有可能筛选不到任何的矩形，也有可能筛选到多个矩形出来。比如本图的例子图片，就筛选出了两个矩形出来，如下(我用绿色的线画出来了)：
![](/img/lunkuo-result.jpg)

这里的只是初步将车牌区域定位出来，甚至可能定位出多个不是车牌的区域，后面需要继续优化。

###### 实现代码如下：

```python
import cv2
import numpy as np

minPlateRatio = 2.5 # 车牌最小比例
maxPlateRatio = 5   # 车牌最大比例

# 图像处理
def imageProcess(gray):
    # 高斯平滑
    gaussian = cv2.GaussianBlur(gray, (3, 3), 0, 0, cv2.BORDER_DEFAULT)

    # Sobel算子，X方向求梯度
    sobel = cv2.convertScaleAbs(cv2.Sobel(gaussian, cv2.CV_16S, 1, 0, ksize=3))

    # 二值化
    ret, binary = cv2.threshold(sobel, 150, 255, cv2.THRESH_BINARY)

    # 对二值化后的图像进行闭操作
    element = cv2.getStructuringElement(cv2.MORPH_RECT, (18, 4))
    closed = cv2.morphologyEx(binary, cv2.MORPH_CLOSE, element)

    # 再通过腐蚀->膨胀 去掉比较小的噪点
    erosion = cv2.erode(closed, None, iterations=2)
    dilation = cv2.dilate(erosion, None, iterations=2)

    # 返回最终图像
    return dilation

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

def detect(img):
  # 转化成灰度图
  gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
  # 形态学变换的处理
  dilation = imageProcess(gray)
  # 查找车牌区域
  region = findPlateNumberRegion(dilation)
  # 默认取第一个
  box = region[0]
  #在原图画出轮廓
  cv2.drawContours(img, [box], 0, (0, 255, 0), 2)
  # 找出box四个角的x点，y点，构成数组并排序
  ys = [box[0, 1], box[1, 1], box[2, 1], box[3, 1]]
  xs = [box[0, 0], box[1, 0], box[2, 0], box[3, 0]]
  ys_sorted_index = np.argsort(ys)
  xs_sorted_index = np.argsort(xs)
  # 取最小的x，y 和最大的x，y 构成切割矩形对角线
  min_x = box[xs_sorted_index[0], 0]
  max_x = box[xs_sorted_index[3], 0]
  min_y = box[ys_sorted_index[0], 1]
  max_y = box[ys_sorted_index[3], 1]

  # 切割图片，其实就是取图片二维数组的在x、y维度上的最小minX,minY 到最大maxX,maxY区间的子数组
  img_plate = img[min_y:max_y, min_x:max_x]
  return img_plate


if __name__ == '__main__':
        imagePath = 'd:\\CarImage\\car_04.jpg' # 图片路径
        img = cv2.imread(imagePath)
        img = detect(img)
        cv2.imshow("img",img)
        cv2.waitKey(0)

```
