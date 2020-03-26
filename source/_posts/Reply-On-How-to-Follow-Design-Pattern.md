---
title: '给一位Udemy学员的回复（之一）：Follow Design Pattern的正确姿势'
date: 2020-03-26 09:52:43
tags:
  - Udemy Course
  - 技术面试
  - 设计模式
categories: Udemy教程
keywords: Udemy,坦克大战,设计模式,Design Pattern
---

去年在Udemy上发了两门课，算是了却了一桩心事，当然，也多出来N桩心事——时不时有学员就“[放码过来！作个Java实战派](https://www.udemy.com/course/java-warrior-part1/?referralCode=5777839AB585A9602DFE)”、“[放码过来！新版Java坦克大战](https://www.udemy.com/course/java-tank-war/?couponCode=JAVANEVERSLEEP-MAR20)”提问，目前为止，所有提问我都按承诺24小时内回复——当然，通常是几小时内就回复。但是，一位学员来信提到的下面这个问题，我却首次破例了😔

[![](/img/udemy-tankwar.jpg)](https://www.udemy.com/course/java-tank-war/?couponCode=JAVANEVERSLEEP-MAR20)<!-- more -->

这位台湾小帅哥问的是什么呢？

> Nathanael老師您好, 我即將完成您的tank課程！這一路學到很多, 但是也有發現自己有很多不足。特別是在OOD的部分, 很難想到像老師您一樣有系統性的去設計一個一個的class, 想請問老師您有follow什麼Design pattern嗎？
>
> 另外, 我想將這個project寫在履歷上, 但是我沒有做過這種project, 所以我不知道從何寫起, 以及一個面試官, 一個senior希望從這個項目看到什麼樣的重點及技能。能不能請老師稍微分享一下您的經驗, 若您是面試官, 您希望在這個project上看到我學會了什麼, 以及什麼市值得寫在履歷上的呢？
另外雖然我是一個newbie, 但是希望老師之後課程可以加入一個design pattern 或是一個更詳細high level 的 project idea, 謝謝老師的教導～！

为了回答这些问题，我至少掉了三十根头发。厌烦絮叨，只想看看该如何行动的急性子，可以直接跳到“[读一本书，行万里路](#读一本书，行万里路)”。

### 我Follow了什么Design Pattern？

长话短说，我没有刻意Follow什么Design Pattern。对坦克大战这样一个小型项目，设计Class实际是水到渠成的过程。分析需求、迭代开发和测试，一个个Class的出现就会是顺理成章的事情。这些Class的一些共性，随着开发过程的深入会慢慢浮现出来，在此基础上，我们可以作一些必要的抽象和剥离，比如[`GameObject`](https://github.com/ny83427/tankwar/blob/solution/src/GameObject.java)是在后期——而不是一开始就出现。

有两种思考进路：先具体再抽象，先抽象再具体。体现在开发过程中，可能就是在动手写代码之前，设计、规划需要做到何种程度的问题。这两种进路，根据情况的不同，使用的方式也不同。但大体来说，我更习惯于先具体再抽象，因为这更能保证抽象是必要的、最小的。可能有人担心没有想清楚就开干是否会导致很多返工，有可能，但是不幸的是，我不是那种一开始就可以想清楚的人😭当然，这并不可怕，有重构和各类测试保驾护航，小改、大改甚至魔改都不是问题。

实际上，“很難想到像老師您一樣有系統性的去設計一個一個的class”，这主要是一个经验积累的问题，与Design Pattern几乎无关。完成的项目越多，掉进的坑、踩过的雷越多，当拿到一个需求并完成分析之后，OOD的设计会在自然不断的迭代中完成更新，并逐步达到一个较为理想的状态。所以，少年，日写代码三百行，不辞长作程序猿。

我所参与不同公司的项目中，通常当一个Feature下来之后，项目组的资深架构师会要求开发人员完成一个初始设计并进行评审，其主要目的是避免开发人员的思路太不合理——以致几乎可以确定这么折腾必定会埋下N多地雷，评审不会期望在正式开发之前，可以事无巨细、面面俱到地规划未来。

### Follow Design Pattern的正确姿势

先下结论：根据需要Follow，为了让代码变得更好去Follow。这很废话，但是我也只能这么说了，抱歉。

比如坦克大战中，`GameClient`是个全局唯一的对象，那么，当游戏中的其他对象需要访问它时，我们需要确保`GameClient`在虚拟机中只有一个实例，如何做到？

单例模式（Singleton）就自然而然出现，饿汉式（Eager Singleton）单例也会是我们的第一选择：最简单、最易于实现。

饿汉式单例可以运行得很好，直到某天你发现，为了让饿汉吃饱，为了让这个单例被构建出来，系统的启动时间显著增加了，而实际上系统启动时我们并不需要这条汉子，仅仅只在某个下雨的星期天，我们才想起这汉子，叫他给我们送外卖。

于是懒汉式（Lazy Initial Singleton）应运而生，仅当第一次用到时，才去初始化。

懒汉式单例可以运行得很好，直到某天你发现，有一定的概率，系统中悲剧地出现了两条汉子——天下大乱了！

于是懒汉式单例的线程安全问题浮出水面，方法级`synchronized`，还是代码块`synchronized`，如果是代码块`synchronized`，是否需要双保险才能保障？

单例问题、模式似乎就到此终止了，直到某天你发现，前Google工程师Bob Lee在折腾[Initialization on Demand Holder (IODH) idiom](http://blog.crazybob.org/2007/01/lazy-loading-singletons.html)，你不禁感叹：还能这么玩？

```Java
static class SingletonHolder {
  static Singleton instance = new Singleton();    
}

public static Singleton getInstance() {
  return SingletonHolder.instance;
}
```

看清楚了吗少年？我们不需要刻意Follow什么Pattern，但是在解决一个个问题的过程中，我们会自然而然地接触到这些Pattern，了解它们合适的使用场景。某些时候，像Crazy Bob这样脑洞清奇的大牛，还能时不时给你来点猛料和惊喜😄

接下来我们再谈一下“为了让代码变得更好去Follow”，这里的好代码，我们尽可能不搞印象派，而暂时只从以下三个方面衡量：

- 代码Bug多吗？测试同事喜欢你吗？
- 需求变更，改起来方便快捷吗？改完之后心里踏实吗，还是要烧香拜佛？
- 代码方便测试、易于测试吗？还是改完之后因为太难测试，干脆就不测试了？

就先只谈这三个很直观的方面，暂时略过其他不太直观的东东。

设计模式使用得当，可以确凿无疑地让代码易于测试、易于维护、Bug最少，实际上，代码写多了，出于对更高更强更快的追求和折腾，有一些设计模式你会无师自通，因为英雄所见，一般都略同嘛。

比如，你受不了裸写JDBC这种非人类的搞法，你发现模板模式让你的生活多少美好了一些：你要写的代码显著变少了，容易搞出问题的复制黏贴基本没了，要升级到数据库连接池也只需要修改一处了...

### 读一本书，行万里路

最后，我的建议是快速通读一本关于设计模式的好书，然后丢下它，在痛苦而漫长的代码生涯中去自然而然地Follow他们。为赋新词强说愁的Follow方式很危险，很容易为了设计模式而设计模式。

如果只能推荐一本，那么我会推荐这本小人书：[Head First Design Patterns: A Brain-Friendly Guide](https://amzn.to/2vQ4YcA)，中文译本是“[深入浅出设计模式](https://amzn.to/2wGRT5L)”。相信我，她比四人帮的名著更适合绝大部分人。

[![](/img/head-first-design-patterns.jpg)](https://amzn.to/2vQ4YcA)

关于这个项目如何用在简历中的问题，下回再说，等掉够了四十根头发，应该就成了。