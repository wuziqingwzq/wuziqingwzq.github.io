---
layout: post
title:  "X.509学习记录（二）：X.509证书解析"
date:   2018-05-11 17:00:00 +0800
categories: CA
tags: X.509 CA Certificate ASN.1
---
## 前言
又看了以前写的内容，觉得比较晦涩，于是找了一张证书进行分析，希望能够将这一部分的内容讲清楚。

### ASN.1、X.509、RFC2459的关系
ASN.1：抽象语法定义:ASN.1(Abstract Syntax Notation One)是描述在网络上传输信息格式的标准方法。它是由国际电信联盟电信-标准分局（ITU-T）定义的标准之一。我们可以理解为各种信息系统间有互相沟通的语言，而ASN.1，就是这些通用语言的语法定义。

X509: X.509 是由国际电信联盟（ITU-T）制定的数字证书标准。它也是一个由国际电信联盟电信-标准分局（ITU-T）定义的标准之一，制定这个标准的原因是为X.500目录检索的一部分提供安全目录检索服务（这一部分我也没深入了解过，理解为保障网络安全就行）。

RFC2459：首先，RFC(Request For Comments)-意即“请求注解”，包含了关于Internet的几乎所有重要的文字资料。当一些标准制定机构对某方面的标准感兴趣时，他们会在RFC中发布相关文档，向全网络征求意见。很多重要的网络标准的制定都是由RFC文档的形式开始的，但是最后，并不是每一份都会成为标准规范，但是作为参考，RFC是非常非常重要的参考资料，而RFC2459的一些部分介绍了关于证书的知识。

总结一下，就是证书的标准是由ITU-T定义的，这个标准叫做X.509,在X.509中，证书的模型与定义都是基于ASN.1的标准，而ASN.1是一种通用的信息定义规范。本文在介绍X.509时，参考了文档RFC2459。

上面的描述都是基于本人的理解，可能有错误的地方，希望大家指出。

### ASN.1的组成与作用
首先，上面说了，ASN.1就是一种传输信息的标准，将二进制数据结构化，便于各种不同系统间的数据传输。

ASN.1是基于8位编码的TLV三元组<Type,Length,Value>的。这是什么意思呢？
```
<Type,Length,Value>
<类型,长度，值>
```
当一个系统读到一串数据后，就会判断，这是个什么类型的数据？有多长？然后读这个数据的值。其中类型是定长的，即8位（1字节），顺带说一句，ASN.1提供的数据类型有32中，其中包含了一些保留值，已经够用了。而长度与值都是不定长的。毕竟数据我们默认是没有限制的。

计算机识别的数据都是以0/1来记录的，一串下面这样一串01字符串，共3个字节，24个bit位，可能在不同的系统中代表了不同的含义。
```
  二进制：00000010 00000001 00000010
十六进制：   0   2    0   1    0   2
  十进制：       2        1        2
```
但是用TLV来读，就会知道，首先第一个字节表示类型，02代表的就是INTEGER，这是一个整数；第二个字节为长度，长度为1个字节，表示往后的一个字节为数据的内容；第三个字节表示值，因此这些数据代表了一个值为2的整数。

有人可能会问，对于一个简单的整数2，计算机中直接用10（二进制）就可以表示了，总共2个bit，但是上面却使用了32个bit，岂不是效率极低？但是ASN.1真正的作用并不是用来传递简单数据，而是使用了层层嵌套用于不同系统间，传输复杂的对象，使得数据具有极好的兼容性。

### 一张X509证书示例
下面这是一张X509标准的证书在Windows系统中打开的截图，系统从这张证书获取到了证书的序列号、有效期、颁发者等等信息。同样的，不管是windows还是linux或者是OS系统，都可以拿到一样的结果。
![certInfo][certInfo]

#### 证书结构
显然，证书文件的数据结构一定有某种规范，才能让所有的操作系统、浏览器或是程序能够识别。那现在我们来看看证书文件的定义（RFC2459 Section4.1）。	
```
Certificate  ::=  SEQUENCE  {
	tbsCertificate       TBSCertificate,      
	signatureAlgorithm   AlgorithmIdentifier,
	signatureValue       BIT STRING
	}
```
它的含义是一个证书对象中包含了3个类型的值，分别是:
* tbsCertificate（To Be Signed Certificate）待签名证书
* signatureAlgorithm 签名算法
* signatureValue 签名值

使用TLV的嵌套形式就是：
```
证书-
    -待签名证书-
              -版本号
              -序列号
              -算法标识
              -签发者
              -......
    -签名算法
    -签名值
```
ASN.1的形式
```
<证书的类型,证书数据的长度,
	<tbsCertificate的类型，tbsCertificate长度，<数据> >
	<signatureAlgorithm的类型，signatureAlgorithm长度，<数据> >
	<signatureValue的类型-比特字符串，signatureValue的长度,<数据> >
>
```

下面这张图是一张X509证书，通过ASN.1解析工具（asn1dump）解析出来的结果。
![certBer][certBer]
左边的列表是这个证书的数据嵌套，可以看到Certificate是SEQUENCE类型，包含了SEQUENCE类型的tbsCertificate和signatureAlgorithm，最后的BIT STRING是证书的签名值。

在右边是具体的数据，以16进制表示。绿蓝黄不同的颜色分别代表了类型、长度、值三元组。我现在选中的是左边的Certificate对象，类型30（Sequence），长度861字节，后面是它的值。

再选中左边的signatureAlgorithm这个对象，看看右边的数据，类型30（Sequence），长度为0D，也就是十进制的13，说明后面的十三个字节是它的值。
![asn1dump][asn1dump]

对比一下两个数据的区别，发现了什么？类型都是一个字节，但是对于Certificate，他的长度有3个字节，但是signatureAlgorithm的长度却只有一个字节。为什么？
![length][length]

思考一下，由于我们没有限制数据的长度，那么描述数据长度的字段也应该是无限长的。如果只用一个字节来表示长度，那么8bit最多只能表示0-255的数据，也就是说长度超过255字节的值就没办法了表示了。

#### 签名算法与签名值
签名算法的定义如下（RFC2459 Section4.1.1.2）：
```
AlgorithmIdentifier  ::=  SEQUENCE  {
        algorithm        OBJECT IDENTIFIER,
        parameters       ANY DEFINED BY algorithm OPTIONAL  }
```
![SigOID][SigOID]
在asn1dump中解析出的签名算法OID是一个11字节的数据:
```
06 09 2a 86 48 86 f7 0d 01 01 05
```
通过程序（JAVA自带的org.ietf.jgss.Oid包），我们可以解析出这个数据的OID为1.2.840.113549.1.1.5

在www.oid-info.com上查询这个OID的结果，它代表的是sha1-with-rsa-signature，即使用了SHA1摘要算法的RSA签名算法。

![OIDquery][OIDquery]






[certInfo]: /assets/pic/2018-05-12/certInfo.png
[certBer]: /assets/pic/2018-05-12/certBer.png
[asn1dump]: /assets/pic/2018-05-12/asn1dump.png
[length]: /assets/pic/2018-05-12/length.png
[OIDquery]: /assets/pic/2018-05-12/OIDquery.png
[SigOID]: /assets/pic/2018-05-12/SigOID.png