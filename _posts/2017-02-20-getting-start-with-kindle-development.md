---
layout: default
title: 【翻译】Kindle 开发入门
author: Geansea
date: 2017-02-20
categories: Kindle
---

> 译注：[原文](http://cowlark.com/kindle/getting-started.html)写于 2011 年，很多内容可能已经不适用

## 越狱（Jailbreak）

这是后面所有事情的前提：

> [Lu Yifan 的越狱包](http://yifan.lu/p/kindle-jailbreak/)
> （最新越狱方法可以参考 [Kindle 伴侣的总结](https://kindlefere.com/skills/jailbreak)）

越狱后就可以进行开发了，不需要再安装其他东西。

注意事项：
* **越狱不意味着可以直接解密 AZW 格式电子书。**
亚马逊对其电子书的 DRM 保护和设备越狱是两件完全不相关的事情。
* **越狱不意味着可以连接 Whispernet。**
授权使用 whispernet 和越狱是不同的机制，如果你想借此实现免费上网的话，还是不要折腾了。

越狱 **唯一** 的作用就是让你的设备能运行其他程序（包括可以把你的设备变砖的程序，对此要有思想准备）。

## 开发工具

Kindle 应用使用 Java 开发。

## 应用签名

## 一步到位

## 坑

现在给一些警告。

我认为亚马逊官方不允许自制 Kindle 应用有一个和业务无关的原因：Kindle 是一个非常 **脆弱的设备**。它太容易崩溃，而且在用户应用和系统界面之间没有隔离。

也就是说：
* 每当加载包含静态成员的类时，这些成员的内存 **永远** 不会被释放。
* 没有办法强制终止应用。不小心弄出一个死循环？那个线程将 **永远** 运行。
* 如果你让某个线程 exit 了，你的 Kindle 会崩溃。

这意味着，你在 Kindle 上开发和测试时，会频繁遇到崩溃重启。虽然到目前为止我没有遇到丢失数据的情况，但不敢保证你也这么幸运。

尤其是当你运行很多第三方应用时，设备会慢慢变得卡顿和不稳定，直到崩溃重启后恢复正常，然后再次重复。记住，Kindle 在正常情况下是 **永远** 不会重启的，这意味着那些难以察觉的内存泄漏，会随着时间的推移不断积累。

总结起来就是：亚马逊不想被用户指责说 Kindle 不可靠。通过对第三方应用进行审核，可以让他们至少测试一下是否适用。

## 致谢

这些工作大部分不是我做的，我只是把别人的工作成果整理和总结起来。特别是制作越狱包、签名工具等这些艰苦的工作，是 [mobileread.com](http://www.mobileread.com/) 上 [Kindle Developer's Corner](http://www.mobileread.com/forums/forumdisplay.php?f=150) 板块的优秀开发者们完成的。

另一个非常好的技术信息来源是[亚马逊官方开发者论坛](http://forums.kindlecentral.com/forums/index.jspa)……虽然我不建议发到那里。
