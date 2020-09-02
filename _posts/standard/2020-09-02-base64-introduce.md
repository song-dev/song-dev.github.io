---
layout: post
title: "Base64 介绍"
subtitle: ' Base64 是一种基于 64 个可打印字符来表示二进制数据的表示方法 '
author: "Song"
header-style: text
tags:
  - base64
---

## 概念

Base64 是一种基于 64 个可打印字符来表示二进制数据的表示方法

### 说明

由于 log64=6，所以每 6 个比特为一个单元，对应某个可打印字符。3 个字节相当于 24 个比特，对应于 4 个 Base64 单元，即 3 个字节可由 4 个可打印字符来表示。它可用来作为电子邮件的传输编码。在 Base64 中的可打印字符包括字母 A-Z、a-z、数字 0-9，这样共有 62 个字符，此外两个可打印符号在不同的系统中而不同。

> Wiki: https://zh.wikipedia.org/wiki/Base64

### 标准版 Basic

RFC 4648 描述了一种称为 Basic 的 Base64 变体。此变体使用 RFC 4648 和 RFC 2045 的表中所示的 Base64 字母表进行编码和解码。编码器将编码的输出流视为一行； 没有输出行分隔符。解码器拒绝包含Base64字母表之外的字符的编码。

### MIME

RFC 2045 描述了一种称为 MIME 的 Base64 变体。此变体使用 RFC 2045 的中提供的 Base64 字母表进行编码和解码。编码的输出流被组织成不超过 `76` 个字符的行； 每行（最后一行除外）通过行分隔符与下一行分隔。解码期间将忽略Base64字母表中未找到的所有行分隔符或其他字符。

### URL and Filename Safe

RFC 4648 描述了一种称为 URL 和文件名安全的 Base64 变体。此变体使用 RFC 4648 的表中提供的 Base64 字母表进行编码和解码。字母表与前面显示的字母相同，只是 - 替换 + 和 _ 替换/ 。不输出行分隔符。解码器拒绝包含Base64字母表之外的字符的编码。 Base64编码在冗长的二进制数据和 HTTP GET 请求的上下文中很有用。我们的想法是对这些数据进行编码，然后将其附加到HTTP GET URL。如果使用 Basic 或 MIME 变体，则编码数据中的任何 + 或 / 字符必须被 URL 编码为十六进制序列（ + 变为 %2B 和 / 变为 %2F ）。生成的 URL 字符串会稍长一些。通过更换 + 同 - 和 / 同 _ ，URL和文件名安全消除了对URL编码器/解码器（和它们的编码值的长度影响）的需要。此外，当编码数据用于文件名时，此变体很有用，因为 Unix 和 Windows 文件名不能包含  / 。

## 算法说明

Base 64 Encoding 的编码原理是将每三个字节（byte）转换为四个字符，每个字符占6 bit。

<img src="https://song-dev.github.io/img/in-post/post-base64/base64_1.png" alt="jvm-object-create-process" style="zoom:80%;" />

6 bit一共有64种组合方式，也就是说该编码共需要使用至少64种字符（后面我们还会介绍一个特殊字符 =）。Base 64 Encoding使用了从 A 到 Z、a 到 z、0 到 9、以及 + 和 / 这些字符（即[A-Za-z0-9+/]）。

<img src="https://song-dev.github.io/img/in-post/post-base64/base64_1.png" alt="jvm-object-create-process" style="zoom:80%;" />

假设我们有三个字节的数据，byte[] {1, 2, 3}，用二进制表示为：
00000001 | 00000010 | 00000011
依据上面的原理，使用Base 64 Encoding编码后结果应该为：
000000 | 010000 | 001000 | 000011
转换为十进制为 0 | 16 | 8 | 3，对照上面的表，编码后的文本为 AQID
既然Base 64 Encoding将每三个字节转换为四个字符，那如果一幅图片的字节数不能被3整除该怎么办？
如果剩余一个字节，该字节同样被转换为四个字符。第一个6 bit 转换成一个字符，接下来2 bit 转换成一个字符（注意这里是向右添加0），最后添加两个=字符。

<img src="https://song-dev.github.io/img/in-post/post-base64/base64_1.png" alt="jvm-object-create-process" style="zoom:80%;" />

假设我们有四个字节的数据，byte[] {1, 2, 3, 4}，用二进制表示为：
00000001 | 00000010 | 00000011 | 00000100
依据上面的原理，使用Base 64 Encoding编码后结果应该为：
000000 | 010000 | 001000 | 000011 | 000001 | 000000
转换为十进制为 0 | 16 | 8 | 3 | 1 | 0，对照上面的表，编码后的文本为 AQIDBA==
如果不能被3整除，而余下两个字节，编码方式类似剩余一个字节，同样是转换成四个字符，最后一个字符用=。

<img src="https://song-dev.github.io/img/in-post/post-base64/base64_1.png" alt="jvm-object-create-process" style="zoom:80%;" />

假设我们有五个字节的数据，byte[] {1, 2, 3, 4, 5}，用二进制表示为：
00000001 | 00000010 | 00000011 | 00000100 | 00000101
依据上面的原理，使用Base 64 Encoding编码后结果应该为：
000000 | 010000 | 001000 | 000011 | 000001 | 000000 | 010100
转换为十进制为 0 | 16 | 8 | 3 | 1 | 0 | 20，对照上面的表，编码后的文本为 AQIDBAU=

## 实现代码

> 见 GitHub 仓库：https://github.com/song-dev/c-study
