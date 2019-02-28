---
layout: post
title: Beautiful-Jekyll Theme 增加中文字体
subtitle: 为github.io的Jekyll静态网页blog主题增加中文字体支持。
bigimg: 
  -  "/img/big-imgs/hangzhou.jpeg" : "杭州，西湖，2016"
tags: [教程, Github]
comments: true
---

&ensp;&ensp;&ensp;&ensp;之前是用的 Hexo 的方案搭建的 Github.io，Mac 升级到 10.14 之后，莫名其妙的 Safari 钥匙串自动填充功能就挂了，登录框得焦点再也不会自动提示你可用的账户名称了，咬着牙么也能用，心病难除，于是就抹盘重装了，然后你懂的 Hexo 没有备份☹️。好在我是一个极其懒惰的人，几年也没写过多少文字，直接`clone`下来rm -rf了重新提交个空白回去也不心疼。  

# 所以需要有个模版主题

&ensp;&ensp;&ensp;&ensp;人不能掉入同样的坑两次，所以这次老老实实用 Jekyll 方案了，简单好用。简单好用的另一面就是真“简单”，找来找去，找到现在用的这个[beautiful-jekyll](https://github.com/daattali/beautiful-jekyll)主题看起来还不错。

## 居然正文用的是Serif字体，不能忍啊！

&ensp;&ensp;&ensp;&ensp;乔老爷子及其厨子接班人，成功的连续不断用 Retina 屏和 PingFang 字体降低着我们眼睛的耐受力，Serif 字体真的不太适应了，就想着自己改吧。直接 Google 尽然没有任何地方说怎么改这个主题的中文字体的。这事情就麻烦了，只能自力更生了，前端的世界基本不懂，但是起码知道 Html 和 CSS 文件是干嘛的么，大概文件结构什么情况，该去哪里找东西直这些基本概念还是有的么，咱也不算什么都不知道。

## 直接说结果

&ensp;&ensp;&ensp;&ensp;这套生态都是用的GoogleFonts，当然这个模版也也是这样，GoogleFonts上简体中文的 Sans Serif 字体有且仅有一个： Noto Sans SC，没得挑没得选，费力去改到加本地中文字库，那就还要有各种浏览器适配到问题了，没这能力，想都不想。折腾好了就赶紧记录下来，免得忘了，也方便有同样需求的“爱美人士”取用，免得再浪费不必要的时间。过程就不说了，直接上结果。  

&ensp;&ensp;&ensp;&ensp;作者的字体方案是：  

&ensp;&ensp;&ensp;&ensp; - 标题，第一顺位字体：**Open Sans**  

&ensp;&ensp;&ensp;&ensp; - 正文，第一顺位字体：**Lora**  

&ensp;&ensp;&ensp;&ensp; - 等宽字体，第一顺位字体默认使用 Bootstrap 选择的： **Menlo**  

基于个人喜好，我加了下几个字体：  

- **Noto Sans SC**
- **Roboto**
- **Roboto Mono**

&ensp;&ensp;&ensp;&ensp;Noto Sans SC 第一顺位，Roboto 第二顺位，Roboto Mono 是等宽字体，用于代码块和注释部分。

### 要动的文件

&ensp;&ensp;&ensp;&ensp;只需要动三个文件，分别如下，其中`blog`是你的博客根目录，相信你是知道的。  

``` bash
blog/_layouts/base.html
blog/css/main.css
blig/css/bootstrap.min.css

```

&ensp;&ensp;&ensp;&ensp;`blog/_layouts/base.html`修改部如下，原来的两个默认字体我没删，换成了GoogleFonts文档推荐的写法。

``` html
---

common-googlefonts:
  - "Lora:400,400i,700,700i"
  - "Noto+Sans+SC:300,400,500,700&amp;subset=chinese-simplified"  
  - "Open+Sans:300,300i,400,400i,600,600i,700,700i,800,800i"  
  - "Roboto:100,100i,300,300i,400,400i,500,500i,700,700i,900,900i"
  - "Roboto+Mono:100,100i,300,300i,400,400i,500,500i,700,700i"  

---
```

&ensp;&ensp;&ensp;&ensp;`blog/css/main.css`文件改的地方比较多，但是也就是个替换的事情，比如：

``` css
body {
  font-family: 'Lora', 'Times New Roman', serif;
  ......
}
```

&ensp;&ensp;&ensp;&ensp;改成：

``` css
body {
  font-family: 'Noto Sans SC', 'Roboto', sans-serif;
  ......
}
```

&ensp;&ensp;&ensp;&ensp;再比如：

``` css
h1,h2,h3,h4,h5,h6 {
  font-family: 'Open Sans', 'Helvetica Neue', Helvetica, Arial, sans-serif;
  ......
}
```

&ensp;&ensp;&ensp;&ensp;改成：

``` css
h1,h2,h3,h4,h5,h6 {
  font-family: 'Noto Sans SC', 'Roboto', 'Open Sans', 'Helvetica Neue', Helvetica, Arial, sans-serif;
  ......
}
```

&ensp;&ensp;&ensp;&ensp;考虑到Github还是能正常访问的，后面的默认字体还是不要删除为好，剩下的都和上面这一样，查找替换就好。等宽字体其实 Menlo 也挺好看的，不改也挺好，要改的话，如下操作：  

&ensp;&ensp;&ensp;&ensp;打开`blig/css/bootstrap.min.css`查找`Menlo,Monaco,Consolas,"Courier New",monospace`，定位后，在`Menlo`前面增加 **`"Roboto Mono", `**千万别忘了后面还要有个`,` 。

## BTW

&ensp;&ensp;&ensp;&ensp;如果要在首页上使用全宽的图片，可以在 `blog/index.html` 中按如下格式增加内容：

``` vhtml
---
layout: page
title: your title
subtitle: Write something...
bigimg:  
  - "/path-to-imgs/image-namge1.jpeg" : "image description1"
  - "/path-to-imgs/image-namge2.jpeg" : "image description2"
  - "/path-to-imgs/image-namge3.jpeg" : "image description3"
use-site-title: true
---

```

&ensp;&ensp;&ensp;&ensp;然后就可以实现通栏图片的轮循了，同样的，在blog页面头部同样写法也可实现单独页面的图片变化。  

EOF