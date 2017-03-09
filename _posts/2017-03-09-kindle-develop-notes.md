---
layout: default
title: Kindle 开发纪要
author: Geansea
date: 2017-03-09
update: 2017-03-10
categories: Kindle
---

## 纪要

前面翻译过的两篇文章，涵盖了 Kindle 非触屏和触屏两种版本的开发，对应 Kindlet-1.x 和 Kindlet-2.x。

> [【翻译】Kindle 开发入门][previous1]
> [【翻译】Kindle Paperwhite 开发入门][previous2]

[previous1]: {% post_url 2017-02-20-getting-start-with-kindle-development %}
[previous2]: {% post_url 2017-02-28-kindle-paperwhite-hello-world-v2 %}

另外可以参考 [KUAL 工程][kual]，在这两个 Hello World 工程上做些修改。例如把编译参数、jar 签名操作都写到 build.xml 里，以便在 Windows 和 Linux／Mac 上都可以直接编译。

[kual]: https://bitbucket.org/ixtab/kindlelauncher

另外签名时可以直接使用 KUAL 工程里提供的 developer.keystore。这样的话，只要设备越狱了，就可以跑你的应用，免去了合并 keystore 并导入这样的步骤，分发给别人时也更加方便。

## 杂项记录

淘宝的二手 Kindle，有些会注明不能绑定，其来源有几种说法：真实二手，但是卖掉时没有注销旧账号；被盗设备，已被挂失；加工厂私自加工，但是无法取得亚马逊的设备号。通常会被重刷一个固件，设备号除了开头标示设备类型的外都改成 0，无法绑定（意味着无法访问亚马逊书城，不能使用推送），只能通过自带浏览器上网，基本就是一个安静的阅读器。同时，使用 Kindlet 开发时，访问网络的方法会抛出失败异常，提示未注册。尝试伪注册了一下，依然不能访问，抛出一个因为“不明原因”的失败。没有继续深究。

从 Kindle Oasis 和 Kindle Touch 3 开始，亚马逊停止支持 Kindlet 了，也许对他们来说，这个勉力维持了 5 年的框架到了寿终正寝的时候了。我们依然可以跑自制的 jar 包，但是要换一种方式来引导了，具体可以参考 KUAL 的代码。简略来讲，就是把 jar 包导入进去，在数据库里注册一种新的文件格式（*.kual），并将这种新格式文件的打开方式注册为我们的 jar，然后只要在 documents 目录里生成一个相应的空白文件（KUAL.kual），点击它就会引导起我们自己的代码了。

## 感想

从 2011 年工作开始接触 Kindle，一转眼已经快 6 年了，只在别人搭好的 GUI 下开发过，一直没能好好研究下 Kindle 的开发环境搭建。前段时间偶然发现新的越狱出了，跟着操作了一遍，又借着有些空闲找了很多相关的文章，终于在这几天弄清了开发的原理，也在 Kindle 4 和 Kindle Touch 2 上都跑通了，最终发现这项技术已经没有什么价值了。

越狱的门槛越来越高，Kindlet 停止了维护和支持，加上 Kindle 的定位日渐固定，在其上的个性化开发已经没有了市场，就当作了却了一个心愿了。
