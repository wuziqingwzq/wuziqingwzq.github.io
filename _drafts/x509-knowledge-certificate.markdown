---
layout: post
title:  "X509学习记录（二）：证书格式"
date:   2017-12-27 00:00:00 +0800
categories: CA
tags: X509 CA Certificate RFC
---
在之前我们已经了解过了ASN.1的基本语法（参见[X509学习记录（一）：ASN.1概述][x509knoledge1]），可以开始通过相应的RFC文档来学习证书的结构了。

### 相关标准
公钥加密标准（Public Key Cryptography Standards, PKCS），此一标准的设计与发布皆由RSA信息安全公司所制定。有一系列的规范，从PKCS#1到PKCS#15，具体列表可以参考[公钥加密标准的中文WIKI][PKCS]。

其中PKCS#1与PKCS#7是CA机构使用较多的部分，其对应的文档分别是RFC8017与RFC2315。

X509是ITU-T（国际电信联盟电信标准化部门）为了公开密钥基础建设（PKI，Public Key Infrastructure）而设立的标准，对应的文档是RFC5280。

### RFC5280
在RFC5280中，主要描述了X509V3版本的证书格式与X509V2版本的CRL废止列表。

在该文档中
* 第一节是文档概述
* 第二节描述了文档的范围
* 第三节描述了与其他一些相关规范的关系
* 第四节描述了证书结构
* 第五节描述了CRL列表机构
* …………

### 证书结构
#### 基本证书域

```
   Certificate  ::=  SEQUENCE  {
        tbsCertificate       TBSCertificate,
        signatureAlgorithm   AlgorithmIdentifier,
        signatureValue       BIT STRING  }

```
证书Certificate是由三部分组成
* 待签名证书（to be signed certificate）tbsCertificate
* 签名算法signatureAlgorithm
* 签名值signatureValue，该项是bit字符串类型的

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
                             -- If present, version MUST be v2 or v3
        subjectUniqueID [2]  IMPLICIT UniqueIdentifier OPTIONAL,
                             -- If present, version MUST be v2 or v3
        extensions      [3]  EXPLICIT Extensions OPTIONAL
                             -- If present, version MUST be v3
        }

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
        extnValue   OCTET STRING
                    -- contains the DER encoding of an ASN.1 value
                    -- corresponding to the extension type identified
                    -- by extnID
        }
```







[x509knoledge1]: /ca/2017/12/27/x509-knowledge-asn1.html
[PKCS]: https://zh.wikipedia.org/wiki/%E5%85%AC%E9%92%A5%E5%AF%86%E7%A0%81%E5%AD%A6%E6%A0%87%E5%87%86