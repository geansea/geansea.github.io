---
layout: default
title: NSXmlParser 的使用和问题总结
author: Geansea
date: 2016-09-22
categories: Demo
---

## NSXmlParser 的使用和问题总结

### 基础使用

标准的 SAX 解析，简单使用时只需要实现三个 delegate：

### 优点

SDK 直接提供的 SAX 解析器，不用依赖第三方库就可以解析简单的 XML 文档。

### 问题：需要自行处理空白字符

### 问题：不适用于 HTML 解析

NSXmlParser 选择了非常严格的 XML 标准，如果解析标准的 XHTML 文档不会有问题，但是如果想用于解析规范比较混乱的 HTML 文档就会遇到很多问题。现在遇到的最主要的麻烦是 DTD，


