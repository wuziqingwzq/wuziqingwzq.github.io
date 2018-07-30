---
layout: post
title:  "X.509学习记录（三）：X.509证书解析"
date:   2018-07-24 17:00:00 +0800
categories: CA
tags: X.509 CA Certificate ASN.1
---
## 关于ASN1的一点补充
在RFC2459里面，我看到了一些证书的定义中包含了EXPLICIT这个标识，经过查询一些资料，我发现了这样一些更细的定义。

之前有提到过，ASN.1以TLV（Tag标签、Length长度、Value值）来表示数据，ASN.1格式中的第一个字节表示了类型，在ASN.1中，有预定义的三十多种类型（好像是32，但是没有找到相关文档）。32位只需要5个bit就够了，那么一个字节有8bit，有三个bit就被浪费掉了，肯定是漏掉了什么。

通常的TAG是一个字节，8bit，被分为了3段。 最高的两位bit[7]和bit[6]表示TagClass，表示这个标签的类；接下来的一位bit[5]是表示这个标签是基本类型（0）还是结构化类型（1），如果是结构化类型的话，后面会再跟一层TVL；

![TAG][TAG]

### Tag Class
标签有四种类型，分别是：UNIVERSAL，APPLICATION，context-specific和PRIVATE,现在APPLICATION（应用标签）和PRIVATE（私有标签）已经不建议使用了，所以我们只讲一下另外两个。

![TagClass][TagClass]

### UNIVERSAL 通用标签
在ASN.1中的Universal类型如下表：

|Tag Value|Tag类型|
| :-: | :-: |
| 1 |BOOLEAN                        |
| 2 | INTEGER                       |
| 3 | BIT STRING                    |
| 4 | OCTET STRING                  |
| 5 | NULL                          |
| 6 | OBJECT IDENTIFIER             |
| 7 | ObjectDescripion              |
| 8 | EXTERNAL,INSTANCE OF          |
| 9 | REAL                          |
| 10 | ENUMERATED                   |
| 11 | EMBEDDED PDV                 |
| 12 | UFT8String                   |
| 13 | RELATIVE-OID                 |
| 14 | 保留                         |
| 15 | 保留                         |
| 16 | SEQUENCE,SEQUENCE OF         |
| 17 | SET,SET OF                   |
| 18 | NumericString                |
| 19 | PrintableString              |
| 20 | TeletexString,T61String      |
| 21 | VideotexString               |
| 22 | IA5String                    |
| 23 | UTCTime                      |
| 24 | GeneralizedTime              |
| 25 | GraphicString                |
| 26 | VisibleString,ISO646String   |
| 27 | GeneralString                |
| 28 | UniversalString              |
| 29 | CHARACTER STRING             |
| 30 | BMPString                    |
| 31 | 保留                         |

### Context-specific 上下文专用标签
既然ASN.1的UNIVERSAL中已经定义了很多基本类型，比如各种集合、各种字符串等，那我们为什么还需要使用到Context-specific呢？

我是这样理解的，在某些定义中，会出现可选项。只使用Universal Tag的情况下无法解决混乱的问题。
```
  People  ::=  SEQUENCE  {
        name      UFT8String,
        age       INTEGER,
        gender    UFT8String,
        adress    UFT8String OPTIONAL,
        school    UFT8String OPTIONAL,
        hobby     UFT8String OPTIONAL      }
```
看一下上面这个定义，一个人包含了姓名、年龄、性别，还带有3个可选项，地址、学校和业余爱好。当你解析出来这样一个People信息的时候，第四项应该是什么呢？
```
SEQUENCE--
         -UFT8String  "Bob"
         -INTEGER     14
         -UFT8String  "male"
         -UFT8String  "ChangAn"
         -UFT8String  "Comic"
```
前三项我们可以对照定义来解析，是一个名字叫做Bob的12岁小男孩，但是第四项和第五项究竟是表示哪一个呢，因为地址、学校和业余爱好的类型相同，并且都是可选项，你并不知道发送者对Bob的定义。

