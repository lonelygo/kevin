# AI芯片软件栈（一）

> Some other hardware could be just as good in terms of raw processing, power model, memory, and kinds of architecture and so on. If they don't have a good software stack, they're simply not competitive. -- **Luis Ceze**: [Accelerating Machine Learning Systems](https://wandb.ai/wandb_fc/gradient-dissent/reports/Luis-Ceze-Accelerating-Machine-Learning-Systems--Vmlldzo4MDA3OTk?galleryTag=gradient-dissent)
> 
> **如果只有看起来很厉害的硬件，没有好的软件栈，将毫无竞争力！**

说到AI芯片的软件栈，这是一个非常宏大的主题，如果从驱动、固件到软件栈核心到MLOps支持，全链路的去考虑软件栈这个主题，涉及的知识广度已经超越了绝大多数人的技术储备，而其细节层面的复杂性几乎可以说超越了小团队/个人可以把控的复杂度上限，所以，我很清楚我显然不可能全部驾驭，只能不负责任的说到哪算哪；另外，一次性全部写完真不知道要写多久（摸鱼难，人也懒，内容更难），所以准备分成几个部分来慢慢完成。

为了不浪费大家宝贵时间，先说几个想法，如果觉得这些观点有一些共识，那么看看也无妨，如果觉得完全是胡扯，或者咱们三观不一致，八字不合，那就不用浪费时间了，做标题党博眼球的自媒体挺没意思的。

  - **软件栈**全链路的规划（软件产品路线图）与设计目标，不能是为了实现某些局部的极端技术理想，而是要为**商业目标**服务
  - 编译器是最难、最重要的，但必须意识到**编译器**仅仅是新硬件的**软件栈**重要的**一部分**，切不可一叶障目，顾此失彼
  - 对开源软件的~~白嫖~~改造，如果涉及到大量的**代码侵入**，最好考虑清楚是否有更优解，或者将来如何升级，磨刀不误砍柴工
  - 精心设计的架构与接口非常重要，才能比较轻松的实现软件的前后向**兼容**与未来的**扩展**迭代
  - 提供极致的**性能**与提供一套好用的**编程模型**与**稳定**的**运行时**同样重要
  - 某种意义上，**文档**与**参考代码**、**参考应用**就是公司的**窗口**与**形象**，值得投入人力与精力认真对待

先暂时说这几条，以我自己对自己的认识，估计写着写着可能会有新想法或者部分否定自己，这都是正常的，毕竟啊，善于总结和反思不是什么坏习惯，对吧💪💪💪？

这篇先说第一部分。

# 软件栈与营收

咋一看这好像是风牛马的两个事情，为什么要放到一起来说？所以先解释一下我的观点：

> AI芯片公司，做芯片（准确说现阶段主要卖的是板卡，下同）是为了卖出去，实现盈利，对吧。
>
> 没有软件的芯片，不过是这世界上昂贵的沙子，所以：
>
> 还要为芯片开发一套软件，让用户能按预期目标来使用芯片，才有可能下单，对吧。
>
> 事实上，流片回来，只要验证符合spec的设计目标，芯片能不能用，好不好用，有没有客户买单，基本就是软件说了算了。
>
> 所以，基本可以理解为：芯片量产是前提，**软件是营收的充分必要条件，且营收与软件完备度正相关**。

软件是芯片商业落地的保障，那么AI芯片公司，在规划**2亿营收**这个目标的时候，有没有考虑过下面这两个非常现实的问题：

> - **软件要做到什么样子，有哪些关键功能必须预判并提前实现，才能关键时刻不掉链子，支撑起来2个亿的营收目标**。
>
> - **软件与交付团队要做哪些人才、人力、方法论、标准化储备，才能保障2个亿营收的顺利交付**。

我猜，大概率没有，因为正如上一篇所说的，大多数技术创业者，对于如何做生意这件事，理解的真的太少了。

按下面逻辑来简单捋一下看看怎么规划软件栈看起来合理点，算是抛砖吧。

`营收预测` --> `软件刚需` --> `软件版本规划` --> `标准化、方法论准备` --> `售前、交付团队持续培训` --> `反馈、变更、螺旋上升`实现正向闭环。

## 营收预测

> 从上篇文章有过互动的朋友的背景来看，八成以上都是技术背景，所以大家可能对**营收预测**这件事，没太多的概念，先简单说一下，方便判断后续观点是否能站得住脚。

营收（销售）预测，绝对不是靠**拍脑袋**选出来一个能让自己开心的数字，就完事了的（但，目前的AI芯片公司，有几家不是这样拍出来的？）。道理很简单，`拍`出来的数字，几乎就没有任何可执行或者可跟踪的机会，只能是年初雄心壮志，形式一片大好，年终客观原因，明年会更好💪💪💪。

虽然`科学`这个词真的被太多太多太多的**滥用**了， 没有什么事情是不`科学`的，但是，从**销售管理**这个管理主题来说，营收（销售）预测，是真的有一些科学或者有效方法的，**销售漏斗**就是一个喜闻乐见的模型。

### 销售漏斗概念

首先，我们要对`销售线索`和`商机`区分清楚。

`销售线索`是什么，简单说就是“听说，某某公司明年准备采购500张AI加速卡……”，这信息大概率就真是一个**信息**，某种意义上，这种线索，除了能成为销售出差拜访客户的理由外，暂时对公司没太大意义。

`商机`是什么，简单说就是对`销售线索`进行了核实，明确了更多的细节，也基本搞清楚了对方的决策链，感觉还是有机会攻一下的。这些细节有哪些呢，大概是这些`故事`：1）、他们过去每年都有300～800片的T4采购量，所以明年这个500片看起来数字靠谱；2）、主要业务场景是跑目标检测、分割这些CV模型；3）、对国产卡不排斥，某某家已经在送测了；4）、对解码密度和推理精度比较看重；5）、对价格比较敏感，希望降低采购成本；6）、算法工程师C++水平好像一般，希望能提供Python的API测试；7）、主要用某某型号服务器，某某OS；8）、决策流程是……，某某和某某某已经对接了，其中某某是过去某关系的朋友。吧啦吧啦下来大家觉得看起来不错喔，立项吧，跟跟看。

第二，对于靠谱的`商机`肯定要跟下去的，那就进入了`意向`或者`施加影响`阶段（各行有各行的，各领域有各领域的，各公司有各公司的，专有名词这东西吧，很多时候就是让外行觉得**高大上**，其实就是那么个意思，造词这件事（比如互联网黑话、阿里味）有时候真的很没意思）。

对AI芯片这个行业来说，这个阶段其实是大家最喜闻乐见和干过不少的：**送测**。没太多好展开说的，既然`郎有意`，那就必须赶紧展示`妾有情`嘛，如果指定模型适配、精度、性能都通过初步验证，算是过了第一关，如果有好的售前和解决方案，针对客户场景和贵司芯片优势，再能讲出来一个让对方觉得`终于等到你`的故事，那这个阶段就更完美了，对吧👏👏👏。下一步，那就是想办法让对方明确采购`意向`，大多数情况下这是水到渠成的事情了（虽然实操中从来没有什么水到渠成，讲故事嘛，还是要搞点艺术渲染，哈哈哈哈）。

第三，折腾这么久终于进入实质性`决策`环节了，芯片行业一般叫这个阶段是`Design in`。

初筛通过，双方基本信任也建立了，毕竟大家都挺忙的，谁也没空在没意义的事情上浪费时间，可以实质性的全面测试、验证、提需求，方便对方得出决策依据了。这个阶段，其实也是让很多AI芯片公司感受到各种**花式Surprise**最多的环节，其实也是今天这篇的核心观点的主要出处。

让对方决策者能做决策，下决心买单，不是跑个Yolo，ResNet50就完事了，一个生产系统切换到一套新的软硬件，不能说要和原来的一模一样，只能说和原来的要几乎一摸一样，然后得到的**整体收益**要大于计入了切换的全部投入才会有足够动力。

第四，结果总是很简单`Win` ro `Loss`，`签单`或者`丢单`，无需多说。

好了，到这里，大家快速增加了一点新的知识储备，让我们继续前进。

### 无责的做个营收预测模型

稍等，我们先回忆下上一篇说的四个主要`AI芯片目标市场`：互联网/云计算，运营商，G和大B，以及ISV。虽然，上篇说了前三个市场要拿下没那么容易，时间也远没有**预计**中那么快，但是，梦想总是要有的，万一实现了呢。

所以，后续的分析的营收预测模型，在我自己的认知范围内是**相当不靠谱**的（当年非常难实现，如果做个3年的预测，靠谱度会高很多，但是目前投资环境不等人啊，就按一年来说吧。），但为了简化事情，也为了更符合从业者现在看到、听到的信息，还是就按这个不那么靠谱的模型来分析。

上面这句话的意思其实是：我觉得我已经把尺子放的非常松了，其实还不够松，大家如果准备对下面说的事情准备按图索骥的琢磨下自己公司，强烈建议**打五折**来看。

既然是模型，那一定会需要设定一些假定和依赖，以求降低模型复杂度，方便得出一些结论。

> **假定1**
>
> 下一年度的营收目标是：人民币 2 亿元。
>
> 我们先不去探讨是纯卖卡的收入还是连服务器等都算进来的`Revenue`。

> **假定2**
>
> 我们有很好的`关系`，在互联网/云计算，运营商及B和大B市场都有非常明确的市场机会。

> **假定3**
>
> 我们有足够规模的销售团队，上一年`储备的商机`已经足够支撑下一年度的~~故事~~预测。

> **假定4**
>
> 我们有足够的售前、FAE及售后，可以支撑我们需要的量级的工作量。

> **假定5**
>
> 我们软件团队有足够的工程师，来支撑需要的开发工作快速完成。
>
> 我们软件团队有出色的架构师、技术产品经理和技术决策者，保证我们每一步都能走对。

#### 收入分解

既然需要2个亿的收入，那么肯定只有把任务分解开来，才好做几件事：

  - 观测这个预测靠谱程度有多大
  - 分解细化到具体可执行/跟踪的计划（翻译成人话：每个销售背多少数字）
  - 判断我们的资源储备情况能不能跟得上
  - 判断哪些事情必须提前做好
  - 知道完成可能性有多大，来年还要再去发掘多少靠谱商机才有机会完成收入目标

下面基于上面的5个假定，以本人**臆想**的**看起来逻辑合理**的思路对2亿的目标收入做个极简的分解，目标就是能把意思表达清楚即可：

2亿营收拆解为：
  - ISV：3000万
  - 互联网：5000万
  - 运营商/G/大B：1.2亿

进一步不负责任的分解如下：

> - 毕竟国情在这，就不参考国外几家的收入数字了，以某上市芯片公司财报数据为参考，在芯片销售初期，每年能在ISV销售侧拿下三千万订单，真的不少了。单个ISV的年采购量能到三百万已经不算小公司了，考虑到刚接触试试看的心态更重，所以需要有：**15家**ISV的订单来产出3000万收入，是相对比较合理的预估；
>
> - 互联网/云计算虽然采购量不小，但是基于上一篇的讨论，全量给到新芯片的可能性个人真觉得不大，另外，互联网服务器是定点厂商，卖整机几乎不可能，只有单独卖卡的收入，所以，需要有：**3家**互联网的订单来产出5000万收入，是相对比较合理的预估；
>
> - 理论上，一个**大项目**签2个亿都不算太大的单子，但是这种好单子几率不高，前期工作量不小，咱稍微现实点，不能太乐观了。所以，就假设分包也好，联合体也罢，反正真的人脉爆棚，有**2个**此类订单来产出1.2亿收入（实际上，大单子能当年确认全部收入的情况非常罕见，或者说几乎不可能，依然为了简单化问题讨论，这个细节咱也忽略了，老司机别喷我不懂行就行）。

好了，营收都落到行业销售头上了，来数一下未来一年我们合理的**签单客户**数量：

> 15个ISV
>
> 3个互联网
>
> 2个大项目

#### 销售漏斗

还记得前面科普的销售漏斗的一点点知识点么，销售漏斗为啥叫这个名字，因为**形象**啊（赶紧一键三连，然后跳出去百度，看下直观的图片解释再回来😁😁😁）。从销售线索到最终成交，这个漏斗真能漏下去的真不多，确实不多，非常不多。

那么问题来了。

我们的2亿营收目标是漏斗出来的合同，那么我们按照漏斗的逆序往上考察一下这个问题：

**需要多少靠谱商机才能达成目标合同**。

这里又要继续开始为了简化问题，开始假设了。

> **假定6**
>
> 本来公开市场竞争就非常惨烈，做点生意不容易，现在大家芯片都回来了，竞争更激烈了，所以对ISV行业的销售签单率，我们定义为：`33%（即：1/3）`。老销售都知道这个比例是多高了。
>
> **假定7**
>
> 互联网和大项目，**运气**（自己琢磨什么是运气吧）对了怎么都有，否则怎么都挺难，但是本着国产替代和供应链安全这两个主题的客观存在，这两个行业的销售签单率，我们定义为：`50%（即：1/2）`。

基于上述两个假设，和前面的**签单客户**数量，那么我们应该在手的商机是多少呢？

> **ISV**：9000万商机/45个潜在客户
>
> **互联网**：1亿商机/6个潜在客户
>
> **大项目**：2.4亿商机/4个潜在大项目

事情到这里看起来已经有点儿严峻了，来笑一个，别那么悲观么，人定胜天。笑完了么，继续往下看，事情还没完呢。

前面说了销售线索和商机不是一码事，只有接触了，初步沟通了，也判断有**拿下**的机会的销售线索才能叫商机，所以，让我来继续假设。

> **假定8**
>
> 由于AI芯片这个行业的特殊性，目标行业比较清晰，所以我们有理由相信，销售线索到商机的转化率还是比较高的，所以假定：
>
> ISV：`66%`，也就是`3`个线索能变出来`2`个商机（好可怕的效率）
>
> 其余：`75%`，也就接触到`4`个项目机会，`3`个能看起来像有生意。

所以，终极数据可以出来了，要完成2亿的营收，实际上我们目前在手的销售线索和潜在客户数量应该是：

> **ISV**：1.3亿线索和商机/70个左右潜在客户
>
> **互联网**：1.3亿线索和商机/8个潜在客户
>
> **大项目**：3.2亿线索和商机/5个潜在大项目

当然，这些数字不是说要我们在年底全部在手了才能做预测。如果，年底我们真储备了这么多机会，那么来年的目标达成率肯定高很多。当然，还有一年呢，我们完全可以有理由**鼓励**自己，虽然现在手里机会不多，但是来年我们找到这么多机会。现实点来说，一半一半吧，储备占一半，来年再找一半，可能才是比较理智和现实的。

所以到这个时候，其实喊一个：下一年营收预期 5.8 亿也行，也没什么不合适的，对吧。

那么问题来了，废这么大篇幅说了一堆基于各种假设的废话，想表达什么？让我们进入下一个主题。

## 交付压力

基于上述对于营收的不负责任的预测，我们得到了几个关键的信息：

> 来年，我们要：
> 
> FAE与AE能支持超过`80`个客户的产品测试与业务代码迁移遇到的各种奇奇怪怪的问题。
>
> 而且其中`70`个ISV的下游客户不可能高度集中，一般都是背靠背的签合同（那种真兄弟，帮着压货的，不是这个话题讨论的问题）。
>
> 一个ISV可能就冒出来`10`个不同的项目，不可忽视其中存在的模型及软件特定feature需求。简单点，一般ISV摊子也不会铺的特别大，不会各行业生意都能做，就当有`3`个不同的行业case支持需求。
>
> 现在**真麻烦**来了，其实我们心中要做好来年有**200+**的客户支持与交付需求。
>
> 软件团队能支持并且是高效支持好上述客户的看起来奇奇怪怪的需求。
>
> 售前团队能支持上述客户的售前活动。

可能会有人质疑`70`和`200`这两个数字是不是刻意夸大，传递恐慌。其实，按我前面说的要打`五折`的说法，把这个数字在心里翻译成`150`和`400`都一点儿不过分，质疑的原因大概是没有在纯公开的市场里一个单子一个单子去死磕过项目。据说，某公司明年的销售团队规模要扩到一个非常恐怖的数字，其实心里挺慌的，人家软件栈成熟了，芯片也经过了大量的市场验证，这么大规模的去扫市场，我们剩下的公司去攻ISV的市场，会真的越来越难了，呜呜呜，给点活路好不好。

书归正传。

当然这些工作量压力，是不可能**平均分配**到12个自然月的。而且，一般的理解上，1、2两个月春节前后放羊、收心；11、12两个月年终总结、下年预算，这四个月对销售来说，落单的效率是非常不高的，那么一年就剩下了**8个月**是产出率高的时间段，考虑到销售即使技巧再高，逼单手艺再丝滑，都是要时间的，流程总要一步一步走的，而且合同签了到交付、验收回款还是要有时间余量的。

技术同学也要理解理解看起来就是吃吃喝喝的销售同学的苦逼和艰难，什么活都不好干，那碗饭都不好吃。给销售兄弟也留点时间和腾挪空间，别算时间的时候只考虑自己研发周期（更别提测试和回归时间都不预留的`勇敢`开发计划了）。

好了，大家互相理解，有一个团结的团队，真好。但是2个亿的目标还要完成，本来说好是一年的事情，可能就变成这样了：

  - 八月底前，起码要把80%的合同签订前期技术工作做完，给销售签单、交付、年底前回款留够时间。
  - 十月底前，尽量把阻碍验收的问题全部解决了，给销售回款留够时间。

再考虑到1、2两个月基本没太多事情，也就是说从售前到交付的绝大多数任务要在6到8个月内完成，2亿营收才稳一点。

就问你，慌不慌，慌不慌，慌不慌。

这就是说，峰值人力需求，起码是平均值的**三倍**，才能扛住营收压力带来的峰值工作强度。

由于每个客户的支持不是说一天两天就结束的，有可能持续几个月，人的绝对数量需求，这事情不是互联网流量能玩“削峰填谷”，也没有“弹性扩容”这种好事，硬碰硬的事情，从来没有凭运气的，只有靠实力与才华。

但，这个实力绝对不是：**提前招一堆人**。

其实，就算公司有钱养人，能做好AI芯片的FAE、售前，我的理解是：算法、推理软件栈、视频编解码、PEIe设备、服务器、操作系统、网络、虚拟化、容器、K8S、数据中心以及目标客户的最终客户的业务场景这些奇奇怪怪的技术与业务知识，懂一半是需要的吧，以目前AI芯片圈子的总体人才数量，就算不怕烧钱，也招不到那么多啊。

那么这个问题怎么破？

简单说就是这个问题没法回答。各公司情况、阶段、人力储备、管理风格、协作程度以及大技术体系有几个分管VP、哪边话语权高、山头多不多，这些情况都不一样，求一个通用解是不可能的。

当然，可以试着去从几个思路下手试试看，比如：

> - AE 做为 FAE 的二线，在关键时刻顶到一线不是不可以嘛（当然，人要机灵点，别出门瞎说大实话）。
>
> - FAE 天然就具有售前属性，在关键时刻顶一下售前的缺，出去念念PPT，忽悠忽悠也是可以的嘛（当然，不能太老实，一看做不了就say no怎么玩）。
>
> - QA 对自己的软件栈非常熟悉，也有错误排查经验，潜伏在客户支持群里，帮着FAE和AE盯一下，解决点问题不是不可以嘛（当然，不能是个问题都往回来揽，有些活就是客户自己的）。
>
> - 解决方案、售前也不是都只会画PPT，也有技术背景不错的，这时候不上，什么时候上。
>
> - 关键大客户的支持，级别够的领导亲自跟一下么，了解一线火力，有助于做决策的时候判断准确。
>
> - 人还不够用，可以看看研发的兄弟们，有没有思路清楚，口才过关，和陌生人打交道也不怵的，可以顶一顶纯技术的对接任务。

创业公司，专业化很重要，通才也很重要。没有一批人做什么像什么，放到哪里都能发光的，能很好应对突发的人力需求峰值的多面手，真的很难搞成大事。否则就是招很多冗员，一年忙2个月，其余10个月悄悄摸鱼还好，就怕无事生非，非要给自己加戏，这种公司几乎没有内部不乱的。

上面的这人员复用策略，也**不是银弹**，只要有点管理经验和人员调动权限，都能想到，但是能不能做好，是另外一件事。

前面说了，时间紧，任务重，压力大，这时候其实比较忌讳把人动来动去的，比较容易让具体干活的兄弟产生迷失状态：我是谁，我在哪，我该干什么，我要干什么。

但，当这种救火式/运动式人员调动成为有且仅有的解决方案的时候，当领导的，以公司利益为出发点，工作还是要做细一点，要对手下的兵的真实能力有非常好的判断和管理直觉。

第一，提前沟通好，让大家早早完成心理建设阶段。

第二，对调动和轮转安排，有个预判和安排，让大家提前能结对，知道发生什么事情，自己什么角色，什么任务，和谁配合，找谁支持。事到临头慌慌张张抓壮丁，不乱，不掉链子只能说公司命硬，还能撑到掉链子。

第三，要有领导力，说清楚锅来了，不是干活的背，否则临时顶缺，本来是帮公司解决问题，结果给自己解决回来一口大黑锅，谁不怕，不推事才怪。

第四，提前培训，甚至，面试的时候就要琢磨下这些岗的候选人，还能帮什么岗顶一顶。当然，这里有个前提，面试官自己就要是个全才，还能真站在公司一盘棋角度考虑问题，而不是山头主义。

第五，文档和知识管理一定提前做好。刚开始接触客户，各种已知问题，奇怪异常的解决方案等等，如果仅仅作为经验沉淀在特定人员脑袋中，每个坑都要所有人都踩一遍，这事情是非常可怕的。

第六，做好软件规划和版本管理，送出门的软件版本越少，支持工作量越少，如果软件玩出来N多个版本，基本上就离那个啥那个啥了。

好了，终于说到软件了。

我怀疑你们都是来看各种段子的，但是我没有证据，但这并不妨碍我给你们时不时讲个段子。

某家软件栈早期出现过这样的`奇景`：模型A可以在AA版本的SDK跑起来，模型B可以在BB版本的SDK跑起来，模型C可以在CC版本的SDK跑起来，但是你要把这个组合关系换一下，对不起，不是软件有问题，是你不会用。

还是某家，因为有这个模型和SDK版本强耦合的问题，在更早一点的时候，据说，据说啊，FAE带着服务器去给客户演示，到了现场三天，版本各种对不齐，还没有把自己家软件跑起来。

脑补一下，这种软件栈状态，这是靠加几个人，就能从本质上解决交付难度，交付效率和交付质量的么？

交付跟不上，要么就是永远在**Design in**，怎么都走不到**Design win**，不善于反思的团队，甚至design in了十几家，没有一家走到落单，但是就不选择先停下来反思反思到底哪里做错了，总是不能`Win`一把。要么就是好不容易来试单小合同，但是怎么都验收不了，导致用户放弃了后续的真正下单的大合同。

> 公司的愚蠢在于一次次用同样的方式去做同样的事，但确期待会有不同的结果。
>
> Page：1， Edward Yourdon 《Death March》（2nd Edition），中译本：《死亡之旅》

但，有时候软件的锅真不应该全是软件团队来背，甚至软件团队的码农不但不该背锅，还是受害者。

那么，问题来了：谁该背锅呢？

## 软件栈规划

知彼知己，百战不殆。

凡事预则立，不预则废。

我要说的其实就这么多，散会。

这真不是开玩笑，下面就算怼出来一万字，核心思想其实还是上面两句，我是坚信大道至简的。但是已经憋出来七千字了，就这么虎头蛇尾，怎么也不是我的风格，多少也要啰嗦几句。

其实，AI芯片软件栈的规划这件事本身，真没什么好说的。`NVIDIA`的[CUDA-X](https://www.nvidia.cn/technologies/cuda-x/)已经把`AI+HPC`的软件栈整体拼图交代的清清楚楚了，要是觉得NVIDIA这套太庞大，一时半会儿是~~抄~~学不明白的，也没关系，打开友商的官网，基本都有软件栈的架构图，画的细节可能有点不一样，但是整个架构逻辑，不能说一摸一样，只能说大家思路都出奇一致。

当然要一致了，一方面NVIDIA这么多年不是吃干饭的啊，以CUDA为生态核心的那套东西，就是这个领域的事实标准了，不跟着人家节奏走，自创一套，去教育用户，改变开发者对AI软件栈的理解，要这样想，太猛了吧？另一方面，往简单说，抛开各家硬件设计引入的细节差异，AI芯片的软件栈就那么点东西，画架构图，真画不出来花。

但是，这里要说的软件栈规划，并不是指上述那种高度抽象的框架图，那些官网展示的“软件栈架构”对于实际开发工作的指导意义非常有限，对于干活的工程师来说，我们眼中的架构图在某些细节层面甚至需要展开到概要设计级别，才能做到知道到底要做什么，要做成什么样，以及如何做。

### 知彼知己，百战不殆

毕竟我们做的是AI芯片，和AI芯片软件栈，这一套软硬件是为AI计算加速服务的。软件研发安排如何服务好我们的商业目标，这是一个问题，一个很大的问题。

发现一个很有趣的事情，没有一家不对以编译器为核心的工具链不高度重视的。重视到什么程度呢？不排除有人以为不管是手写算子，还是手工优化，只要编译器能把可执行二进制吐出来，模型能跑起来，软件就算完善了，软件就具备交付条件了。

之前一直在强调，芯片公司创业者大多是**技术背景**，所以对做生意这件事，可能理解有点儿偏差，到了这里，我们必须要对`技术`再做一次细化。更准确的说技术背景应该是**芯片设计技术背景**。

多了这四个字的定语，我想表达的意思就呼之欲出了。

你猜的没错，我们也没有必要回避这个问题。

对软件不熟悉，对软件工程不熟悉，对AI软件不熟悉，对用户的开发使用需求不熟悉，对算法模型的生产部署需求不熟悉，对用户的基础设施对软件的需求不熟悉，林林总总加在一起，导致的各种让纯软件背景的看了哭笑不得的事情层出不穷。

不举例了，不讲段子了，太伤人。大家回顾下自己公司的软件故事，品味下就好，别人家故事也没啥好说的，反正家家有本难念的经。

前面费了大量的篇幅，得出来一个数据，再放一遍：

> 现在**真麻烦**来了，其实我们心中要做好来年有**200+**的客户支持与交付需求。

如果，我们对销售预期有信心，那么就进入执行层面的问题了：怎么能确保计划实现？

一句话回答就是：判断这些潜在客户从接触到签采购合同到最终交付，我们都要提供哪些服务和产品（软件）交付。

只有比较清晰的知道了我们必须要做什么，要交付什么，大概是什么节奏，那么研发才有机会和销售去**对齐**，哪些我们已经ready了，哪些预计什么计划，让销售和售前有足够的心理预期，把握下接触不同类型客户的节奏。

一方面，我们要对软件栈需要做什么，做到什么程度才能快速实现客户交付，有判断。

比如下面这些都是有可能的**需求**：

  - 我们想做大项目，那么数据中心的系统集成以及需要的相关软件，我们准备拿开源自己搭，还是找合作伙伴，算法模型呢，哪里来？
  - 只要进数据中心，基本就离不开`K8S`、`Kubeflow` 、`MLflow`这样的生态，那么相关的`plugin`这些必备的东西，有没有招到写`go`的程序员，有没有开发计划。
  - ISV客户的数量非常多，而一般情况下ISV客户开发人员有限，能写好CPP的也不多，都会希望提供全套的软件，那么要不要提前规划`Python API`还有`推理引擎`等方便快速部署的软件栈。
  - 现代主流的应用软件架构都是以`服务化`部署来实现解耦的，那么遇到需要`服务化部署`需求，要求提供一套类似`Triton Inference Server`的软件，或者就要求你支持做为`Triton Backend`使用，或者要用`KServe`、`Seldon Core`、`BentoML`这些，要不要提前做好开发安排和对接技术预研。
  - 对于互联网客户，几乎家家都有自己的推理引擎和服务化部署方案，是先提供一套能满足他们业务需求的软件，快速完成集成验证、灰度这些，还是等人家基础设施工程团队完成新硬件的后端开发完成，这是个很大的问题。
  - 不管什么行业的客户，换了一套新的软硬件，学习和迁移成本是人家必须要担忧的，文档全不全，参考代码质量怎么样，人家业务场景有没有完整的参考代码，这些也要考虑好
  - 除了参考代码、文档和基本工具链（编译器），开发过程中少不了要用的`GDB`，`Profiler`，量化丢精度了有没有精度对比的工具，这些工具都要有。
  - 除了SMI，运行期的监控工具有没有，能提供哪些监控项，除了`C/CPP API`，能不能直接提供`Prometheus`风格的`endpoints`服务，方便用户在现有监控系统快速集成。
  - 峰值算力、PCIe带宽、DDR带宽，编解码路数等`Peak performance test`套件及压力测试工具，不论是早期客户验证还是服务器厂商适配都是需要的。

另一方面，我们要对模型适配节奏有很好的规划。

当然，不用说大家也会准备，那就是常见模型的适配与性能优化，精度对齐工作。但是，这里面有个问题就是，早期，大家手工优化痕迹都非常重，靠编译器实现模型泛化只能存在售前的PPT中，实际什么能力，我们关起门来心理要有数。

所以，我们优化的模型的选择，其实是非常重要的，如果前期销售摸回来的信息靠谱，那么模型支持的节奏就能很好的和销售的找客户节奏对起来，否则就会陷入：

> 来一个客户 --> 来两个新模型 --> 加班适配，性能优化 --> 送测 --> 没下文 --> 再来一个客户，再来两个新模型

的恶性循环中，研发拼命干活，销售总是不成单；销售找个客户回来，等模型就是三个月，客户丢了。

理智一点，知道现阶段的真实能力，瞄准目标客户的目标模型，拿出来7成人力按照梳理好的模型列表加速优化，否则，东一榔头西一棒槌，很难有大收益。

第三，如果我们有人能（这个人其实挺难找）帮着梳理出来软件和模型的需求，并能设定看起来大概合理而不是“拍脑袋”的计划，那就太好了。

你预判了你的客户的需求，多美妙。
### 凡事预则立，不预则废

预判这件事，说起来就是大道理，没人不懂。从观察和听到的故事来看，有两个难度，导致了这个非常浅显的道理几乎实践的都有很大偏差。

难度一：信任。

其实大家都知道，在AI芯片创业公司中，出来人最多的那家美国芯片公司，软件有什么级别的人，之前在做什么。

但是，这些是跟了十几年的自己人，信任无价啊。

外来的，不管你啥背景，先不说能做多大事，能给予到多大的信任。其实，能融入到什么程度，都是一个悲伤的话题。

做事的基础没有，把事情做好真挺难的。

难度二：文化。

不知道各位公司里面“War Room”还多不多，不知各位公司的月度、季度、年度“Award”标准是怎么衡量的。但是，我是听到不少类似“丧事喜办”、“坑先放着，放大了，有人关注了再填”这样魔幻的故事。

一句话，当公司没法对真实产出和产出价值做出正确判断对时候，那么只能用“看谁在关键时刻靠的住”这个最简单的主观方案去做判断。但是也许永远不会知道，也不去琢磨：为什么总有那么多事情需要**War Room**才能解决，平时都在干嘛？

轰轰烈烈的讲故事；有点进展就敲锣打鼓；故事讲不下去了，赶紧撤换下一个故事。这样的同事，贵司多么？

当，有这种文化和工作模式的存在土壤，有人**不幸**具有极强的规划和预判能力，做的所有事情都平静的听不到声音，因为都预计到了，那么自然很难出现什么突发情况需要“War Room”表演一番。

工作干的平静如水，毫无波澜，那么，有多少人觉得这才是真高手？这才是应该Award的种子选手？

当然，还有一个不可回避的难度问题，各家遇到的情况不太一样，所以，不能放在第三。

正如开头提到的考虑到AI芯片的完整软件栈的链条长度，很难有一个帅才全部能把控的很好。如果，公司人脉一般，吸引不到软件大牛加盟，那么大概率大软件团队会比散，各干各的，谁也说不服不了谁，等到全部做出来，集成的时候傻眼了，这种情况不是不存在。

> 只有对自己，对行业，对用户，对收入目标有了清晰、理性的分析和判断，软件能提前做的事情才有机会提前做对、做好。
>
> 软件一步到位就能做好，这显然是幻觉。​但是**基于营收目标的软件规划**要比先闭门做软件，等到**用户需求**进来在匆忙应对，来的靠谱的多，而且软件质量更容易保证。
>
> 软件首先要确保销售**敲门**环节没问题，同步做好**签单**和**交付**必备软件的开发计划，节奏感很重要。
>
> 如果可以，请做好软件栈的总体设计和概要设计之后再动手，最坏的情况，也要先把接口设计好。

这就是我理解的：知己知彼，预则立。

好了，今天这一篇先到这里。

下一篇接着说软件，试着聊聊软件栈总体规划。

欢迎加我的微信“**doubtthings**”，欢迎交流与探讨。

欢迎关注我的公众号“**书不可尽信**”，原创文章第一时间推送。

<center>
    <img src="https://s2.loli.net/2022/11/27/WAC1ml5X8GTvOuH.jpg" style="width: 100px;">
</center>