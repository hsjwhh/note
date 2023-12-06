- 多图合并长图
```batch
magick convert -append *.jpg【需处理图像】 to.jpg【输出图像文件】
# +append 图像左右排列，横向
# -append 图像上下排列，竖向
```
> 以下信息引用自：https://www.ipcpu.com/2022/03/imagemagick/

一、概述

图片在网站所占的比重越来越重。这里所说的优化是指在不影响用户体验的情况下，将图片做到最小，有些网站上的图片是用户直接上传的，有些用户会直接上传10M的原图，假如一个页面有10张图片，那么这个页面每刷新一次就要消耗100M的带宽流量，这无疑是个巨大的浪费。

在客户端我们可以用 PhotoShop 等 GUI 工具处理图片，不过在服务器端要处理海量的图片格式转换，缩放裁剪，翻转扭曲等操作， GUI 软件就很难下手了，所以此处需要召唤命令行工具来帮我们完成这些事。

ImageMagick: 是一款创建、编辑、合成，转换图像的命令行工具。支持格式超过 200 种，包括常见的 PNG, JPEG, GIF, HEIC, TIFF, DPX, EXR, WebP, Postscript, PDF, SVG 等。功能包括调整，翻转，镜像(mirror)，旋转，扭曲，修剪和变换图像，调整图像颜色，应用各种特殊效果，或绘制文本，线条，多边形，椭圆和贝塞尔曲线等。

接下来，我们来使用ImageMagick来对图片进行分析和优化，减少图片文件大小。

二、ImageMagick的安装

以CentOS为例
```shell
 #安装常见图片格式支持
 yum install libjpeg libjpeg-devel libpng libpng-devel libtiff libtiff-devel libungif libungif-devel freetype zlib -y
 #增加webp格式支持
 yum install libwebp-tools -y
 #安装ImageMagick
 yum install ImageMagick  -y
 ```
安装完成以后，我们可以看到/usr/bin/目录下生产了animate、compare、composite、conjure、convert、display、identify、import、mogrify、montage、stream等文件，这些命令都可以直接调用。

三、查看图片信息

再进行优化之前，我们需要对图片进行一个整体的了解，identify命令可以查看图片的一些信息。
```shell
# 查看图片宽度、高度、位深、大小信息
[root@ipcpu.com root]# identify IMG20211022075834.jpg
IMG20211022075834.jpg JPEG 4000x3000 4000x3000+0+0 8-bit sRGB 2.59941MiB 0.000u 0:00.000
# 仅显示图片宽度和高度
[root@ipcpu.com root]# identify -format "%wx%h" IMG20211022075834.jpg
4000x3000
# 显示图片详细信息（包括EXIF）
[root@ipcpu.com root]# identify -verbose IMG20211022075834.jpg
Image: IMG20211022075834.jpg
  Format: JPEG (Joint Photographic Experts Group JFIF format)
  Mime type: image/jpeg
  Class: DirectClass
  Geometry: 4000x3000+0+0
  Resolution: 72x72
  Print size: 55.5556x41.6667
  Units: PixelsPerInch
  Colorspace: sRGB
  Type: TrueColor
<省略>
  Compression: JPEG
  Quality: 95
  Orientation: TopLeft
  Properties:
    date:create: 2021-10-22T08:03:31+00:00
    date:modify: 2021-10-22T08:03:18+00:00
    exif:ApertureValue: 17/10
<省略>
  Filesize: 2.59941MiB
  Number pixels: 12000000
  Pixels per second: 44.2019MB
  User time: 0.220u
```
四、去除EXIF信息

EXIF是图片的元数据，所记录的元数据信息非常丰富，主要包含了以下几类信息：

拍摄信息、拍摄器材（机身、镜头、闪光灯等）、拍摄参数（快门速度、光圈F值、ISO速度、焦距、测光模式等）、图像处理参数（锐化、对比度、饱和度、白平衡等）、图像描述及版权信息、GPS定位数据、缩略图。 还有图片的方向(旋转角度)也是存储在EXIF中的。

去除EXIF信息对于图片大小来说影响不大，可以忽略。
```shell
[root@ipcpu.com root]# convert -strip IMG20211022075834.jpg new.jpg
[root@ipcpu.com root]# ll  |grep .jpg
-rw-r--r-- 1 root  root 2725684 Jul 21 16:03 IMG20211022075834.jpg
-rw-r--r-- 1 root  root 2718697 Jul 21 16:10 new.jpg
```
五、通过调整图片宽度和高度来压缩图片

