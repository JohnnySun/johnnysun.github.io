---
layout: post
status: publish
published: true
title: Socket operation on non-socket 错误的原因
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 50
wordpress_url: http://www.bmysoft.org/?p=50
date: !binary |-
  MjAxNC0wOS0wMSAyMTo0Mzo0OSArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wOS0wMSAxMzo0Mzo0OSArMDgwMA==
categories:
- 未分类
tags: []
comments: []
---
<p style="color: #232323;">&nbsp;出现Socket operation on non-socket 错误的原因是：<&#47;p></p>
<p style="color: #232323;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if(listenfd= socket(AF_INET,SOCK_STREAM, 0)==-1)<&#47;p></p>
<p style="color: #232323;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if(connfd=accept(listenfd,(struct sockaddr *)&amp;client_addr,(socklen_t *)&amp;sin_size)==-1)<&#47;p></p>
<p style="color: #232323;">&nbsp;&nbsp; 这两句中缺失了（）造成的。赋值符合优先级最低，导致listenfd和connfd在创建&#47;连接成功是为0，不成功时为1<&#47;p></p>
