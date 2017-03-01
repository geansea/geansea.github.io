---
layout: default
title: 【翻译】Kindle Paperwhile 开发入门
author: Geansea
date: 2017-02-28
categories: Kindle
---

> 译注：[原文][source]写于 2015 年，在[上一篇文章][previous]（[原文][previous_en]）的基础上介绍了新机型上的开发。

[source]: http://www.kimhauser.ch/index.php/articles/kindle/hello-world-v2
[previous]: {% post_url 2017-02-20-getting-start-with-kindle-development %}
[previous_en]: http://cowlark.com/kindle/getting-started.html

## 背景

这篇教程基于如下环境编写：

| 开发环境 | MacBook Pro, OS X Mountain Lion, Eclipse, ANT
| Kindle 设备 | Paperwhite (K5)
| 固件版本 | 5.3.5
| API 版本 | 2.2 (Kindlet-2.2.jar)

为了基于自己的 Kindlet 做开发，我想找个 Hello World 示例来入门，于是我搜索到了 [cowlark.com 的文章][previous_en]（译注：[翻译版](previous)）。虽然这篇文章提供了很好的 Kindle 开发入门知识，但是它是基于 Kindlet-1.2.jar 的，无法解决我的问题，我只好再次搜索 Kindle API 2.2 有什么改动。这就是我写这个 Hello World V2 示例的原因。它基于 cowlark.com 上的 Hello World 示例，增加了对于 Kindle API V 2.2（Kindlet-2.2.jar）的改动。另外，会向你演示 `KMenu`（Kindle Menu）和 `KOptionPane.showMessageDialog`（Message Box）的使用。这篇教程也展示了如何修改自动化脚本，之前的脚本是为老版本 Kindle 写的，不适用于 Paperwhite。为了更好地理解教程的内容，请先阅读 cowlark.com 的文章。

## 概述
1. 阅读 cowlark.com 的文章
2. 从设备导出依赖库
3. 创建开发者密钥并导入设备
4. 修改源代码以便基于 Kindlet-2.2.jar 编译
5. 编译、打包并签名
6. 把 azw2 上传到设备

## 教程

### 1. 了解原理

请先阅读 cowlark.com 的文章！

### 2. 从设备导出依赖库

要基于 Kindle API 构建代码，需要先从你的设备获取一些文件。你可以手动操作或者借助 cowlark.com 介绍的 Kindle 库提取器（适用于老版本 Kindle）。

cowlark.com 介绍的 Kindle 库提取器没有提供 Kindle 3 之后的版本，因此无法在 Paperwhite 上使用，但是手动获取依赖库并不麻烦。KDK 的主要 jar 位于/opt/amazon/ebook/lib/Kindlet-2.2.jar。cowlark.com 建议复制此文件夹中的所有 jar，但实际上我从没用过其他的，取决于你想做什么了。无论如何，对于本教程，你只需要 Kindlet-2.2.jar。然后你需要将这个 jar 包含到项目构建路径中，以便编译器和链接器使用。

### 3. 创建开发者密钥并导入设备

To successfully run your Kindlet on a device you have to sign the project with your developer key. The public key must also be installed on the device you want to run the Kindlet. You can create the keys by yourself.

