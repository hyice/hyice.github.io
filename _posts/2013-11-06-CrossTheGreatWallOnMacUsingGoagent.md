---
layout: post
title: Mac翻墙指南+浅析goagent
date: November 11, 2013
category: mac
tags: mac safari goagent 翻墙
excerpt: 平时用google搜索的时候，难免会碰到一些敏感词汇，这时候我们就必须得等待几分钟，然后小心翼翼地去除敏感词汇后才能继续搜索。这不免让我们感到不爽，更不用说对youtube这些神奇网站的强烈好奇心了。于是翻墙成为了我们的必然选择，那么如何翻墙呢？翻墙的原理又是什么呢？

---
关键字：`mac` `safari` `goagent` `翻墙`

##背景
平时用google搜索的时候，难免会碰到一些敏感词汇，这时候我们就必须得等待几分钟，然后小心翼翼地去除敏感词汇后才能继续搜索。这不免让我们感到不爽，更不用说对youtube这些神奇网站的强烈好奇心了。于是翻墙成为了我们的必然选择，那么如何翻墙呢？翻墙的原理又是什么呢？

##用什么翻墙
我对翻墙工具了解的不多，因为我一开始用的就是 goagent + Chrome 的组合。最近升级了Mavericks，因为听说Safari的性能得到了很大的优化，就改用了 goagent + Safari 的组合，然后就研究了下如何在Safari下实现翻墙。

所以，这篇文章的主要内容就是在Safari下实现用goagent翻墙，加上对goagent的一点浅显分析。

如果你不想使用goagent，而是想用诸如自由门这样的软件的话，请出门右转。

##安装并使用goagent
关于这一部分，网上的教程已经泛滥成灾了，我自己也是看着其中的一些教程学会的，建议大家可以直接google一下，或者看一下我挑选的这篇攻略，额，准确说是[官方教程](https://code.google.com/p/goagent/wiki/InstallGuide)。

如果你使用的是Chrome，Firefox或者Opera的话，那么你可以满足地离开了，或者花点时间读读我对goagent的简单理解。

##Safari该如何使用代理
我们先不谈原理，先来说说用Safari该怎么设置代理。有没有像Chrome那样方便使用的插件呢？我觉得我下面这个比插件更方便，因为完全原生无污染哦！

打开`系统偏好设置`>`网络`>`位置`，然后可以看到如下界面：

![image](http://hyice-github-io.u.qiniudn.com/20131111-1.png)

然后点击`编辑位置`，新增一个名为goagent的位置.

![image](http://hyice-github-io.u.qiniudn.com/20131111-2.png)

然后确认，进入`高级`设置，选择`代理`的那个选项:

![image](http://hyice-github-io.u.qiniudn.com/20131111-3.png)

最后勾选上`Web 代理`，然后代理服务器框里填上goagent的代理地址就可以了。

![image](http://hyice-github-io.u.qiniudn.com/20131111-4.png)

当我们要使用代理的时候，在左上角菜单里即可进行快速选择。

![image](http://hyice-github-io.u.qiniudn.com/20131111-5.png)

##goagent原理浅析
我们要浏览一个页面，一般来说是不可以从我们的电脑直接获取到目标服务器上的数据的，中间往往经过了很多服务器的中转。

那么整个过程便可以这么理解：

1. 我们在我们的电脑上访问了一个站点
2. 我们的电脑发出了一个请求，要获取目标站点的数据
3. 这条请求被我们电脑和目标服务器之间的服务器不断传递，最后到达目标服务器
4. 目标服务器处理这条请求，然后决定把数据传给我们的电脑
5. 目标服务器传送的数据经过中间的服务器不断传递到达我们的电脑
6. 我们的电脑处理获得的数据，然后把页面展示在我们面前

但是，这个过程并不是每次都可以顺利完成的。在忽略网络原因或者目标服务器坏了的情况下，我们还有伟大的GFW孜孜不倦地工作着，我们可以把GFW理解成其中一台必定会参与传递的中间服务器，当他发现要他传送的请求或者数据不符合他的规则时，他就会选择不传递，那么我们自然无法顺利加载我们的页面了。

那么，goagent又是怎么骗过了伟大的GFW呢？首先，附一张原理图：

![image](http://hyice-github-io.u.qiniudn.com/20131111-6.png)

然后我来简单地讲解下：

1. goagent在本地运行的client成为了我们的第一个中转服务器，每一个我们发出的http请求都会先到达这个服务器。
2. 这个服务器做的事不是简单地把请求往下一个服务器传递，而是先对数据进行了一次封装，然后再把数据往后传送
3. 往后传送的时候，目的地不再是原来的目标服务器，而是位于GFW外部的GAE服务器，封装过的数据到达GAE服务器后，会被解封，然后把数据发往我们原来要求的服务器。
4. 这样，目标服务器就可以正确接受我们的请求包了，然后会按照正常流程相应，并把数据传给GAE服务器
5. GAE服务器对响应的数据包进行封装，安全地传回到我们的Client
6. Client对封装的数据进行解封处理，传给浏览器进行加载，然后我们就看到了正确的页面了。

##参考内容
1. [goagent的官方文档](https://code.google.com/p/goagent/)
2. [知乎上关于goagent原理的回答](http://www.zhihu.com/question/20370059)
