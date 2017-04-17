---
layout: default
title: Kindle 开发纪要 2
author: Geansea
date: 2017-04-17
update: 2017-04-17
categories: Kindle
---

忍不住继续研究了 Kindlet 开发，主要是基于 1.3 SDK 的（非 touch 版），把遇到的坑和解决方法留个记录。

## 网络请求问题

对于未注册设备，可以通过 runtime 调用 curl 命令行实现网络请求。

## 权限问题

通过引入 jailbreak.jar 可以让应用获取足够的权限，具体可以看 KUAL 的代码。

## 线程安全

借助 HelloWorld 里的 eventbus 可以实现安全地异步调用，就是要写 event 和 event handler 两个类比较繁琐。

## 类的使用

看到一些地方说应该基于 Java ME 的 SDK 进行开发，但是没找到 Mac 版，并且为了少装一点程序，用的 Android Studio 进行的开发，先将就吧。

遇到的一个坑是，如果在会被实例化的类里用了不支持的方法或类（基于 Java SDK 1.4 可以编译通过，但是 Kindle 上没有），即使代码不会被执行到，都会导致应用卡死，一开始被坑了好久。

## 界面和跳转

尝试了好些方式，最后发现 chitanka4kindle 的基于 screen 的方式比较好，不太了解 JApplet 的开发，不知道是不是一个常见的做法。

## 中文显示

这个也被坑了两天，一个是 `KLabel` 和 `KLabelMultiline` 默认不支持汉字，需要设置 `new Font("hans", Font.PLAIN, size)` 来显示中文字体，`hans` 这个字体名来自 /usr/java/lib/font.properties，要支持繁/日/韩的话也可以找到相应的 family name。

另外就是在代码里写中文字符串，直接写是有问题的，无法正确显示，暂时找到的办法是用 `\uXXXX` 的写法。

而 `KTextField` 和 `KTextArea` 不存在上面两个问题。
