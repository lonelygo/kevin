---
layout: post
title: 使用Tesseract训练并识别集装箱号
subtitle: 使用Tesseract训练lang文件并OCR识别集装箱号
tags: [Tesseract，OCR]
comments: true
---

在某产品的规划中，想要通过监控视频进行 ***“集装箱计数”*** 与 ***“集装箱号”*** 识别，以便与舱单数据进行自动化的对比，发现潜在的监管风险。在研究了不少商用的集装箱号识别系统后，发现为了保证识别准确率几乎都采用了 ***固定角度*** + ***多位置识别*** 的原始图像获取方式以降低图片质量干扰，提高识别准确率。
考虑到用户业务的应用场景，不能在像“卡口”这样的位置进行三机位的固定场景的集装箱号识别。

所以，为了做这方面尝试，首先从集装箱号的OCR识别开始研究，理论上，集装箱编号就是“英文字母+数字”的组合，但是实践验证利用Tesseract自带的eng词库进行识别，准确率有限，还是要考虑自己做针对性的训练，生成专用词库。  

在这里记录下Tesseract训练集装箱号词库过程。

# 1.环境准备
## 1.1系统环境
之前在Ubuntu 16.04 下跑过一遍完整的流程，最近又收集了一些样本图片，所以再训练看看效果，本次准备在Mac环境下进行。

所以，本文的所有操作，没有特殊说明都是在macOS 10.12.6下，Mac下与Ubuntu下过程、命令都一样，唯一不同的就是最后lang文件保存的位置了。

另外，建议Mac党准备一个Windows电脑，虚机没测试，但是理论上应该不会有问题，为什么需要下面会进行说明。

## 1.2基础环境安装

