---
layout: blog
title:  "Java 中的运算符"
date:   2015-05-12 14:30:00
categories: java
permalink: /java-floating
author: Kylin Soong
duoshuoid: ksoong2015051201
excerpt: Java 浮点运算符
---

### 浮点乘运算符 <<

浮点乘运算 a << b 即为 a * (2^b)，例如：

* 1<<0  -> 1 * (2^0)  = 1           (1 字节)
* 1<<10 -> 1 * (2^10) = 1024        (1 KB)
* 1<<20 -> 1 * (2^20) = 1048576     (1 MB)
* 1<<30 -> 1 * (2^30) = 1073741824  (1 GB)

### 浮点除运算符 >>

浮点除运算 a >>b 即为 a / (2^b)，例如：

* 1073741824>>10 -> 1073741824 / (2^10) = 1048576 (GB 转化 MB)
* 1048576>>10    -> 1048576 / (2^10) = 1024       (MB 转化 KB)
* 1024>>10       -> 1024 / (2^10) =1              (KB 转化字节) 

NOTE: 一个应用实例, Teiid BufferManager 默认设定的 MaxBufferSpace 为 50 GB(50L<<30)  
