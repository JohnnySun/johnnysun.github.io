---
layout: post
status: publish
published: true
title: linux socket c
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 48
wordpress_url: http://www.bmysoft.org/?p=48
date: !binary |-
  MjAxNC0wOS0wMSAxODo1NjowMCArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wOS0wMSAxMDo1NjowMCArMDgwMA==
categories:
- C&#47;CPP
tags: []
comments: []
---
<p>struct addrinfo {<br />
int ai_flags;<br />
int ai_family;<br />
int ai_socktype;<br />
int ai_protocol;<br />
socklen_t ai_addrlen;<br />
struct sockaddr *ai_addr;<br />
char *ai_canonname;<br />
struct addrinfo *ai_next;<br />
};</p>
<p>&nbsp;</p>
<p>int getaddrinfo(const char *node, const char *service,<br />
const struct addrinfo *hints,<br />
struct addrinfo **res);</p>
<p>int connect(int sockfd, const struct sockaddr *addr,<br />
socklen_t addrlen);</p>