我们通过手机拍的照片有的可以高达10M，宽和高是4000x3000，可是一般我们的显示器是1920x1080(FHD)或者高端的2560x1440(2.5K)，很明显显示不了那么细致，因此我们要把图片宽度和高度缩小，以便于节省空间和带宽。
```shell
convert -resize "1920x1080>" IMG20211022075834.jpg resize.webp
```
常用的组合方法有：
```
假设原图是400*300
50%        按照50%比例进行缩放，(新图200*150)
100             按指定的宽度缩放，宽高比不变，(新图100*75)
 x500          按指定高度缩放，宽高比不变，(新图667*500)
100x100      给定高度和宽度的最高值，宽高比不变，(新图100*75)
100x100^     给定高度和宽度的最低值，宽高比不变，(新图133*100)
100x100!      宽度和高度强制转换，忽略宽高比，(新图100*100)
400x200>     缩小图片，当图片宽或高超过规定的尺寸，给定高度和宽度的最低值，宽高比不变
100x100^>   缩小图片，当图片宽或高超过规定的尺寸， 给定高度和宽度的最高值，宽高比不变
800x500<     放大图片，当图片宽或高小于规定的尺寸。给定高度和宽度的最低值，宽高比不变
700x600^<   放大图片，当图片宽或高小于规定的尺寸。给定高度和宽度的最高值，宽高比不变
实际案例，先看下缩小图片
```
```shell
# 缩小图片
[root ]# convert -resize '180x150>' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
180x135
[root ]# convert -resize '210x150>' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
200x150
```

原图是400x300, 按照180x150(宽和高分别是原大小的45%和50%)进行缩小， 给定最低值，也就是45%宽度，所以出来的新图片宽和高是180x135。
原图是400x300, 按照210x150(宽和高分别是原大小的52%和50%)进行缩小， 给定最低值，也就是50%高度，所以出来的新图片宽和高是200x150。

```shell
[root ]# convert -resize '180x150^>' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
200x150
[root ]# convert -resize '210x150^>' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
210x158
```
原图是400x300, 按照180x150(宽和高分别是原大小的45%和50%)进行缩小， 给定最高值，也就是50%高度，所以出来的新图片宽和高是200x150。
原图是400x300, 按照210x150(宽和高分别是原大小的52%和50%)进行缩小， 给定最高值，也就是52%宽度，所以出来的新图片宽和高是210x158。
再来看下放大图片
```shell
# 放大图片
[root ]# convert -resize '800x500<' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
667x500
[root ]# convert -resize '800x700<' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
800x600
```
原图是400x300, 按照800x500(宽和高分别是原大小的200%和166%)进行放大， 给定最低值，也就是166%高度，所以出来的新图片宽和高是667x500。
原图是400x300, 按照800x700(宽和高分别是原大小的200%和233%)进行放大， 给定最低值，也就是200%宽度，所以出来的新图片宽和高是800x600。
```shell
[root ]# convert -resize '800x500^<' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
800x600
[root ]# convert -resize '800x700^<' 400x300.jpg last.jpg && identify -format "%wx%h" last.jpg
933x700
```
原图是400x300, 按照800x500(宽和高分别是原大小的200%和166%)进行放大， 给定最高值，也就是200%宽度，所以出来的新图片宽和高是800x600。
原图是400x300, 按照800x700(宽和高分别是原大小的200%和233%)进行放大， 给定最高值，也就是233%高度，所以出来的新图片宽和高是933x700。

五、通过降低图片的quality品质来压缩图片

对于好多高质量的图片，降低图片的quality品质，可以明显减少图片的大小，例如85%的品质对于肉眼来说很难分辨，再加上生成的图片大部分是用于WEB传输展示而非打印，变更品质和大小就是图片优化的两大手段。
```shell
[root ]#  convert  -quality 85 orignal.jpg orignal-85.jpg
[root ]#  ll -h |grep orignal
-rw-r--r-- 1 root  root 1.3M Jul 22 12:44 orignal-85.jpg
-rw-r--r-- 1 root  root 2.6M Jul 21 16:03 orignal.jpg
品质参数和大小参数可以一起使用

[root ]#   convert  -quality 85  -resize '1920x1280^>' orignal.jpg orignal-85-re.jpg
[root ]#   ll -h |grep orignal
-rw-r--r-- 1 root  root 1.3M Jul 22 12:44 orignal-85.jpg
-rw-r--r-- 1 root  root 296K Jul 22 12:48 orignal-85-re.jpg
-rw-r--r-- 1 root  root 2.6M Jul 21 16:03 orignal.jpg
效果是不是很明显？
```