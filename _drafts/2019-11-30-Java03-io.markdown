---
layout: post
title:  "JAVA学习（三）——泛型"
date:   2019-11-21 09:00:00 +0800
categories: JAVA
tags: JAVA IO
---

之前有看过关于泛型的一些知识，但是有一点忘了，决定再梳理一下。JAVA泛型是从JKD1.5开始的，将所操作的数据类型作为参数的一种语法。

{% highlight java %}
public class Play<T>{
    T play(){}
}
{% endhighlight java %}

在这里，Play类通过传入一个类型T，设置了一个方法返回的类型。

