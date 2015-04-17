---
layout: post
status: publish
published: true
title: Build Both shared library and executable
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 156
wordpress_url: http://libdll.so/?p=156
date: !binary |-
  MjAxNS0wMi0wMiAxNTo1Mzo1NSArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wMi0wMiAwNzo1Mzo1NSArMDgwMA==
categories:
- C&#47;CPP
- Linux
tags: []
comments:
- id: 52
  author: WHR
  author_email: msl0000023508@gmail.com
  author_url: http://127.0.0.1/
  date: !binary |-
    MjAxNS0wMi0wMiAxNjozMDozNyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMi0wMiAwODozMDozNyArMDgwMA==
  content: libtoolbox.so.1.3
- id: 53
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMi0wMiAxNjozMTo0NyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMi0wMiAwODozMTo0NyArMDgwMA==
  content: Wanting master branch push to github.
- id: 59
  author: WHR
  author_email: msl0000023508@gmail.com
  author_url: http://kernel-panic.tk
  date: !binary |-
    MjAxNS0wMi0wNiAxMzo0OToxNyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMi0wNiAwNTo0OToxNyArMDgwMA==
  content: ! "The master branch is pushed\r\nWaiting for branch merge"
- id: 74
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMy0xNSAyMzoxODo1MCArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xNSAxNToxODo1MCArMDgwMA==
  content: 测试回复
- id: 75
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMy0xNSAyMzoyOToxMyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xNSAxNToyOToxMyArMDgwMA==
  content: Test again
- id: 76
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMy0xNSAyMzoyOTo1NyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMy0xNSAxNToyOTo1NyArMDgwMA==
  content: ip看起来是没问题了
---
<p>简单的实现方式，在makefile里加入如下方式</p>
<p>就可以编译出可执行可以执行的SHARED LIBRARY了</p>
<pre class=""># Both shared library and executable<br />
# Note: Option --pie -Wl,-E replaces --shared<br />
ifdef SHARED_OBJECT<br />
CFLAGS += -fPIC<br />
# LDFLAGS += --shared<br />
LDFLAGS += --pie -Wl,-E<br />
OUTFILE = libtoolbox.so<br />
endif<&#47;pre></p>