### 1.2.1 brew 安装
**[Homebrew](https://brew.sh)** 是macOS下的包管理工具，教程很多，自行搜索就好,基本就是终端执行一句话：

`
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
`

### 1.2.2 imagemagick 安装

 **[imagemagick](https://www.imagemagick.org/script/index.php)** 是一个牛逼到“奇葩”😺😺的图像处理库，可以说只有你想不到，没有他处理不了的，不明觉厉，不信可以看看[这里](http://www.imagemagick.org/script/command-line-processing.php)的命令行用法和[这里](http://www.imagemagick.org/script/resources.php)的配置说明。

有了这个库，后面做图像格式转换和预处理会非常方便，命令行就能解决问题，用到了再具体说。

`
brew install imagemagick
`

## 1.3 Tesseract 安装

[Tesseract-ocr](https://en.wikipedia.org/wiki/Tesseract_(software))2005年由HP开源，2006年以后是Google赞助并开始开发，Github地址在 ***[这里](https://en.wikipedia.org/wiki/Tesseract_(software))*** 目前最新的版本是Tesseract 4.0 alpha，和之前版本最大的变化是引入了LSTM（长短期记忆）这个在语音识别、自然语言处理和机器翻译领域非常火爆的神经网络技术。

本次训练的记录是基于3.05版本的方法（虽然安装的是4.0 alpha），Github的项目Wiki页面也提供了对于4.0版本的[训练方法](https://github.com/tesseract-ocr/tesseract/wiki/TrainingTesseract-4.00)，一方面是英语渣，看来看去感觉官方文档写的太晦涩不好理解；一方面是懒，反正基于3.X版本训练出来的效果也还行(其实还是没快速看懂文档😓😓😓😓）。

Ubuntu 下可以从源码编译安装，参考官方说明执行就好，Mac下可以源码也可以brew安装，brew安装4.0 alpha：

`
brew install tesseract --HEAD --with-training-tools --with-all-languanges
`

--HEAD    不加这个，应该就是默认安装3.05了;
--with-training-tools    包括训练工具，否则就只能做识别不能训练;
--with-all-languanges    包括所有的语言包，否则只有默认的英语识别（硬盘空间有限的话，可以以后按需[下载](https://github.com/tesseract-ocr/tesseract/blob/master/doc/tesseract.1.asc#languages)后甩到 /tessdata/目录下）。
# 2.准备图片
## 2.1 预备知识
在开始准备用于训练的图片之前，建议先看看官方Wiki的[ImproveQuality](https://github.com/tesseract-ocr/tesseract/wiki/ImproveQuality)提高图像质量的说明，重要几点是：

- Tesseract在DPI为至少为300 DPI的图像上效果最佳，所以需要考虑提高图片的DPI，一般图片默认的都是72 。
- 将图像转换为黑白（就是二值化）。
- 消除噪点（就是降低干扰）。
- 旋转（就是将文字尽量水平）。
- 移除不必要的边界（就是要适当裁剪图片）。

## 2.2 准备图片
首先让兄弟们帮我拍了一些集装箱的各种角度的照片，有单个的，有堆场的，也有箱门位置的特写的，为什么要这些照片，在有后续的TensorFlow转移训练的文章中会具体说明。
收集到的原始素材，筛选了一部分集装箱号比较清晰并相对比较水平的，大概像这个样子：

![](http://upload-images.jianshu.io/upload_images/154674-3a2f98d0dd358273.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面就是惨绝人寰的体力劳动时间，一张一张图选择合适的集装箱号，把集装箱号单独裁剪出来，像这样：

![](http://upload-images.jianshu.io/upload_images/154674-0ce0127ebcc0215e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这一轮训练，我准备了387张集装箱号的图片。

## 2.3 调整图片DPI
按照2.1预备知识中的内容，官方推荐的DPI是300以上，一般图片都是72，所以需要请出imagemagick了，终端cd到保存剪切好的图片的目录，比如我的在这里：

`
~/Documents/Soyuan/container_number/line
`

图片名称都批量修改为：img-*.jpg，在终端执行：

`
convert '*.jpg' -density 300 ~/Documents/Soyuan/container_number/new/img-%d.jpg
`

对的，我把转换后的图片换了个目录，以防万一，检查没问题再手动删除吧，也不累。下一步，安装要求，再把图片转换成 **.tif** 格式，cd 到刚才的`new/`目录，执行：

`
convert '*.jpg' ~/Documents/Soyuan/container_number/line/img-%d.tif
`

嗯，我又给挪回去了。
convert 命令是imagemagick的其中之一，好用吧😄。

# 3.开始训练

官方教程在[这里](https://github.com/tesseract-ocr/tesseract/wiki/Training-Tesseract)，完全可以跟着Wiki的教程进行，或者看 [翻译版本](http://yanghespace.com/2015/11/01/Tesseract3训练新语言/) 没有没要往下看了😄😄。

## 3.1 合并tif文件
嗯，Windows第一次隆重出场👏👏👏。
由于是训练这种奇葩的文字场景，准备训练数据文件，貌似有且仅有手工操作这一条路了，好在[这里](https://github.com/tesseract-ocr/tesseract/wiki/AddOns)提供了一些有用的工具，反正都没用过，就选择了排序第一个的工具：[jTessBoxEditor](http://vietocr.sourceforge.net/training.html) 。原因有2个：1、好歹也是2016年的版本，其余的都是2013的，你怎么选？2、多终端支持，不挑不拣，Win／Mac／Linux 都能Run起来。

下载的时候稍微留意下，页面目录左侧的下载地址，下载的不是jTessBoxEditor，而是VietOCR-4.5，这个对咱们没用，真实下载地址应该是 [这里](https://sourceforge.net/projects/vietocr/files/jTessBoxEditor/) ，所以啊，仔细看文档是很重要的一件事情。
下载，解压缩，直接执行就好，免安装，对了需要 *JRE 8u40* 以上支持。

开始干正事，我选择在Win下进行，主要是用Win的人多，弄明白了以后可以发动群众一起来干活，手头反正有小黑，就用Win好了。

把转换好的tif文件摆渡到Win电脑，双击`jTessBoxEditor.jar`启动工具，菜单栏选择：`Tools-Merge TIFF`：
![](http://upload-images.jianshu.io/upload_images/154674-1373861c99e71afd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

弹出对话框选择tif图片文件夹：
![](http://upload-images.jianshu.io/upload_images/154674-23a493db4027d8a3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择一部分图片，生成一个新的tif文件，我每次选择100张，不要问我为什么，点击`打开`：
![](http://upload-images.jianshu.io/upload_images/154674-413815163ac21888.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

输入文件名，保存就好。保存路径可以和原始文件不是一个文件夹，强烈建议单独建一个文件夹，没什么，就是后面还要用到，好找。

***说明：***
这个文件名，是有讲究的，可以看官网文档的说明，有明确的格式要求：

`
 [lang].[fontname].exp[num].tif 
`

lang--你准备训练的语言的名称，这个名称也是将来你使用的时候 -l 后面指定的名称；

fontname--fontname就是fantname，需要你指定你的语言的字体名称，这个事情可以这么理解：不同的字体，字符的展现形式可是千变万化啊，比如楷体和王羲之字体。

exp[num]---exp不解释，人家就这么定义的，照着敲，后面是编号，看明白了吧，这玩意其实支持增量训练，今天弄一点，明天弄一点，慢慢就能让自己训练的语言文件准确率越来好。

所以，我的文件名是这样的(后面都假设你也是4组咯）：

```
cont.Arial.exp1.tif
cont.Arial.exp2.tif
cont.Arial.exp3.tif
cont.Arial.exp4.tif
```
好了，tif文件合并好了，摆渡回Mac（其实我是弄了个共享文件夹，后面来回次数不少呢，这样最方便）。

## 3.2 生成 .box 文件
终端cd到刚才copy回来的目录，依次执行：
```
tesseract cont.Arial.exp1.tif cont.Arial.exp1 batch.nochop makebox
tesseract cont.Arial.exp2.tif cont.Arial.exp2 batch.nochop makebox
tesseract cont.Aria3.exp1.tif cont.Arial.exp3 batch.nochop makebox
tesseract cont.Arial.exp4.tif cont.Arial.exp4 batch.nochop makebox
```
在目录下应该就这样了：
![](http://upload-images.jianshu.io/upload_images/154674-633ef2beece746c0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们就有了4组 .tif 和 .box 文件组成的文件对，这个很重要。

***此处有大坑，千万小心***

按正常流程，应该切换到下面 3.3 步骤了，但是我在Ubuntu 16.04 下面踩过一个硕大无比，真心想砸电脑的坑，但是在Mac下没遇到，谁让我上次手欠，用Ubuntu玩耍呢。一句话描述这个坑：

再下下一步，也就是 3.4 中，我们会用到 3.3 的成果去生成 .tr 文件，但是，TNND居然会报错，报错谁没见过，但是这个报错坑的有点大。在 3.3 中，我们会千辛万苦的手动把一张张图片识别出来的文字校对正确，然后，他报错了！！！报错了！！！上一次，我合并的tif文件有200+图片，第一次弄也不熟悉，修改了差不多两天时间，然后就一下子重头开始了（所以现在知道我为啥100张一合并了吧，其实我上次后来50张一合并，吓坏了）。

为了不踩坑，想到了一个简单有效还好用的方案：跳过 3.3 先执行 3.4 的命令，保证不报错，正常执行下去后，把输出结果删了，再回头从 3.3 走起。

所以，我的建议是，先执行一遍下面的命令：

```
tesseract cont.Arial.exp1.tif cont.Arial.exp1 box.train
tesseract cont.Arial.exp2.tif cont.Arial.exp2 box.train
tesseract cont.Arial.exp3.tif cont.Arial.exp3 box.train
tesseract cont.Arial.exp4.tif cont.Arial.exp4 box.train
```

确定执行没异样，生成了4个 .tr 文件，说明不太会踩坑了，删了 .tr 文件，正式开始下一步。

## 修改 .tif .box 文件对
把这4对文件，全部Copy回Win电脑，开始人肉标注工作。

启动`jTessBoxEditor.jar`,菜单栏第二排选择`Box Editor`,然后选择`Open`，打开`cont.Arial.exp1.tif`,老老实实按顺序开始吧，打开后Win下界面应该这个样子的：

![](http://upload-images.jianshu.io/upload_images/154674-73e6a91529358629.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Mac下界面是这样子：

![](http://upload-images.jianshu.io/upload_images/154674-aa59bb0fd69dcdb2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


三个红⭕️是主要用的地方，左边：
Box Coordinates——如果识别出来了文字，一行对应一个字符，就像图中一样。可以看中间集装箱号图片右下角的“1”，有一个红框，左边选中右边变红框，否则就是蓝色框。X、Y，表示框框的坐标，Width、Height，表示框框的宽和高，好理解吧。
Box Data——没啥用，就是显示你的 .box 文件的内容，随便用个编辑器看都比他看的爽。
Box View——有重大作用，后面说。

上面，左边：

四个按钮用途就是名字的意思：合并，拆分，插入，删除。操作的对象就是字符上的框框，选中一个框框，点点就明白了。

上面，右边：
Character——就是识别出来的字符，错了就改正确，正确就不管（左边的Char，单击也能修改）。
剩下四个就是调整框框的坐标和大小，主要就是：不但要识别准备，还要识别位置准确。

下面：
翻页，一张一张图的修改吧，差不多400张图片，每个图片最多15个字符，估计总共5000个字符是有的，基本上要以一个一个的过一遍，我差不多花了一天半才搞定。

现在知道Ubuntu 的报错为什么让我想砸电脑了吧，纯粹体力活。

![](http://upload-images.jianshu.io/upload_images/154674-d90b7f040ae7d8cd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图，要识别的字符都不会太大，肯定不太容易准确定位，切换到`Box View` 一切困难迎刃而解，图片太小看不清楚，就改下上面的3和4好了。

全部修改完了吧？好，回到Mac，距离训练成果不远了。

## 3.4 启动 Tesseract 训练
修改完成后，box文件和图片应该像这个样子了：

![](http://upload-images.jianshu.io/upload_images/154674-a7598366b3869528.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每一个图片中的字符都正确，并且识别框位置准确。

把 .box 和 .tif 文件Copy回Mac，正式开始训练，逐个使用下述命令，生成 .tr 文件。

```
tesseract cont.Arial.exp1.tif cont.Arial.exp1 box.train
tesseract cont.Arial.exp2.tif cont.Arial.exp2 box.train
tesseract cont.Arial.exp3.tif cont.Arial.exp3 box.train
tesseract cont.Arial.exp4.tif cont.Arial.exp4 box.train
```

执行后，会在同目录下生成 cont.Arial.exp1.tr ~ cont.Arial.exp4.tr 四个文件。

## 3.5 生成unicharset文件
到这里开始有些东西就开始难理解了，比如这个 `unicharset`文件到底是什么，官方文档说明是：

```
Tesseract’s unicharset file contains information on each symbol (unichar) the Tesseract OCR engine is trained to recognize.
Currently, generating the unicharset file is done in two steps using these commands: unicharset_extractor and set_unicharset_properties.
NOTE: The unicharset file must be regenerated whenever inttemp, normproto and pffmtable are generated (i.e. they must all be recreated when the box file is changed) as they have to be in sync.
```

建议还是看看，能多弄明白点不是也挺好么？不愿意看，那就直接执行命令。

#### unicharset_extractor

```
unicharset_extractor cont.Arial.exp1.box cont.Arial.exp2.box cont.Arial.exp3.box cont.Arial.exp4.box
````

秒返回结果，告诉你创建了一个 `unicharset`文件：

![](http://upload-images.jianshu.io/upload_images/154674-adf8c61aa92d922f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### set_unicharset_properties 

官方Wiki说的是还有这一步，但是不知道是不是一定要通过源码编译才有，反正我是没找到，这一步就忽略了。

```
training/set_unicharset_properties -U input_unicharset -O output_unicharset --script_dir=training/langdata
```

## 3.6 定义 font_properties 文件 
重要的事情再说一遍，具体含义，看官方Wiki，这里就跟着往下执行就好。

在当前的训练文件夹中，新建 `font_properties.txt` 文件，文件名不要自己发挥，必须这个名字，建议使用 **vi** 编辑，而不是用文档编辑器编辑，原因么，就是防止换行符的不一致问题咯。按照你的实际情况，在文档中输入下面一行内容，我输入的是：

`
Arial  0 0 0 0 0 
`

简单翻译下官方解释：
font_properties文件的目的是提供字体样式信息，当字体被识别时将显示在输出中。

在每行font_properties文件的格式如下：
fontname italic bold fixed serif fraktur

其中fontname（！不允许有空格）是一个字符串命名的字体，italic，bold，fixed，serif和fraktur都是简单0或1标志指示字体是否具有命名属性。

我觉得我训练的lang文件没有必要有什么特殊的，所以就全为 0 咯。

## 3.7 聚类：Clustering 
Clustering 官方提供了三个步骤，分别是：shapeclustering 、 mftraining 和 cntraining。

shapeclustering 的说明是 `should not normally be used except for the Indic languages` 所以，不是印度语，千万千万不要执行，否则你就会发现识别的全是错误的 。


```
mftraining -F font_properties -U unicharset -O cont.unicharset cont.Arial.exp1.tr cont.Arial.exp2.tr cont.Arial.exp3.tr cont.Arial.exp4.tr
```

```
cntraining cont.Arial.exp1.tr cont.Arial.exp2.tr cont.Arial.exp3.tr cont.Arial.exp4.tr
```

## 3.8 合并文件

到了这一步，行百里了。

现在需要做的是把训练过程创建的五个文件：shapetable，normproto，inttemp，pffmtable，unicharset，用lang.为前缀重命名（例如cont.），然后运行combine_tessdata：

`
combine_tessdata cont.
`

输出应该像这样的：

![](http://upload-images.jianshu.io/upload_images/154674-c2374913499320d2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，训练目录下多了一个 `cont.traineddata` 文件，这就是我们训练得到的lang文件了。

## 3.9 把 lang 文件放到lang 目录

可以先执行：

`
tesseract --list-langs
`

如果安装的时候选择了 `--with-all-languanges` 输出应该会是这样 `List of available languages (108):` 开头。

这里，Mac 和 Ubuntu 就不太一样了，由于是brew安装的，所以Mac的lang文件路径大概是：

```
/usr/local/Cellar/tesseract/HEAD-f5c18f7/share/tessdata/

实在找不到，就用

where tesseract 

先找到软链接的位置，再去找原身。
```

确定路径后，把 `cont.traineddata` 文件cp过去，再执行下 `tesseract --list-langs` 看看是不是多了一个，多了一个就对了。

## 3.10 验证

`
tesseract path-to-some.pic  stdout -l cont
`


# 4 一些问题处理的方法
