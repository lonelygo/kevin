---
layout: post
title: Rime输入法—Squirrel词库添加及配置
subtitle: Rime输入法—Mac版本-Squirrel（鼠须管）词库添加及配置
tags: [教程，Mac]
comments: true
---

# 为什么用Rime

13年底的时候，日本爆出百度的日本版本输入法的问题，要求政府人员停用，没当回事，反正我没用，当然了，有关搜狗和用户隐私有关的问题就一直没有中断过，也没太在意。但，前几天McAfee爆出的某输入法用户敏感数据未加密传输的问题，就让人担心了。

好吧，既然这样，还是卸了第三方的输入法吧，虽然Yosemite自带的输入法已经进步很大了，但是总是用的不顺手，也想念自己多年在搜狗输入法上积累的词库。既然这样，那就只能考虑自己动手丰衣足食的问题了。

有关“Rime鼠须管”输入法，在各类MAC相关的论坛上都能看到“神级输入法”这样标题的推荐，必须承认仅仅就速度这个角度来说，确是非常优秀，输入过程非常流畅毫无卡钝，当然做为开源软件，作者的思路应该是：字库无需非常庞大，字库是靠用户用自己的输入习惯来“养成”的。

# Rime是什么

有关Rime的介绍及基本的安装介绍，不多说，请自行搜索了解，下面借“眀无梦”的一个Gif图：