We use the keygen-0.1 tool (from http://cowlark.com/kindle/getting-started.html) to generate a key pair for our user. This tutorial assumes you use a password of password for your keystores. Just execute the keygen script and follow the advice on the screen. As a result you should get two files. A developer.keystore and a public.keystore - use these files for signing the project (developer part) and upload to target device (public part)

3.1 Local installation of keystore

Copy new developer.keystore to ~/.kindle/kindle.keystore

3.2 Install keystore on remote

Because the keygen creates no installer.bin for the paperwhite you have to merge the keystores manually. This involves a few more steps than the local installation but we got help from a tool. This is done by the Java keytool (keytool-Key and Certificate Management Tool). Basically the steps are simple.

Download developer.keystore from Device (/var/local/java/keystore/)
Make a backup copy of the .keystore files (original and new one)
Merge remote developer.keystore and new public.keystore locally with keytool (See syntax below)
Upload the merged keystore to /var/local/java/keystore/ (Replace or better rename old developer.keystore)
Syntax keytool:

keytool -importkeystore -srckeystore public.keystore -destkeystore developer.keystore

Important!

You have to restart the device after you changed the keys

### 4. 修改源代码以便基于 Kindlet-2.2.jar 编译

你可以在教程最下方的 **资源下载** 部分获取最终版本，也可以下载 [cowlark.com 的原始版本](http://cowlark.com/kindle/HelloWorld.zip)并修改源代码以便基于 Kindlet-2.2.jar 编译。

#### 4.1 Detailed amendments

##### 4.1.1 Main.java

Basically the only thing you have to change here is the KTextArea. In API 2.2 it does not exist anymore. But we can safely use JTextArea. Your source could look something like this:

import com.amazon.kindle.kindlet.KindletContext;
import javax.swing.JTextArea;
import com.cowlark.kindlet.KindletWrapper;

public class Main extends KindletWrapper {
  JTextArea kta = null;
  
  @Override
  public void onKindletStart() {
      KindletContext context = getContext();
      kta = new JTextArea("Hello World V2 example for Kindlet-2.2.jar");
      context.getRootContainer().add(kta);
  }
}

##### 4.1.2 Manifest file

The projects manifest file for API 2.2 has 3 more properties you have to set. If you fail to you get a compiler error. The manifest should read as follows:

Manifest-Version: 1.0
Main-Class: ch.kimhauser.kindle.helloworldv2.Main
Implementation-Title: HelloWorldV2
Implementation-Version: 1
Implementation-Vendor: Kim David Hauser
Extension-List: SDK
SDK-Extension-Name: com.amazon.kindle.kindlet
SDK-Specification-Version: 2.1

4.1.3 Extension: Usage of KMenu and KOptionPane.showMessageDialog 

Adding a menu to the top left standard menu is a piece of cake. Use a new KMenu and add MenuItems to it in onKindletStart. When you're done you can use the setMenu function on the KindletContext. The menu then will be appended to the existing one(s). You can use an ActionListener on the menu items to catch the selection event. This example shows you the Action command of the selected menu item with a message box (KOptionPane.showMessageDialog). The source looks like this:

public class Main extends KindletWrapper implements ActionListener {

@Override
public void onKindletStart() {
    KindletContext context = getContext();

    KMenu mnu = new KMenu();
    mnu.add("Example Menu", this);
    context.setMenu(mnu);

...

@Override
public void actionPerformed(ActionEvent arg0) {
    KOptionPane.showMessageDialog(getContext().getRootContainer(), "Action command: " + arg0.getActionCommand());
}

5. Compile, pack and sign source

To compile the project you can either import the ant build.xml into Eclipse (via menu New > Project > Java > Project from existing Ant Buildfile) and build it with Eclipse (Create a new builder: Project properties > Builders > New > Antbuilder) or run the makekindlet script directly from within console if you have ant installed on your system

5.1 Makefile

The makekindlet script will compile the project and create a signed azw2 file you can upload to your device. It looks like this

#!/bin/sh
FILENAME=HelloWorldV2
KEYSTORE=$HOME/.kindle/kindle.keystore
JAR=$FILENAME.azw2
MANIFEST=$FILENAME.manifest

ant jar

cp $FILENAME.jar $JAR

jarsigner -keystore $KEYSTORE -storepass password $JAR dk$USER
jarsigner -keystore $KEYSTORE -storepass password $JAR di$USER
jarsigner -keystore $KEYSTORE -storepass password $JAR dn$USER

6. Upload azw2 to device

If everything has gone well you now should be ready to upload the created azw2 file to the device at /mnt/us/documents. For example you can use the following commands to modify the above script

ssh root@192.168.15.244 rm -f /mnt/us/documents/$JAR
scp $JAR root@192.168.15.244:/mnt/us/documents

Source & Downloads
Hello World V2 for Paperwhite (K5)
HelloWorldV2.zip (715 KB - 29.06.2013)

Original Hello World from cowlark.com
http://cowlark.com/kindle/HelloWorld.zip (alternative download)

keygen-0.1 (for signing project)
http://cowlark.com/kindle/keygen-0.1.zip (alternative download)

Credits
Original tutorial by http://cowlark.com
Amendments for Kindle Paperwhite FW 5.3.5 by Kim Hauser
