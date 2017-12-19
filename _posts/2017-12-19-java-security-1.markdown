---
layout: post
title:  "JAVA.Security学习笔记一"
date:   2017-12-19 17:00:00 +0800
categories: CA
tags: CA Certificate JAVA
---

## JAVA.Security包
java security包中包含了为安全框架提供的接口与类，与CA的使用息息相关

### 接口
Certificate      证书接口：该类已经不建议使用，因此不介绍。

DomainCombiner   保护域：该概念涉及到JAVA虚拟机的沙盒模型。对于一个系统，要保障运行程序的安全，需要对于运行在该系统上的程序做安全检查，确定没有有害代码，但是这种安全检查有两个问题，一个是安全检查需要耗费大量的精力，另一方面，安全检查是需要不断更新维护的。而对于JAVA程序是各种类运行在JVM（JAVA虚拟机上的），系统可以设置一个安全域，对于运行于系统的网络程序，只能在制定的沙盒中运行，可以执行各种操作，但是无法影响到沙盒外的系统，而这个安全域就是DomainCombiner。

- Guard 守卫接口：保护某个类不受非法访问。

- Key  密钥接口：是所有密钥的顶层接口。

- KeyStore.Entry：KeyStore证书库的（Mark Interface）标记接口

- KeyStore.LoadStoreParameter KeyStore证书库的标记接口

- KeyStore.ProtectionParameter KeyStore证书库的标记接口

- Policy.Parameters Policy的标记接口

- Principal 表示抽象实体（主体），比如个人、企业、ID等

- PrivateKey 私钥接口，并不包含具体的方法与属性，只是为了分类。

- PublicKey 公钥接口，并不包含具体的方法与属性，只是为了分类。

- PrivilegedAction 

- PrivilegedExceptionAction

对于上面几个接口，需要解释一下的有：

***Key***：密钥接口，是所有密钥的顶层接口。密钥具有三个特征：密钥算法、编码形式、编码格式名称。此类的子类中有多个CA常用的类，比如说PublicKey公钥、PrivateKey私钥、KeyPair密钥对持有人、KeyFactory密钥工厂等。

***标记接口***：KeyStore.Entry、KeyStore.LoadStoreParameter、KeyStore.ProtectionParameter都被标明了是标记接口，是用来标记类的某种属性的，比如说cloneable()接口就是用来表示该类是否支持深度拷贝，按照我的理解，标记接口只是为了标记某种属性，而不涉及具体的实现规范。

***DomainCombiner***：保护域接口


### 对象

由于这个包含有较多的类，针对CA证书的常用操作，主要介绍以下几个类：

- KeyFactory：作为密钥对象

- KeyPair：密钥对持有人对象，KeyPairGenerator是它的子类，平时我们可以用它来生成RSA的密钥对。了解它的工作原理可以帮助我们更好的理解国密算法的JAVA实现。

- KeyStore：密钥库，在JAVA keytools中，密钥库是存储公私钥的容器，可以放置多对密钥，是加密存储的。

- MessageDigest：摘要对象，可以做多种哈希运算。

- Provider：安全服务对象，可以使用第三方提供的服务。

- Signature:签名对象，用于数据的签名与验证数字签名。该对象的使用分为三个流程。第一个是初始化，比如传入公钥进行验证的初始化，比如说传入私钥进行签名的初始化。第二个是传入字符串，使用update方法进行，比如说签名原文与签名值。第三就是进行签名或验证操作。

### 对象使用
在这个包中使用的对象初始化方法很多都是Instance的方法，例如KeyFactory.getInstance()，关于这一部分为何这么使用，需要学习一下**JAVA设计模式**的相关知识。

#### KeyFactory
{% highlight java %}
KeyFactory keyFactory = KeyFactory.getInstance("DSA");
//初始化一个DSA证书对象
{% endhighlight %}

#### KeyPairGenerator

密钥对生成器在生成密钥对的时候，有两种初始化方式，一种是与算法无关的方式，另一种是指定算法的方式，两种对象初始化方式的代码不同，这里我们展示的是指定算法的初始化。

{% highlight java %}
KeyPairGenerator  keyPairGenerator = KeyPairGenerator.getInstance("RSA");
//制定密钥对生成器类型
keyPairGenerator.initialize(2048);
//初始化密钥对生成器
KeyPair key = keyPairGenerator.generateKeyPair();
//获取生成的密钥对
PublicKey publicKey = key.getPublic();
PrivateKey privateKey = key.getPrivate();
//获取公钥与私钥
{% endhighlight %}

#### KeyStore
请求一个KeyStore对象

{% highlight java %}
KeyStore ks = KeyStore.getInstance(KeyStore.getDefaultType());
//获取默认对象，KeyStore.getDefaultType()会返回字符串jks

KeyStore ks = KeyStore.getInstance("JKS");
//获取指定类型的密钥库，可选的参数有JKS与PKCS12（还有其他参数）

KeyStore ks = KeyStore.getInstance("JKS","proivder");
//还可以加上服务提供商，可以加在其他提供商的包，需要加载，比如说BouncyCastle的参数为"BC"
{% endhighlight %}

#### MessageDigest
摘要对象的使用分为两步，初始化摘要对象后，使用Update上传需要摘要的原文，然后使用digest方法来完成摘要计算。可以选择多种参数做不同类型的摘要，比如“SHA”，“MD5”等

{% highlight java %}
MessageDigest md = MessageDigest.getInstance("MD5");
//初始化对象
messageDigest.update(messge);
byte[] md5message = messageDigest.digest();

//直接完成摘要计算（包含了update和digest）
byte[] md5message = messageDigest.digest("messge");
{% endhighlight %}


