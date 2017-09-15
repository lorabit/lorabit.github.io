---
layout:     	post
title:      	"12345"
subtitle:   	"\"Life of a software engineering intern at Google\""
date:       	2017-09-15 00:00:00
author:     	"LoraBiT"
header-img: 	"img/post/bg-google-intern.jpg"
header-mask: 	0.3
catalog: 		true
tags:
    - 硅谷
    - 找工作
    - 实习
    - Google
---

实习开始前老纠结什么时候可以把Google加到我的LinkedIn资料中，好不容易熬到实习开始，却又发现没什么东西可以写的。现在回头看看，要把在Google做的事情跟外面的人讲清楚，确实不是一件容易的事情。

## Onboarding 

实习Orientation的地方在Sunnyvale的Tech Cornner Campus，大概就是简单地介绍一下Google的文化（Google的使命、Google的第一条狗的名字等等），然后一人发了一件体恤、一个双肩包、一台电脑、一个电脑包、一本资料，就差不多结束了。为了能够尽早开始干活，我第一天就跑去Mountain View见了我的Host - 一个浙大学长。

然而我还是想多了，第一周都被安排了各种培训。什么Google的代码管理、本地化、下一百万用户(NBU)、信息安全、Accessibility、Code of Conduct等等。所以第一周也就没干什么事情。但可以确认的是，之前Team Match的时候说要做的project也没法做了，主要因为实习生不允许访问用户数据。

## Java@Google 101

可以说来Google之前因为各种原因抵制了十多年的Java，除了上课需要，唯一一次用Java还是13年去UCSB交流的时候为了方便调用各种NLP库被迫使用的Java。其实主要还是Java的开发环境实在太难配置了。

Java@Google 101是Google内部培训系统Grow上面的一门课的名字，课时大概一整天，我报了名但能去上课的时候感觉已经学得差不多了，也就没去了。这门课之所以叫Java@Google 101而不是Java 101是因为在Google写Java跟在Google外的其他地方写Java完全不一样，从依赖注入框架、代码管理、编译到部署，Google都有自己的一套体系，以至于光这些不一样的东西就得花一天来入门。

首先Google的内部代码管理用的是自己开发的Piper，它跟其他的版本控制系统最大的区别在于它只有一个代码仓库 - google3，Google的所有代码都存储在google3上。如果用git的概念来解释，可以认为google3是一个唯一的一个仓库里的唯一一个branch，任何改动都直接提交到这个branch上。几乎每秒钟都会有一个新的改动（changelist, CL for short）被提交到google3上。这样做的好处在于每个人的工作目录下都有全部的最新的Google代码，任何问题可以更及时的被发现，在各种工作流程中也不用再去指定目标代码在哪个仓库下的哪个分支下，而直接用一个google3的相对路径即可。坏处也显而易见，经常前一天晚上还运行地好好的代码，第二天早上就没法运行了，花了一个上午调试后发现是有一个粗心大意的工程师改动了你的某个依赖。也正因为这样，Google的代码review机制也跟其他地方完全不一样。

当工程师创建出一个改动（CL）之后，需要获得所有改动目录的一位Owner LGTM(looks good to me)和Approve，和改动部分所使用全部语言的一位拥有readability的工程师的LGTM和Approve，代码才可被merge到google3。比如我创建了一个CL，只使用了Java语言，然后改动了A、B两个目录下的文件，那么这份CL需要一位拥有Java语言readability的工程师、一位A目录的Owner和一位B目录的Owner的同意才能通过审核。

Google内部使用的编译工具也跟外面的完全不一样，不管你用什么语言编写的项目都得使用blaze编译。用blaze编译所需要的参数一般由工程目录下的BUILD文件提供，blaze一般会在远程执行编译命令。为了方便Google员工创业，Google还开源了一款无google3的blaze版本 - [bazel](https://bazel.build/)，其本地编译速度远超其他同类工具。

[Guice](https://github.com/google/guice)是Google自己开发的开源依赖注入框架，在其项目主页上也毫不掩饰其对Spring Framework的各种借鉴，但在Google内部的实践中，所有用Guice的项目都使用Java代码动态创建的绑定，而不是用XML文件指定的。绝大部分情况下Guice能直接利用类名和标记直接完成绑定。

在Guice的基础上，Google还开发了Apps Framework用来开发网站和Web service，基于它的ProducerModule可以十分方便地开发异步执行的服务，所有Google内部的身份验证/授权库都被集成在了Apps Framework中，可惜的是这个框架暂时还没有开源版本。

我的实习项目是用Apps Framework+Google内部微服务平台Boq搭建一个新的service用来帮助迁移数据。往简单的说就是搭一个Web service提供一个读取API和一个写入API。如果自己做一个这样的service大概一两个小时就可以搞定，在Google硬是做了两个多月。其中搞清楚用户需求和设计接口（Request里有哪些东西，Response里有哪些东西）就花了两周，想service的名字花了一周，开发+写单元测试用了三周，上线又花了一周，最后帮助一个用户迁移+配置权限又花了两周。这样还是超额完成了任务。。。

值得一提的是，Boq强制要求代码符合所有的Java最佳实践，比如public的class要么是final的，要么是abstract的。按照这些要求写出来的代码，往往会更长。最终实习期间我提交了6,000多行代码。

## 转正面试

Google实习是没有return offer的，但在实习的最后两周可以参加两轮conversion interview来转正。没有实习的new grad拿Google的流程大概是 4轮店面 -> HC Review -> Team match -> Offer，而对于想转正的实习生来说则是 2个实习评价-> 2轮店面 -> HC Review -> Team match -> Offer。唯一的区别就是用实习期间的两个host的final evaluation来代替两轮店面，其他都一样。我的两轮转正面试都问了非常多的design题，甚至第一轮最后面试官在问什么都没搞清楚。结果一般在实习结束后四周内出来，前提是两个host都在那之前提交了final evaluation。

## Life@Google
我实习的Google总部有10来个食堂，每个食堂每天菜都不一样，往往中午12点去吃午饭得11点50开始查看菜单。除了食堂外，还有流动的food trucks。我总能在Google食堂找到我想吃的菜，只要我肯花功夫查菜单并且愿意走上二三十分钟去吃一顿饭。


![某天的午饭](/img/post/google-intern-2017/food.jpg)

除了免费的一日三餐外，工位附近还有Micro Kitchen提供各种奶制品、饮料、坚果、水果等，其中最受欢迎的应该是椰子水和芦荟水了，基本上早上10点半前就要被抢光。

通勤方面，Google在湾区各地都有员工可免费乘坐的gBus，甚至在Santa Cruz都有一站。园区有类似于Uber的免费打车服务gRide，还有园区随地可见的gBike。Mountain View园区停车位充足，我每天都开车上下班，基本早上10点左右到公司，晚上6点半离开公司（主要食堂6:30才开饭）。

很多Google员工周四下午三点之后就开始心不在焉了，因为周四下午经常会有内部的TGIF活动。这个活动一般由Google的两位联合创始人和CEO主持，介绍公司近几周内发生的事情、产品的更新或者分析财报。实习生也可以参加。现场一般得提前半个小时以上占位，TGIF同时也在大部分食堂直播，员工可以一边喝酒一边吃披萨一边看各种“内部消息”。

实习生专属的活动也十分多，不定期会有各种户外运动、美食挑战、游乐场活动，关键是大部分都还是在工作时间。。。

![Intern Cruise](/img/post/google-intern-2017/cruise.jpg)

![Greate America](/img/post/google-intern-2017/great_america.jpg)


![Geo Formal Day](/img/post/google-intern-2017/formal_day.JPG)
