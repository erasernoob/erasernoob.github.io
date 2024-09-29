---
title: 2024su-week4
date: 2024-09-23 00:12:07
tags: daliyRecord
---

## week4

直接进入正题，这一周都是疯狂的**Spring**的一周。

### 基础知识

- SpringSSM -> SpringMVC -> SpringSeqcurity 

从这三部曲学走，则是正式的进入了，Springboot部分。但是就像上文所提到的，是疯狂的一周，是囫囵吞枣的一周。所以还有很多细节的东西，很多实现原理还没有了解透彻。但是至少是过了一遍，有了一遍好的印象。学会了如何去写，如何去正确的操作。接下来则就放出之后需要回去回顾的一些地方，在这一周的学习看视频的过程中，也跳过了一些部分：

#### Spring阶段

- [Spring核心内容的底层实现（源码分析）](https://www.itbaima.cn/document/h7sjo5oy0l03607e?segment=4#doc7-%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E6%8E%A2%E7%A9%B6%EF%BC%88%E9%80%89%E5%AD%A6%EF%BC%89)
- [解读MVC下Dispatcher源码](https://www.itbaima.cn/document/eve8gq72qmdb46sg?segment=2#doc8-%E8%A7%A3%E8%AF%BBDispatcherServlet%E6%BA%90%E7%A0%81)

#### SpringSecurity阶段

这一阶段的源码和实现原理很重要，当时也有意无意的跳过或略过了。所以~~把知识交给未来的自己~~。

- SpringSecurity内部探究，主要是[过滤器](https://www.itbaima.cn/document/63v73g0zh1qlr6fk?segment=3#doc3-%E5%AE%89%E5%85%A8%E4%B8%8A%E4%B8%8B%E6%96%87%E6%8C%81%E4%B9%85%E5%8C%96%E8%BF%87%E6%BB%A4%E5%99%A8)等，基本原理的回顾，能够让后面开发更加灵活。

原理前面落下的了解完实现完之后，给自己开的一个大坑就是：

> ​	[想要彻底的弄懂Sring底层的内部原理，何不自己实现手搓一个SpringBoot呢](https://liaoxuefeng.com/books/summerframework/introduction/index.html)

### 实战项目

到目前为止，也就是在这一周的星期天，算是完成了那个以session进行校验的前后端分离的实战项目，后面还会继续将基于JWT进行验证的版本，继续跟着做完，将会是以一个项目为[两个分支的形式](https://github.com/erasernoob/Spring-Vue-proj0)。就算是完成了白马的SpringBoot阶段了。

在国庆节前一周中，除了完成上面的JWT验证阶段以外，在**<u>表面层</u>**，**就是去寻找一个完整的前后端分离项目继续跟着做了**。紧跟着的便是`SpringCloud`微服务阶段，慢慢来吧。差不多就是这一周，风平又呼啸的一周了。
