---
layout: default
title: 【翻译】Kindle Paperwhite 开发入门
author: Geansea
date: 2017-02-28
update: 2017-03-10
categories: Kindle
---

> 译注：[原文][source]写于 2015 年，在[上一篇文章][previous]（[原文][previous_en]）的基础上介绍了新机型上的开发。

[source]: http://www.kimhauser.ch/index.php/articles/kindle/hello-world-v2
[previous]: {% post_url 2017-02-20-getting-start-with-kindle-development %}
[previous_en]: http://cowlark.com/kindle/getting-started.html

## 背景

这篇教程基于如下环境编写：

| **开发环境** | MacBook Pro, OS X Mountain Lion, Eclipse, ANT
| **Kindle 设备** | Paperwhite (K5)
| **固件版本** | 5.3.5
| **API 版本** | 2.2 (Kindlet-2.2.jar)

为了基于自己的 Kindlet 做开发，我想找个 Hello World 示例来入门，于是我搜索到了 [cowlark.com 的文章][previous_en]（译注：[翻译版][previous]）。虽然这篇文章提供了很好的 Kindle 开发入门知识，但是它是基于 Kindlet-1.2.jar 的，无法解决我的问题，我只好再次搜索 Kindle API 2.2 有什么改动。这就是我写这个 Hello World V2 示例的原因。它基于 cowlark.com 上的 Hello World 示例，增加了对于 Kindle API V 2.2（Kindlet-2.2.jar）的改动。另外，会向你演示 `KMenu`（Kindle Menu）和 `KOptionPane.showMessageDialog`（Message Box）的使用。这篇教程也展示了如何修改自动化脚本，之前的脚本是为老版本 Kindle 写的，不适用于 Paperwhite。为了更好地理解教程的内容，请先阅读 cowlark.com 的文章。

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

为了顺利在设备上运行你的 Kindlet 应用，你需要用开发者密钥签名，相应的公钥也需要安装到设备上。你可以自己生成密钥。（译注：更建议使用 **公用** 密钥，不用自己生成，并且易于分发。例如使用 KUAL 的密钥，基本越狱设备上都有。）

我们使用 keygen-0.1（来自 cowlark.com）来生成密钥，只需要运行 keygen 脚本然后按照屏幕提示操作。最后你会得到两个文件，一个 developer.keystore（用于签名），一个 public.keystore（需要导入到设备用于验证）。

#### 3.1 本地 keystore 安装

拷贝 developer.keystore 文件到 ~/.kindle/kindle.keystore。

#### 3.2 设备 keystore 安装

keygen 无法生成适用 Paperwhite 的安装包，所以我们需要手动导入。这比本地安装要多好几个步骤，好在我们有工具帮助。我们使用 Java keytool（keytool - Key and Certificate Management Tool），基本上步骤很简单。

1. 从设备获取 developer.keystore（位于/var/local/java/keystore/）
2. 备份 .keystore 文件（原始的和新的）
3. 在本地用 keytool 将设备 developer.keystore 和新的 public.keystore 合并（参考下面的语法）
4. 将合并后的 developer.keystore 导入设备的 /var/local/java/keystore/ 路径（替换或将旧的改名）

**keytool 语法：**

{% highlight shell %}
keytool -importkeystore -srckeystore public.keystore -destkeystore developer.keystore
{% endhighlight %}

**注意！**

修改后需要重启设备。

### 4. 修改源代码以便基于 Kindlet-2.2.jar 编译

你可以在教程最下方的 **资源下载** 部分获取最终版本，也可以下载 [cowlark.com 的原始版本][hello_world]并修改源代码以便基于 Kindlet-2.2.jar 编译。

[hello_world]: http://cowlark.com/kindle/HelloWorld.zip

#### 4.1 详细改动

##### 4.1.1 Main.java

基本上这里需要改的只有 `KTextArea`，它在 API 2.2 里已经不存在了，但我们可以安全地使用 `JTextArea`。源代码应该为如下样子：

{% highlight java %}
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
{% endhighlight %}

##### 4.1.2 Manifest 文件

API 2.2 的 manifest 文件增加了三个必须设置的属性，否则会得到编译错误。manifest 应该为如下样子：

{% highlight shell %}
Manifest-Version: 1.0
Main-Class: ch.kimhauser.kindle.helloworldv2.Main
Implementation-Title: HelloWorldV2
Implementation-Version: 1
Implementation-Vendor: Kim David Hauser
Extension-List: SDK
SDK-Extension-Name: com.amazon.kindle.kindlet
SDK-Specification-Version: 2.1
{% endhighlight %}

##### 4.1.3 拓展：`KMenu` 和 `KOptionPane.showMessageDialog` 的使用

要在左上角添加一个标准菜单是小事一桩。在 `onKindletStart` 里创建一个 `KMenu` 并添加 `KMenuItem` 对象，然后调用 `KindletContext` 的 `setMenu` 方法。你可以在目录项上使用 `ActionListener` 来截取选择事件。下面的例子通过信息框（`KOptionPane.showMessageDialog`）显示了选择菜单项后的操作指令，源代码如下：

{% highlight java %}
public class Main extends KindletWrapper implements ActionListener {

@Override
public void onKindletStart() {
    KindletContext context = getContext();

    KMenu mnu = new KMenu();
    mnu.add("Example Menu", this);
    context.setMenu(mnu);
    ...
}

...

@Override
public void actionPerformed(ActionEvent arg0) {
    KOptionPane.showMessageDialog(getContext().getRootContainer(), "Action command: " + arg0.getActionCommand());
}
{% endhighlight %}

### 5. 编译、打包并签名

要编译工程，你可以把 ANT 的 build.xml 导入 Eclipse（通过菜单 New > Project > Java > Project from existing Ant Buildfile），然后使用 Eclipse 构建（创建一个新的构建：Project properties > Builders > New > Antbuilder）。或者，如果你已经安装了 ANT，直接在终端运行 makekindlet 脚本即可。

#### 5.1 脚本构建

makekindlet 脚本会编译并签名，最终得到一个 azw2 文件来上传到设备。脚本内容为：

{% highlight shell %}
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
{% endhighlight %}

### 6. 把 azw2 上传到设备

如果一切顺利，你现在应该准备把 azw2 文件上传到设备的 /mnt/us/documents 目录了（译注：/mnt/us 是以 USB 存储设备连接时的根目录）。比如说你可以把下面的命令加到上面的脚本末尾：

{% highlight shell %}
ssh root@192.168.15.244 rm -f /mnt/us/documents/$JAR
scp $JAR root@192.168.15.244:/mnt/us/documents
{% endhighlight %}

## 资源下载
* 适用于 Paperwhite 的 [Hello World V2][hello_world_v2]（715 KB）
* cowlark.com 原版 [Hello World][hello_world]
* 用于生成密钥的 [keygen-0.1][keygen]

[hello_world_v2]: http://www.kimhauser.ch/downloads/kindle/HelloWorldV2.zip
[keygen]: http://cowlark.com/kindle/keygen-0.1.zip

> 译注：Hello World V2 的下载已经失效，可以参考 [Github][hello_world_v2_github]。

[hello_world_v2_github]: https://github.com/jetedonner/ch.kimhauser.kindle.HelloWorldV2

## 致谢
* cowlark.com 的[教程][previous_en]
* Kim Hauser（译注：本文作者）的 Amendments for Kindle Paperwhite FW 5.3.5
