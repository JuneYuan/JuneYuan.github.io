---
title: "ARTS (1125-1201)"
date: 2019-12-01T14:56:26+08:00
draft: false
---

![what-is-arts](https://static001.geekbang.org/resource/image/d5/47/d5ef2a0f9077fc90bb11230f63638647.jpg)

# Algorithm

题目来源于 The Go Programming Language, Ch6.5 Example: Bit Vector Type 和后边的 Exercise. 

`Bit Vector` 简单说就是一个基于位操作、实现高效集合运算的数据结构：

> A bit hector uses a lice of unsigned integer values or "words", each bit of which represents a possible element of the set. The set contains `i` if the `i`-th bit is set.  

通常的集合数据结构，如果元素是整数，就用一个整型变量来表示一个元素： `int x`.  `Bit Vector` 则是用一个整型变量的每一位，表示一个元素。对于 `int64` 类型，`N` 个变量就可以表示取值在 [0, N*64) 范围内的任意元素。这样，根据集合中元素值的大小计算好 `N`, 就能表示整数集合了。基于位操作实现集合运算，不用遍历，效率较高。

```
type IntSet struct {
	words []uint64
}

func (*IntSet) Has(x int) bool
func (*IntSet) Len() int
func (*IntSet) Add(x int)
func (*IntSet) AddAll(... int)
func (*IntSet) Remove(x int)
func (*IntSet) Clear()
func (*IntSet) Copy() *IntSet
func (*IntSet) UnionWith(t *IntSet)
func (*IntSet) IntersectWith(t *IntSet)
func (*IntSet) DifferenceWith(t *IntSet)
func (*IntSet) SymmectricDifference(t *IntSet)
```

# Review

[The programming talent myth](https://lwn.net/Articles/641779/)

> Hi, I'm Jacob, and I'm a mediocre programmer.  
> I am, at best, an average programmer  

这两句话出自 Kaplan-Moss 在 2015 年 PyCon 论坛上的演讲。虽然是 python 的论坛，但这场演讲的主题却并不局限于 python 语言，而是道出了很多程序员都有过的自我怀疑，并由此延伸到更多思考。 Kaplan-Moss 的演讲想必影响了很多人，国内就有一篇[《一个平庸程序员的自白》](https://blog.csdn.net/shengxiaweizhi/article/details/46383743)与之呼应，被很多网站转载。

我最初是在 2016 年七八月份准备校招时，碰到过一篇文章介绍 Kaplan-Moss 的演讲内容，浮光掠影地读了一遍也就过去了。这次为 task 作准备，很奇妙地又想起来，凭着一点模糊的记忆搜索了好久，反复尝试各种关键词，才终于找到。 Kaplan-Moss 的演讲最“击中”我的有两点：一是他不避讳地公开承认自己是 "mediocre programmer", 二是他站在行业发展的角度看待 "programming talent myth" 造成的不利影响，视角高了一层。下边分别来说。

大多数程序员肯定都经历过一些特别绝望的时刻，怀疑自己能否做得好编程这件事。尤其是想到大牛们如何精通最新技术而又掌握底层原理，懂得基础理论又能高效解决实际问题，反观自己，只能以寻常的水平做着普通的工作，偶尔要是出去面试，大概就会被算法、技术原理、源码实现等问题虐一遍。可能许多次都觉得自己作为一个程序员实在是功夫平平，然后叹息一声，再硬着头皮走下去。这声叹息是不便说出来的，既碍于面子羞于承认，也怕被人斥为找借口、不努力。所以 Kaplan-Moss 这么说，我很钦佩他的勇气。

另外一点，我想先从“鄙视链”说起。很多事情上都能折射出那么个无形的链条，程序员当中尤其典型。大体就是使用有些低级编程语言的程序员鄙视用高级语言的，从事不同技术领域的程序员之间可能相互鄙视，还有科班出身的鄙视从别的行业半路出家做程序员的，等等。坦白说，最后这种情况我也有，倒不完全是鄙视，更多是一种别人来和自己抢饭碗的感觉，毕竟从业者变多就意味着竞争更激烈、个人议价空间变小，看起来总是不大妙的，由此心态出发，鄙视鄙视别人也未尝不可。可 Kaplan-Moss 看到的却是行业的人才缺口，是业内存在的性别歧视、种族歧视、"im a suck" 心态等等束缚了个人发展的问题。他希望大家认识到 “programming talent myth” 对行业内外的人皆有伤害，而摒弃对 programmer 高到离谱的定义标准，摒弃那种非 rock 即 suck 的观念，则有助于部分问题的解决。

其实，无论是过分崇拜大牛，还是妄自菲薄，又或是转而鄙视他人，都无助于我们在编程这条路上走得更远。 "Most people are average at most things" 的规律虽然打破不了，但我们仍然可以通过适当努力取得合理的进步，有尊严地去工作。个人发展与行业发展相关，从后者出发看问题，有时会发现不同的角度。

- - - -
BTW, 由于是演讲，所以语言本身也很精彩，忍不住做了一些摘录——

> + There are things I have done that I'm incredibly proud of.   
> ...   
> There are things that I have done that do make me feel qualified to be on the stage.  
（这些自我肯定的话语让人更相信他说自己是 "mediocre programmer" 时是真诚的，而非故作谦虚）

> + When humans don't have any data, they make up stories, but those stories are simplistic and stereotyped. So, we say that people "suck at programming" or that they "rock at programming", without leaving any room for those in between. Everyone is either an amazing programmer or "a worthless use of a seat".  

> + Most people are average at most things.  

> + It is actively harmful in that is keeping people form learning programming, driving people out of programming, and it is preventing "most of the growth and the improvement we'd like to see".  

> + If the only options are to be amazing or terrible, it leads people to believe they must be passionate about their career, that they must think about programming every waking moment of their life. If they take their eye off the ball even for a minute, they will slide right from amazing to terrible again.  

> + That's because "programming is something you are in this myth, not something you do."  

> + Like any other skill, you can program professionally, occasionally, or as a hobby, as a part-time job or a full-time job. You can program badly, program well, or, most likely, be an average programmer.  

> + The tech industry is rife with sexism, racism, homophobia, and discrimination. It is a multi-faceted problem, and there isn't a single cause, but the talent myth is part of the problem.  

> + It is going to take a serious effort to overcome the diversity problem. We are never going to get there if we can't come up with a more nuanced way to think about what a programmer is -- and what programming skills really are.  

- - - -

# Tip - Custom JSON Marshalling/ Unmarshalling in Go

Golang 默认的 Marshal/ Unmarshal 方式有时不符合期望，需要自行调整个别字段的处理。

[Reference](http://choly.ca/post/go-json-marshalling/)

[code](https://github.com/JuneYuan/gobyexample/blob/master/examples/json/custom.go)

# Share

## 为什么做 ARTS

最主要的两个原因——

一是工作能力需要提高。拿自己跟身边同事作比较，工作效率和解决问题的水平都有一定差距。这种差距虽然部分与工作年限相关，但更多还在于积累和练习。每天花最多时间相处的就是工作，当然自我价值高低、自我肯定/否定也较多会受到工作影响，到头来，提高工作能力就是在提高自信力、幸福力。这个“提高”，当然是相对于过去的自己，而非去跟别人比。退一步说，如果老是停留在当前水平，我就得担心自己哪天找不到工作了，可我还不想这么早离开本行。

二是希望自己能够无愧于计算机系学生这个背景。这么说是因为，我常觉得自己是注了水的计算机系毕业生，对很多专业知识的掌握只有应付考试的水平， 为此心中有一种惭愧与羞耻。所以要么得付出实际行动去挤掉身上的水分，要么就得长期伴着这种羞耻感生活。

针对上述两个问题，我也做过零散的、不成系统的努力，实际效果进进退退，未能令自己满意。看到 ARTS 这个建议，感觉像一味复合制剂，配比科学，效果令人期待。

就这样，我决定开始了。

## 单独说一下 "S"

陈皓前辈关于 "A", "R", "T" 的说明我都基本认同，最后一个 "S", 稍有点不同想法。

"Share" 我会去做，目的却不是（至少目前不是）为了“建立影响力”、“输出价值观”。 "Share" 这件事包含两部分，一是思考，二是写出来。思考的目的不用多说，所以问题就落在了“写出来”的目的。促使我决定做 ARTS 的理由并不包括“建立影响”和“输出价值”，只是为了解决自己的问题和困境。具体到 Share, 写，我的理由也很简单，那就是，有时候产生了某种想法，它憋在肚子里，鼓鼓荡荡的，或者是什么念头在脑子里飘晃却不得其所，就好像从树上脱落的孤叶似的无处停靠。偶尔能拿出纸笔，把想法写下来，就会感到憋着的东西得到了释放，同时书写过程帮助自己理清了一些新旧想法的关联，于是飘晃着的东西也找到了相应的位置。这种体验非常好，只可惜我乱想的时候远多于记录的时候，表达能力由于缺乏锻炼也只能差强人意。 Share 作为一种约束，对解决这个问题是有益的。

除了自己“写”的理由，我还想略谈几句当下的“写作现象”。如今“写作能力”已经被渲染成最重要的能力之一，说句玩笑话，人们像修炼武功一样地磨练自己的写作能力。自媒体、公众号写手们大多都有一个 10w+、100w+ 的“小目标”，日益壮大的微商从业者每天要想着朋友圈怎么发才能吸引更多人下单，手机上一众 App 背后是更多的运营团队在琢磨活动文案怎么设计才能更有效地刺激转化率，还有许许多多知名不知名的博客作者在笔耕不辍……大家有的注重招式，有的在意内功，有的自行修习，有的拜师学艺，推动着琳琅满目的写作课市场也迎来了春天。同样是写文章，学生时代同学们练习作文的主动性跟今天参加付费写作课程的热情，实在不可同日而语。

这种“写作现象”是好是坏，我难以评价。只不过作为阅读者，我不太喜欢公众号上的很多所谓文章，而且有越来越不喜欢的趋势。个人还约略觉得，微信公众号内容质量整体来看要不如上一个时代的新浪博客，太多公众号文几乎就是为了点击量而存在的，它必须极尽所能达到吸睛和煽动的目的，相比之下，新浪博客时代有更多作者在专注于表达自己。当然，我并不想把意思推向一个极端，公众号中也有十分优秀的作者和文章，有对人们有帮助的内容。筛选出这些内容，并且屏蔽掉其他东西的干扰，就像披沙拣金一样，需要付出一些代价，至于值不值得，全看个人选择了。另一方面，对于文章背后的写作者，还是回到 Review 部分 "Most people are average at most things" 的规律，我觉得如果以变现为目标，那么 80% 靠 10w+ 赚钱的机会仍然只属于 20% 的人，并且竞争在持续变得激烈。撇开这个目标来看，作为普通人写写东西，既不损及他人，又令自己有所收获，同时还能把被微信微博抖音快手等“毒品 App”侵占去的时间拿回来一些，那还是不错的。
