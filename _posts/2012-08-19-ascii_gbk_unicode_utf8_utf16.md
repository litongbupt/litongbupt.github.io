---
layout: post
title: "ASCII,GBK,UTF8,UTF16"
description: "ASCII GBK UTF8 UTF16"
category: normal
tags: []
---
{% include JB/setup %}
    
GBK简体中文版和UTF8简体中文版有什么区别?    
ASCII(ISO-8859-1)是鼻祖，最简单的方式，字节高位为0    
GB2312、GBK、GB18030，这几个是中文编码方式，并向下兼容。GB2312包含7000多个汉字和字符，GBK包含21000多个，GB18030更厉害，到了27000多个。他们都是用2个字节来表示一个汉字。跟ascii是怎么区分的呢？如果高字节的高位为1（也就是高字节大于127），就表示是汉字，低字节并无明显特征。    
Unicode是统一编码，它建立了一个全世界统一的码表。世界上的所有文字，在这张码表中都是唯一的。    
UTF－8是Unicode的一种存储、传输方式。它将整个Unicode码表分为3部分。    
0000 - 007F 这部分是最初的ascii部分，按原始的存储方式，即0xxxxxxx。    
0080 - 07FF 这部分存储为110xxxxx 10xxxxxx    
0800 - FFFF 这部分存储为1110xxxx 10xxxxxx 10xxxxxx    
UTF－16是双字节存储，这就带来一个问题，即高低字节的顺序。两个字节有两种顺序，它们也用BOM来标明。分为大尾码和小尾码两种。大尾码的BOM是FEFF，小尾码的BOM是FFFE    