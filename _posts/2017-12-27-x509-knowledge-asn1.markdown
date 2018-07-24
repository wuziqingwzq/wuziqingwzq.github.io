---
layout: post
title:  "X.509学习记录（一）：ASN.1概述"
date:   2017-12-27 00:00:00 +0800
categories: CA
tags: X.509 CA Certificate ASN.1
---
在有关X509证书各类RFC文档中，都采用了ASN1的语法来描述证书结构，因此需要先了解一下ASN.1的语法规范，便于接下来的学习

### ASN.1语法规范
ASN.1抽象语法标记（Abstract Syntax Notation One） ASN.1是一种 ISO/ITU-T 标准，描述了一种对数据进行表示、编码、传输和解码的数据格式。

在接触到的很多接口说明文档中，也采用了这种格式来描述。

ASN1含有一套编码规则，包括了：
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

### ASN1基本语法规则

1）在ASN1中，使用了巴克斯范式，有如下规则：

* 在双引号中的字("word")代表着这些字符本身。而double_quote用来代表双引号。

* 在双引号外的字（有可能有下划线）代表着语法部分。

* 尖括号(< > )内包含的为必选项。

* 方括号([ ] )内包含的为可选项。

* 大括号({ } )内包含的为可重复0至无数次的项。

* 竖线```(| )```表示在其左右两边任选一项，相当于"OR"的意思。

* ::=是“被定义为”的意思。

以下是使用巴克斯范式来定义的JAVA for循环的实例：
```
　　FOR_STATEMENT::=

　　"for""(" ( variable_declaration |

　　(expression ";" ) | ";" )

　　[expression ] ";"

　　[expression ]

　　")"statement
```

2）在ASN.1中，符号的定义没有先后次序：只要能够找到该符号的定义即可。例如在下面的例子中，我先定义了Report，其中包含了Bibliography，然后在下面定义了目录Bibliography，顺序反过来也是可以的。

```
Report ::= SEQUENCE {
author OCTET STRING,
title OCTET STRING,
body OCTET STRING,
biblio Bibliography
}
Bibliography ::= SEQUENCE{
	catalog OCTET STRING
}
```

3）所有的标识符、参考、关键字都要以一个字母开头，后接字母(大、小写都可以)、数字或者连字符“-”(但不能以连字符“-”结尾，也不能连续出现两个连字符)，不能出现下划线“_”。这儿需要注意，通常JAVA中定义常用到下划线，而ASN1中不行。

4）关键字一般都是全部大写。

5）在标识符中，只有类型和模块名字是以大写字母开头的，其它标识符都是以小写字母开头。

6）ASN.1中实数实际定义为三个整数：尾数、基数和指数。没有小数表示方式。

7）ASN.1不对空格、制表符、换行符和注释做翻译。但是在定义符号（或者分配符号Assignment）“::=”中不能有分隔符。

### ASN1 数据类型
ASN1中的数据是由基本类型与组合类型构成的
#### 基本数据类型
1. NULL：只包含一个值NULL，用于传送一个报告或者作为CHOICE类型中某些值

2. INTEGER：全部整数（包括正数和负数）

3. REAL：实数，表示浮点数

4. ENUMERATED：标识符的枚举（实例状态机的状态）

5. BITSTRING：比特串

6. OCTETSTRING：字节串

7. OBJECT IDENTIFIER,RELATIVE-OID：一个实体的标识符，它在一个全世界范围树状结构中注册

8. EXTERNAL,EMBEDDED PDV：表示层上下文交换类型

9. …String(除了BITSTRING、OCTETSTRING外)：各种字符串，有NumericString、PrintableString、VisibleStirng、ISO64String、IA5String、TeletexStirng、T61String、VideotexString、GraphicString、GeneralString、UniversalString、BMPString和UTF8String

10. CHARACTERSTRING：允许为字符串协商一个明确的字符表

11. UTCTime,GeneralizedTime：日期

#### 组合数据类型

1. CHOICE：在类型中选择(相当于C中的联合)

2. SEQUENCE：由不同类型的值组成一个有序的结构(相当于C中的结构体)

3. SET：由不同类型的值组成一个无序的结构

4. SEQUENCEOF：由相同类型的值组成一个有序的结构(相当于C中的数组)

5. SETOF：由相同类型的值组成一个无序的结构




