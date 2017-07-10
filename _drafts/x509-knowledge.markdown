---
layout: post
title:  "X509学习记录"
date:   2017-06-29 00:00:00 +0800
categories: CA
tags: X509 CA Certificate
---

### ANS.1语法规范
ASN.1抽象语法标记（Abstract Syntax Notation One） ASN.1是一种 ISO/ITU-T 标准，描述了一种对数据进行表示、编码、传输和解码的数据格式。

在NetoneX的说明文档中，普遍采用了这种格式来描述接口。

ANS含有一套编码规则，包括了：
- BER basic encoding rules
- DER distinguished encoding rules
- CER canonical encoding rules
- PER packed encoding rules
- XER XML encoding rules

ASN.1 定义了的数据类型：
- Universal 1-31

ASN.1 定义的数据结构类型
- 结构（SEQUENCE）
- 列表 (SEQUENCE OF)
- 类型选择(CHOICE)
- ..

语法三元组
- 实际语法
- 抽象语法
- 传输语法

ASN 定义语法

```
Report ::= SEQUENCE {
author OCTET STRING,
title OCTET STRING,
body OCTET STRING,
biblio Bibliography
}
```
