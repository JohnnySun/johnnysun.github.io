---
layout: post
status: publish
published: true
title: uint8_t &#47; uint16_t &#47; uint32_t &#47;uint64_t 是什么数据类型(转)
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 46
wordpress_url: http://www.bmysoft.org/?p=46
date: !binary |-
  MjAxNC0wOC0zMSAyMTowMDozNSArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wOC0zMSAxMzowMDozNSArMDgwMA==
categories:
- C&#47;CPP
tags: []
comments: []
---
<p style="color: #000000;">uint8_t，uint16_t，uint32_t等都不是什么新的数据类型，它们只是使用typedef给类型起的别名，新瓶装老酒的把戏。不过，不要小看了typedef，它对于你代码的维护会有很好的作用。比如C中没有bool，于是在一个软件中，一些程序员使用int，一些程序员使用short，会比较混乱，最好就是用一个typedef来定义，如：<br />
typedef char bool;<&#47;p></p>
<p style="color: #000000;">一般来说，一个C的工程中一定要做一些这方面的工作，因为你会涉及到跨平台，不同的平台会有不同的字长，所以利用预编译和typedef可以让你最有效的维护你的代码。为了用户的方便，C99标准的C语言硬件为我们定义了这些类型，我们放心使用就可以了。<&#47;p></p>
<p style="color: #000000;">按照posix标准，一般整形对应的*_t类型为：<br />
1字节&nbsp;&nbsp;&nbsp;&nbsp; uint8_t<br />
2字节&nbsp;&nbsp;&nbsp;&nbsp; uint16_t<br />
4字节&nbsp;&nbsp;&nbsp;&nbsp; uint32_t<br />
8字节&nbsp;&nbsp;&nbsp;&nbsp; uint64_t<&#47;p></p>
<p style="color: #000000;">附：C99标准中inttypes.h的内容<br />
00001 &#47;*<br />
00002&nbsp;&nbsp;&nbsp; inttypes.h<br />
00003<br />
00004&nbsp;&nbsp;&nbsp; Contributors:<br />
00005&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Createdby Marek Michalkiewicz <marekm@linux.org.pl><br />
00006<br />
00007&nbsp;&nbsp;&nbsp; THISSOFTWARE IS NOT COPYRIGHTED<br />
00008<br />
00009&nbsp;&nbsp;&nbsp; Thissource code is offered for use in the public domain.&nbsp; You may<br />
00010&nbsp;&nbsp;&nbsp; use,modify or distribute it freely.<br />
00011<br />
00012&nbsp;&nbsp;&nbsp; Thiscode is distributed in the hope that it will be useful, but<br />
00013&nbsp;&nbsp;&nbsp; WITHOUTANY WARRANTY.&nbsp; ALLWARRANTIES, EXPRESS OR IMPLIED ARE HEREBY<br />
00014&nbsp;&nbsp;&nbsp; DISCLAIMED.&nbsp; This includes but is not limited towarranties of<br />
00015&nbsp;&nbsp;&nbsp; MERCHANTABILITYor FITNESS FOR A PARTICULAR PURPOSE.<br />
00016&nbsp; *&#47;<br />
00017<br />
00018 #ifndef __INTTYPES_H_<br />
00019 #define __INTTYPES_H_<br />
00020<br />
00021 &#47;* Use [u]intN_t if you need exactly N bits.<br />
00022&nbsp;&nbsp;&nbsp; XXX- doesn't handle the -mint8 option.&nbsp; *&#47;<br />
00023<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a0">00024<&#47;a>&nbsp;typedefsigned char int8_t;<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a1">00025<&#47;a>&nbsp;typedefunsigned char uint8_t;<br />
00026<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a2">00027<&#47;a>&nbsp;typedefint int16_t;<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a3">00028<&#47;a>&nbsp;typedefunsigned int uint16_t;<br />
00029<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a4">00030<&#47;a>&nbsp;typedeflong int32_t;<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a5">00031<&#47;a>&nbsp;typedefunsigned long uint32_t;<br />
00032<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a6">00033<&#47;a>&nbsp;typedeflong long int64_t;<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a7">00034<&#47;a>&nbsp;typedefunsigned long long uint64_t;<br />
00035<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a8">00036<&#47;a>&nbsp;typedefint16_t intptr_t;<br />
<a style="color: #444444;" href="http:&#47;&#47;ccrma.stanford.edu&#47;courses&#47;250a-fall-2002&#47;docs&#47;avrgcc&#47;inttypes_8h.html#a9">00037<&#47;a>&nbsp;typedefuint16_t uintptr_t;<br />
00038<br />
00039 #endif<&#47;p></p>