但是如果更换成Context-specific就不同了，我们可以通过标记的序列号来区分它们。
```
  People  ::=  SEQUENCE  {
        name      UFT8String,
        age       INTEGER,
        gender    UFT8String,
        adress   [0] UFT8String OPTIONAL,
        school   [1] UFT8String OPTIONAL,
        hobby    [2] UFT8String OPTIONAL      }
```
同样的，收到的信息也有相应的变化，可以清楚的分辨对应项。
```
SEQUENCE--
         -UFT8String  "Bob"
         -INTEGER     14
         -UFT8String  "male"
         -Context[0]--
                     -UFT8String  "ChangAn"
         -Context[2]--
                     -UFT8String  "Comic"
```

***
## 回到X509证书
接着上次的总结，一张X509证书是由待签名证书、签名算法、签名值组成的。签名算法是一个OID对象，签名值是一个字符串。而证书主要的信息都是在待签名证书（to be signed certificate）中。

### TbsCertificate
待签名证书的结构如下
```
TBSCertificate  ::=  SEQUENCE  {
    version         [0]  EXPLICIT Version DEFAULT v1,
    serialNumber         CertificateSerialNumber,
    signature            AlgorithmIdentifier,
    issuer               Name,
    validity             Validity,
    subject              Name,
    subjectPublicKeyInfo SubjectPublicKeyInfo,
    issuerUniqueID  [1]  IMPLICIT UniqueIdentifier OPTIONAL,
                         -- If present, version shall be v2 or v3
    subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                         -- If present, version shall be v2 or v3
    extensions      [3]  EXPLICIT Extensions OPTIONAL
                         -- If present, version shall be v3        }
———————————————————————————————————————————————————————                         
Version  ::=  INTEGER  {  v1(0), v2(1), v3(2)  }

CertificateSerialNumber  ::=  INTEGER

Validity ::= SEQUENCE {
     notBefore      Time,
     notAfter       Time }

Time ::= CHOICE {
     utcTime        UTCTime,
     generalTime    GeneralizedTime }

UniqueIdentifier  ::=  BIT STRING

SubjectPublicKeyInfo  ::=  SEQUENCE  {
     algorithm            AlgorithmIdentifier,
     subjectPublicKey     BIT STRING  }

Extensions  ::=  SEQUENCE SIZE (1..MAX) OF Extension

Extension  ::=  SEQUENCE  {
     extnID      OBJECT IDENTIFIER,
     critical    BOOLEAN DEFAULT FALSE,
     extnValue   OCTET STRING  }
```
待签名证书中包含的主要信息有版本号、证书序列号、签名算法、签发者、有效期、使用者。

### 证书版本
证书版本是一个整数，现有的证书版本有三个
* V1版本：对应值为0，证书只有基本项时为V1
* V2版本：对应值为1，证书中除了基本型外，还包含了签发者ID与使用者ID
* V3版本：对应值为2，证书中包含扩展项

### 证书序列号
证书序列号是一串整数，对于每一张证书，证书序列号应该是唯一的。

### 签名算法
这里的签名算法和之前的签名算法必须对应，否则证书无法验证通过。

### 有效期
有效期是一个时间序列，包含签发时间与过期时间，可选的类型有UTCTime和GeneralizedTime两种。
* UTCTime的格式是yyMMddHHmmssZ
* GeneralizedTime的格式是yyyyMMddHHmmss.sZ
在2050年以后的证书时间必须要使用GeneralizedTime

Tips：GMT时间与UTC时间稍有不同

### 签发者和使用者
由于签发者和使用者的类型都是一样的，是一些DN项（Distinguish Name）的集合,所以放在一起来说。

常见的DN项有：

|属性类型名称|含义|简写|
| :-: | :-: | :-: |
|Common Name                   |通用名称        |CN  |
|Organizational Unit name      |机构单元名称    |OU  |
|Organization name             |机构名          |O   |
|Locality                      |地理位置        |L   |
|State or province name        |州/省名         |S   |
|Country                       |国名            |C   |


### 一些问题

1. 证书序列号是以十进制还是十六进制展示的？
2. 为什么证书日期与Windows中的差距一天？
3. 为什么一张签名证书里面有两个Signature字段？
4. 为什么证书指纹在系统中显示是属性而不是扩展项？

### 学习RFC的目的
在具体的工作中，我们可能很少有真正用到文档中的知识，但是通过对文档的学习，可以让我们养成在工作中深入探索细节的习惯。对细节了解得越多，在解决问题的时候才能考虑得越全面，从而提高工作和学习的效率。


[TAG]: /assets/pic/2018-07-25/TAG.png
[TagClass]: /assets/pic/2018-07-25/TagClass.png