![](http://ww2.sinaimg.cn/large/55fc1144gw1eoomtd8jl9g20hp0auwjd.gif)

**原文在此**：[安装及配置 Mac 上的 Rime 输入法——鼠鬚管 (Squirrel)](http://www.dreamxu.com/install-config-squirrel/)

之前对Rime仅仅是当作一个新鲜玩意试着玩玩，没太在意有关配置及词库的问题，这次准备下定决定长期使用，想按照自己的需要进行一些调整，检索资料，发现相关内容分布较散乱，有些内容写的过于简单或者晦涩理解起来还是需要花点时间。

本着兼听则明的原则，在理解了官方的指南以及一些优质的资料（如上面gif的作者）的基础上开始了定制过程，我的目标很简单，主要有三个:  

**调整外观**  
**优化输入体验**  
**增加词库**

***BTW***

1. 我用的是鼠须管，别的OS下的版本还未尝试，但从Rime的系统架构来看，以下的这些东西应该是对非OSX平台也有借鉴的。  
2. 鼠须管在OSX下的**“用户文件路径”**如下，下文中修改/增加的文件都在这个目录下完成。（当然，也可以右键鼠须管的图标，选择“用户设定...”直达目录。）  
    > ~/Library/Rime/
3. Rime的配置文件都要求是“encoding: utf-8”，所以建议准备一个好用的文本文件工具，比如强悍的：Sublime Text等，后面有关字库的处理，最好能适当懂一点终端工具的使用。
4. 在准备自己动手之前，请自行学习官方说明：[**《Rime 定製指南 | rimeime》**](https://code.google.com/p/rimeime/wiki/CustomizationGuide)，在下面的文字中，会对官方资料做一些摘录和引用，在此一并说明。
5. 一定要注意配置文件的缩进控制，所有的修改都必须“重新部署”才能生效。

# 配置方法

目标明确，资料消化结束，下面正式开始介绍鼠须管的配置过程：

## 1. 外观配置

对于外观调整，主要想解决以下几个问题：a、不习惯竖排，要改为横排；b、默认自体太大，要改小一点；c、修改字体颜色和背景颜色。

### 调整候选词数量：

在“用户文件”路径：
> ~/Library/Rime/  

新建文件：default.custom.yaml, 内容如下：  

```yaml
    patch:
        menu/page_size: 9  #设置候选字数量，根据喜好调整。  
```

**注：如果之前修改过`default.custom.yaml`文件，那么请一定按照官方的要求去做：所有 *.custom.yaml文件中只能有一个“path：”！**

### 调整候选栏样式

在“用户文件”路径：
>~/Library/Rime/  

新建文件：squirrel.custom.yaml, 内容如下：

```yaml
    # squirrel.custom.yaml
    patch: 
      # us_keyboard_layout: true		 # 键盘选项：应用美式键盘布局
      # show_notifications_when: growl_is_running  # 狀態通知，默認裝有Growl時顯示，也可設爲全開（always）全關（never）
        style/color_scheme: demo	     # 选择配色方案
        style/horizontal: true           # 候选窗横向显示
      # style/inline_preedit: false   # 关闭内嵌编码，这样就可以显示首行的拼音（MAC下不建议开启）
        style/corner_radius: 3           # 窗口圆角半径
        style/border_height: 4           # 窗口边界高度，大于圆角半径才有效果
        tyle/border_width: 4             # 窗口边界宽度，大于圆角半径才有效果
      # style/line_spacing: 1         # 候选词的行间距
      # style/spacing: 5              # 在非内嵌编码模式下，预编辑和候选词之间的间距
        style/font_face: "Lantinghei TC Extralight"  # 预选栏文字字体，使用中文字体：兰亭黑-纤黑
        style/font_point: 17             #预选栏文字字号
        style/label_font_face: "Myriad Pro Light"  # 预选栏编号字体，使用西文字体：Myriad Pro Light
        style/label_font_point: 17       #预选栏编号字号
         #上述是候选栏的基本设置，确定了文字的大小和候选栏的外观样式。
         #下面是“demo”样式文件的配置，主要确定候选栏颜色配置。
        preset_color_schemes:
        demo:		#样式名称，就是上述“style/color_scheme: demo”
        author: "***** <****@gmail.com>"        #作者
        name: "无语／*****"                      #作者名字
        label_color: 0xf2a45a	                #预选栏编号颜色
        back_color: 0x333333		            #背景颜色
        candidate_text_color: 0xb9b9b9          #非第一后选项文字颜色
        comment_text_color: 0xa5a5a5            #注解文字颜色
        hilited_candidate_back_color: 0x333333    #第一后选项背景颜色
        hilited_candidate_text_color: 0xff7d00    #第一后选项文字颜色
        hilited_comment_text_color: 0x00a5ea      #注解文字高亮
        hilited_text_color: 0x7fffff              #拼音串高亮（需要开启内嵌编码）
        text_color: 0xa5a5a5                      #拼音串颜色（需要开启内嵌编码）
```

已经完成基本的外观调整：横排候选栏，9个候选字，候选栏的：字体、背景色、文字颜色等都完成了调整。  
**注:** 在调整过程中发现Rime可能存在一个小Bug(或者是还没有理解透这个文件)：“label_color: 0xf2a45a”配置的颜色并不是实际显示出来的颜色，需要找个配色工具，将你选择的颜色确定后，选择“色轮”对面颜色的“Hex Value”填上去既可显示你选择的目标颜色。

## 2. 优化输入体验

输入体验的调整主要是想解决以下内容：a、按自己的习惯增加“方案选单”的呼出快捷键；b、屏蔽自己不需要的方案；c、增加混和输入的方式，实现上面gif动画中的“中英文+emoji”输入方案等。

### 调整快捷键：

修改：`default.custom.yaml`, 设定“输入选单”中激活的输入方式，内容如下：

```yaml
    patch:
         menu/page_size: 9      #这是之前增加的候选词数量，可以看见“patch：”只能有一个的意思了。
         schema_list:           #“输入选单”中激活的输入方案定义。
         #   - schema: terra_pinyin
             - schema: luna_pinyin
         #   - schema: emoji
             - schema: luna_pinyin_fluency
         #   - schema: double_pinyin_mspy
             - schema: luna_pinyin_simp
         #   - schema: bopomofo
         #   - schema: double_pinyin_flypy
```

根据自己的输入习惯进行选择，我只保留了“朙月拼音”、“朙月拼音・语句流”、“朙月拼音・简化字”三个方案，其余的都屏蔽了。

由于Rime默认“输入选单”激活的快捷键有三个，其中：F4在MAC下是没用的，“control+\` ”的快捷键在Sublime Text冲突，所以建议增加一个快捷键并优化中西文切换的配置。继续修改`default.custom.yaml`，内容如下：

```yaml
    patch:
        menu/page_size: 9  #这是之前增加的候选词数量。
        schema_list:           #“输入选单”中激活的输入方案定义。
         #  - schema: terra_pinyin
            - schema: luna_pinyin
         #  - schema: emoji
            - schema: luna_pinyin_fluency
         #  - schema: double_pinyin_mspy
            - schema: luna_pinyin_simp
         #  - schema: bopomofo
         #  - schema: double_pinyin_flypy

    #下面定义中英文切换的方式
        ascii_composer/good_old_caps_lock:  true
        ascii_composer/switch_key:
        Caps_Lock: noop
        Control_L: commit_text
        Control_R: commit_text
        Eisu_toggle: clear
        Shift_L: inline_ascii
        Shift_R: inline_ascii

    #下面定义“输入选单”的切换控制
        switcher:
            abbreviate_options: true
            caption: "〔切换〕"          #把默认的“方案選單”修改为了“切换”。
            fold_options: true
            hotkeys:
                - "Control+grave"       #默认方案
                - "Control+Shift+grave"   #默认方案
                - "Control+s"             #新增方案
            option_list_separator: "／"   #以下都为默认custom.yaml文件的默认配置，copy过来就可以。
            save_options:
                - full_shape
                - ascii_punct
                - simplification
                - extended_charset

```

#### 上述配置参数解释如下：

中西文切换键的默认设置写在default.yaml里面，default.custom.yaml 可以在全局范围重可以定义该组快速键。

可用的按键有Caps_Lock, Shift_L, Shift_R, Control_L, control_R，而Mac 系统上的鼠须管不能区分左、右，因此只有对Shift_L, Control_L 的设定起作用。已输入编码时按切换键，可以进一步设定输入法中西文切换的形式。

#### 可选的临时切换策略有三种：

>inline_ascii ：在输入法的临时西文编辑区内输入字母、数字、符号、空格等，回车上屏后自动复位到中文。
commit_text ：已输入的*候选文字*上屏并切换至西文输入模式。
commit_code ：已输入的*编码字符*上屏并切换至西文输入模式。
noop ：屏蔽该切换键。

所以，我的配置的意思是：Caps lock键保持系统默认配置；Shift键临时切换为英文输入，回车确认后继续保持中文输入法；Control键：已经输入的汉字上屏，并切换为英文输入法。

### 增加emoji表情输入

**注:** 由于Rime输入法不同的输入方案有不同的配置文件，以下对输入方案的配置文件的修改都以“朙月拼音”为例进行，其余的输入方案修改基本类似。

在“~/Library/Rime/”下新建：“luna_pinyin.custom.yaml”就是朙月拼音对应的个性化定义文件。
在这里多说一句，为什么不直接修改而是增加一个“custom”文件呢？这个方法的好处是，这些custom的配置文件是可以放在网盘备份的，这个备份可以有三个好处：

1. 个人的数据在自己信得过的网盘中，不会有乱七八糟的事情初现；
2. 可以让多个客户端都指向备份文件夹，这样就可以做到多端同步；
3. 版本升级，不会影响个人数据与个性化配置，一劳永逸。

在`luna_pinyin.custom.yaml`中新增如下代码：

```yaml
    patch:
        engine/translators:
            - punct_translator
            - r10n_translator
            - reverse_lookup_translator
       recognizer/patterns/reverse_lookup: "`[a-z]*$ " #请删除$后的空格！不加一个空格貌似总是解析错误，不知道MarkDown我还有多少不知道的地方:(
        schema/dependencies:
            - emoji
        abc_segmentor/extra_tags:
            - reverse_lookup
        reverse_lookup:
            dictionary: emoji
            enable_completion: false
            prefix: "`"
            tips: 〔表情〕
```

保存，重新部署，切换到“朙月拼音”输入“`”然后输入“biaoqing”看看是不是出来了，如果不生效，请检查代码的对齐是不是有问题。有关emoji表情的介绍看[**符号表**](https://rimeime.googlecode.com/svn/trunk/artworks/schema/emoji-chart.png)（自备梯子，没梯子的同学看最后，我会把图片放在后面）。

到这里，是不是感觉鼠须管有点意思了，我前面说了，目标是实现“中英文混输+emoji”，现在看起来实现了一半，下面开始解决中英文混输的问题。这个问题要解决，就要开始增加词库了。

## 3. 增加词库

开源软件的好处是：总会有很多热心的同学做好一些现成的东西供大家享用，我在这里主要是做开源资料的搬运工，下面引用的资料所有权及解释权还是人家创作者的，请大家怀着感恩的心合理使用（可能需要梯子）。

为什么是“增加词库”而不是导入词库呢？详细的解释看这里：[**〔新手推荐敎程〕关于导入词库及「深蓝词库转换」的正确操作方法 | rime 吧**](http://tieba.baidu.com/p/2757690418)。建议3楼到15楼能耐心看一遍，对理解Rime解构大有帮助，总结下来就是：Rime的词库设计是递归引用和调用的，如果盲目的使用Rime默认提供的词库导入工具把词库全部导入到默认的码表里，很有可能造成Rime性能下降，行云流水般打字的感觉就木有了，木有了！
如果没耐心看，那就先看我的处理过程，然后会头再去看官方的说明。

### 增加可用的词库

原材料在这里：[**朙月拼音擴充詞庫**](https://bintray.com/rime-aca/dictionaries/luna_pinyin.dict)，下载，可以顺便看看网页上的Readme，然后解压缩，开工。
摘录Readme部分内容如下：

>解壓縮得到六个文件。
>如果是**「朙月拼音」系列輸入方案**的用戶，請將補靪文件 luna_pinyin.custom.yaml 改名爲你所使用的輸入方案對應的 id。（比如朙月拼音·簡化字方案，則將 luna_pinyin.custom.yaml 改名爲 luna_pinyin_simp.custom.yaml）
>如果是**雙拼輸入方案**的用戶，請將補靪文件 double_pinyin.custom.yaml 改名爲你所使用的輸入方案對應的 id。（比如智能ABC雙拼方案，則將 double_pinyin.custom.yaml 改名爲 double_pinyin_abc.custom.yaml）
>將六个文件放入用戶文件夾中（*Windows*：%AppData%\Rime，*Mac*：~/Library/Rime，*Linux*：~/.config/ibus/rime/）。
>重新部署（*Windows* 用戶請在開始菜單中找到〔小狼毫輸入法〕，然後點選「重新部署小狼毫」；*Mac/Linux* 用戶請在右上角的輸入法選單中點選「重新部署/ ⟲ (Deploy) 」）。
>驗證：切換到拼音或其他適用方案，輸入「*一介書生*」（驗證擴充詞庫之基本詞庫）、「*一丈紅*」（驗證擴充詞庫之漢語大詞典詞彙）、「*疑是地上霜*」（驗證擴充詞庫之詩詞詞庫）、輸入「*哆啦A夢（duo la a meng）*」（驗證擴充詞庫之西文詞庫，此子詞庫爲朙月拼音系列方案專有，雙拼方案不推薦使用）。

#### 1. 修改luna_pinyin.custom.yaml

**这里一定要注意的是：**因为之前我们已经调整了“luna_pinyin.custom.yaml”文件，所以在copy的过程中一定不能把这个文件覆盖过去，否则，前面你就白忙乎了。

正确的作法是，打开解压缩后的luna_pinyin.custom.yaml文件，将其“patch:”下的代码copy到我们自己的`~/Library/Rime/luna_pinyin.custom.yaml`中，copy完成后，自己的`luna_pinyin.custom.yaml`文件看起来应该是下面这个样子的：

```yaml
    patch:
        engine/translators:
            - punct_translator
            - r10n_translator
            - reverse_lookup_translator
        recognizer/patterns/reverse_lookup: "`[a-z]*$ "   #请删除$后的空格！不加一个空格貌似总是解析错误，不知道MarkDown我还有多少不知道的地方:(
        schema/dependencies:
            - emoji
        abc_segmentor/extra_tags:
            - reverse_lookup
        reverse_lookup:
            dictionary: emoji
            enable_completion: false
            prefix: "`"
            tips: 〔表情〕
    # 載入朙月拼音擴充詞庫
        "translator/dictionary": luna_pinyin.extended
    # 改寫拼寫運算，使得含西文的詞彙（位於 luna_pinyin.cn_en.dict.yaml 中）不影響簡拼功能（注意，此功能只適用於朙月拼音系列方案，不適用於各類雙拼方案）
    # 本條補靪只在「小狼毫 0.9.30」、「鼠鬚管 0.9.25 」、「Rime-1.2」及更高的版本中起作用。
        "speller/algebra/@before 0": xform/^([b-df-np-z])$/$1_/
```

#### 2.Copy文件

将：  
`luna_pinyin.extended.dict.yaml`  
`luna_pinyin.hanyu.dict.yaml`  
`luna_pinyin.poetry.dict.yaml`  
`luna_pinyin.cn_en.dict.yaml`  
这四个词库文件移动到用户资料夹 `~/Library/Rime/`

如果你是双拼用户，请按照Readme的说明将`una_pinyin.poetry.dict.yaml`词库文件修改名称后，移动到用户资料夹 `~/Library/Rime/`

#### 3. 重新部署

由于涉及到词库文件的处理，你会发现本次部署的时间要稍为长一点，在等待的过程，看看上面的Readme中的“验证”。

按照验证中的几个词输入一下试试看，再输入“upan”、“ip”看看，是不是直接输出了U盘和iPad？很惊喜有木有？中英文混输已经可以了。

如果验证不成功，请确认你的“输入方案”选择是“朙月拼音”，如果还有问题，请会头仔细检查过程中有没有出错。

### 增加自己的词库

相信很多人和我一样以前用也是搜狗输入法，所以以下内容都是如何将搜狗的词库转移到Rime中进行使用。在[**〔新手推荐敎程〕关于导入词库及「深蓝词库转换」的正确操作方法 | rime 吧**](http://tieba.baidu.com/p/2757690418)中的9、10、13楼提到另一个开源软件OpenCC，用途就一个：繁简转化。按照贴吧中的说法需要将词库进行繁简转化后再考虑导入系统的问题。所以，我先考虑的是，将搜狗词库转化为方案文件，仿照[**朙月拼音擴充詞庫**](https://bintray.com/rime-aca/dictionaries/luna_pinyin.dict)的处理方式先扩容词库，而不是直接导入到Rime词库中。

#### 1. 词库准备

##### 1.1 下载一个可用的Rime词库

[**这里**](http://pan.baidu.com/share/link?shareid=141610&uk=2902507575&third=0)有一个现成的搜狗词库转为Rime词库的可用文件，感谢愿意分享的同学们（如果链接失效了请自行寻找，或者联系我索取我处理过的词库）。

**这个词库命名为：** `sogou-1.txt`

##### 1.2 导出自己的搜狗词库

不幸的事情是搜狗MAC版本好像没有词库导出的选项，Win下面7.4. * 以后的版本貌似导出的词库是加密的了，我所检索到的资料还没有找到能处理的办法（一个输入法，都要给用户制造选择障碍，体高用户迁移成本，唉，谁让人是免费的呢，也没啥好计较的，想起来了“艰难的决定”）。
当然，问题总是可以解决的：Win下7.1以内的版本是支持将词库导出到TXT文件的，依靠度娘找一个旧版本，赶紧起虚机导出词库文件把（现在不用，以后说不定也用得着）。建议导出的时候格式选择txt，免得后面的处理出现意外。

##### 1.3 词库转换

神器出场：[**深蓝词库转换**](https://code.google.com/p/imewlconverter/downloads/detail?name=imewlconverter_2_0.zip&can=2&q=),如果链接打不开，请百度并安装2.0版本（ZIP包：959KB）。这个工具使用很简单，绿色版本，界面简单，操作友好，只要确定输入是搜狗词库，输出是Rime文件格式就可以。

**转换完成后这个词库命名为：** `sogou-2.txt`

#### 2. 词库简繁转换

##### 2.1 OpenCC安装

有关OpenCC的项目介绍可以看[**这里**](https://github.com/BYVoid/OpenCC)可以看到Rime项目的大神“佛振”也参与了这个项目。如果你是码农，你完全可以从GitHub拉下来这个项目之后安装，如果你是MAC的试用者，对开发不熟悉，那么这个OpenCC的安装还是需要花点时间的。
安装方式看[**这里**](https://code.google.com/p/opencc/wiki/Install)，没有pkg也没有dmg，只有源代码和命令行。如果你不是码农，那么很有可能需要从：

**Homebrew套件管理器开始安装**，[Homebrew](http://brew.sh/index_zh-cn.html)是OSX下的一个好东西，你可以不用去了解他，打开“终端”（默认在“实用工具”里面），输入下面的命令：

```bash
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

中间会有提示，请确认，然后等待安装结束，具体安装时间就看你的网络情况了，安装结束后开始：

**开始安装OpenCC**，不要关闭“终端”，请输入下面的命令：

`brew install opencc`  

具体安装进度看你的网络情况了，等待安装完成。
安装完成后应该可以看到OpenCC的版本是1.0.2。在“终端”输入：

`opencc -h`  

可以看到opencc的命令行帮助信息，这时候就说明opencc已经部署好了，下面开始词库转换。

##### 2.2 OpenCC简繁转换

首先用Sublime Text分别打开sogou-1.txt、sogou-2.txt确定编码格式是否为utf-8。
如果没问题，分别将两个文件另存为：sogou-1.1.txt、sogou-2.1.txt，全选后删除所有数据保存退出。这样我们就有了编码格式正确的sogou-1、sogou-2这2个源文件和sogou-1.1、sogou-2.1这2个目标空文件。

为了方便后面的操作，在`~/Library/Rime/`目录中新建一个文件夹，并命名为`sogou`；然后将上一步准备的四个文件Copy到`~/Library/Rime/sogou`文件夹中。

在“终端”，请输入下面的命令：

```bash
    opencc -i ~/Library/Rime/sogou/sogou-1.txt -o ~/Library/Rime/sogou/sogou-1.1.txt
```

很快就会执行完成，然后在终端中继续转换第二个文件：

```bash
    opencc -i ~/Library/Rime/sogou/sogou-2.txt -o ~/Library/Rime/sogou/sogou-2.1.txt
```

回到“~/Library/Rime/sogou”文件夹中，可以看到sogou1.1txt、sogou2.1txt已经有在了转换后的繁体字的词库，至此词库转换完成。

#### 3. 制做词库文件

将`~/Library/Rime/sogou`中的`sogou-1.1txt`、`sogou-2.1txt`文件copy到`~/Library/Rime/`文件夹中，然后将之前临时新建的sogou文件夹删除或者是剪切到适当的位置保存。

将`sogou-1.1txt`重命名为：`luna_pinyin.sogou.dict.yaml`。

如果不知道怎么修改扩展名，可以复制任一`*.yaml`文件，并将其名字修改后，将`sogou-1.1.txt`中的全部内容copy进去，然后删除没用的`sogou-1.1.tx`t即可。

打开：`luna_pinyin.sogou.dict.yaml`，在文件 **_最上方_** 增加如下内容：

```yaml
        ---
        name: luna_pinyin.sogou    #这就是你自定义的词库的名字:sogou，后面还要用到
        version: "2015.XX.XX"	    #版本时间，最好填当前时间，要版本控制的意识
        sort: by_weight
        use_preset_vocabulary: true
        ...
    #下面就是之前转换好的词库，如：

        釣魚島	diao yu dao	1
        黑瞎子島	hei xia zi dao	1
        南沙羣島	nan sha qun dao	1
        鴻庥島	hong xiu dao	1
        南威島	nan wei dao	1
        景宏島	jing hong dao	1
```

将`sogou-2.1txt`重命名为：`luna_pinyin.yourname.dict.yaml`，同样的，在文件前面增加：

```yaml
        ---
        name: luna_pinyin.yourname    #这就是你自定义的词库的名字:yourname，后面还要用到
        version: "2015.XX.XX"	    #版本时间，最好填当前时间，要版本控制的意识
        sort: by_weight
        use_preset_vocabulary: true
        ...
```

#### 4. 使词库生效

##### 4.1 修改yaml文件

回顾一下，在前面增加词的过程中，对`luna_pinyin.custom.yaml`path了这样一条：

```yaml
        patch:
    # 略去其余部分
        "translator/dictionary": luna_pinyin.extended
```

可以看到，对于“朙月拼音”，现在调用的是`luna_pinyin.extended`，打开`luna_pinyin.extended.dict.yaml`，可以找到下面这段：

```yaml
        ---
        name: luna_pinyin.extended
        version: "2014.09.07"
        sort: by_weight
        use_preset_vocabulary: true
    #此處爲明月拼音擴充詞庫（基本）默認鏈接載入的詞庫，有朙月拼音官方詞庫、明月拼音擴充詞庫（漢語大詞典）、明月拼音擴充詞庫（詩詞）、明月拼音擴充詞庫（含西文的詞彙）。如果不需要加載某个詞庫請將其用「#」註釋掉。
    #雙拼不支持 luna_pinyin.cn_en 詞庫，請用戶手動禁用。
        import_tables:
            - luna_pinyin
            - luna_pinyin.hanyu
            - luna_pinyin.poetry
            - luna_pinyin.cn_en
        ...
```

现在明白了吧：对于“朙月拼音”，我们在全局的`luna_pinyin.custom.yaml`中定义`dictionary`的是`luna_pinyin.extended`，在`luna_pinyin.extended.dict.yaml`中定义`import_tables`是：`luna_pinyin`、`luna_pinyin.hanyu`、`luna_pinyin.poetry`、`luna_pinyin.cn_en`，所以只需要做如下修改，增加`tables`即可：

```yaml
        ---
        name: luna_pinyin.extended
        version: "2014.09.07"
        sort: by_weight
        use_preset_vocabulary: true
    #此處爲明月拼音擴充詞庫（基本）默認鏈接載入的詞庫，有朙月拼音官方詞庫、明月拼音擴充詞庫（漢語大詞典）、明月拼音擴充詞庫（詩詞）、明月拼音擴充詞庫（含西文的詞彙）。如果不需要加載某个詞庫請將其用「#」註釋掉。
    #雙拼不支持 luna_pinyin.cn_en 詞庫，請用戶手動禁用。
        import_tables:
            - luna_pinyin
            - luna_pinyin.hanyu
            - luna_pinyin.poetry
            - luna_pinyin.cn_en
            - luna_pinyin.sogou
            - luna_pinyin.yourname
        ...	
```

##### 4.2 重新部署

如果操作正确，请用你个人词库你认为属于生僻的词组做测试吧，比如你一个奇怪的同事的名字，你们公司奇怪的产品名字等，惊喜不？鸡冻不？

## 5. 同步词库

到了这里，你已经对鼠须管的配置文件比较熟悉了，所以，同步词库很简单。编辑`installation.yaml`文件，添加一行：

`sync_dir: “/Users/username/Dropbox/sync/Rime”`

如果你是细节控，还可以修改：

```yaml
installation_id: "yourname"  #自定义个人文件夹的名字
```

怎么样，现在是不是感觉到Rime真的是一个不错的输入法了？

如果你属于专业人士，对于搜狗的细胞词库依赖度很高，没关系，按照前面的方法：下载——深蓝转换——OpenCC转换-形成词库——修改yaml文件的顺序去处理就OK了。

如果在过程中还有什么搞不定的，请留言。  
最后附上官方提供的emoji表情的介绍看[**符号表**](https://rimeime.googlecode.com/svn/trunk/artworks/schema/emoji-chart.png)
![](http://ww1.sinaimg.cn/large/55fc1144gw1eop5z1urebj21fo37g7pa.jpg)。
