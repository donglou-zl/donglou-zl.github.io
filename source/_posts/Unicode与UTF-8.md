---
title: Unicode与UTF-8
date: 2017-10-21 19:31:14
tags:
	- 编程基础
---
简单来说，
**Unicode**：是ISO组织提出的UCS（Universal Multiple-Octet Coded Character Set），为所有字母和符号编码的字符集，即为每一个字符分配一个唯一的“ID”（学名码位，Code point）；
**UTF-8**：为解决Unicode在网络上传输的问题，面向传输的众多UTF（UCS Transfer Format）标准出现了，它是一种编码规则，每次传输8个位数据。<!--more-->

Unicode其实也是一种对ASCII编码（只包括英文字母、数字和一些字符）的扩展，如同GBK编码一般。Unicode采用两个字节，也就是16位来统一表示所有的字符。对于ASCII编码里的一个字节的“半角”字符，Unicode保持其编码不变，只是将长度由原来的8位扩展为16位（高8位添加0），因此这种方案在保存纯英文文本时会多浪费一倍的空间。

UTF-8是为传输而设计的编码规则，它是一种变长的编码方式，使用1~4个字节来表示一个字符，根据不同的字符而变化字节长度。当字符在ASCII码的范围时，就用一个字节表示。Unicode中一个中文字符占2个字节，而UTF-8中一个中文字符占3个字节。Unicode到UTF-8不是直接的对应，需要一些算法和规则来转换。

编码规则如下：

    Unicode                  UTF-8
    U+ 0000 ~ U+ 007F：    0xxxxxxx
    U+ 0080 ~ U+ 07FF：    110xxxxx 10xxxxxx
    U+ 0800 ~ U+ FFFF：    1110xxxx 10xxxxxx 10xxxxxx
    U+ 10000 ~ U+ 1FFFF：  11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

例如：“知”的码位是U+77E5，那么它的UTF-8编码字节序列为？

        7     7     E     5
        0111  0111  1110  0101
        0111   011111   100101（二进制的77E5）
    1110xxxx 10xxxxxx 10xxxxxx（模板，第三行）
    11100111 10011111 10100101（带入模板）
    E   7    9   F    A   5
故“知”的UTF-8的编码字节序列为：E79FA5(3个字节表示一个中文字符)