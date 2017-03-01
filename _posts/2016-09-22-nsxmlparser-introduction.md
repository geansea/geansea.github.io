---
layout: default
title: NSXmlParser 的使用和问题总结
author: Geansea
date: 2016-09-22
update: 2017-03-01
categories: Cocoa
---

## 基础使用

标准的 SAX 解析，简单使用时只需要实现三个 `delegate`：

{% highlight objc %}
```
- (void)parser: didStartElement: namespaceURI: qualifiedName: attributes:;

- (void)parser: didEndElement: namespaceURI: qualifiedName:;

- (void)parser: foundCharacters:;
```
{% endhighlight %}

## 优点

SDK 直接提供的 SAX 解析器，不用依赖第三方库就可以解析简单的 XML 文档。

## 问题

虽然提供了很多的 `delegate` 来实现，但是可定制性并不强，就使用的情况看，还是只适用于较为简单和可控的场景。例如将 XML 解析为 `NSDictionary` 或类似的字典结构，或者解析简单的 HTML 文档。

### 用于本地解析时有限制

如果要引入命名空间等，去需要确保可以访问网络，以便解析对应的 URL，这点不利于本地处理。另外如果引入 DTD，同样面临这样的问题，虽然可以引用到本地文件，但是相应的 XML 或 HTML 必须预先作修改，导致比较麻烦。

### 需要自行处理空白字符

默认需要自行处理空白字符，和常用的 XML 规范不太一样。
