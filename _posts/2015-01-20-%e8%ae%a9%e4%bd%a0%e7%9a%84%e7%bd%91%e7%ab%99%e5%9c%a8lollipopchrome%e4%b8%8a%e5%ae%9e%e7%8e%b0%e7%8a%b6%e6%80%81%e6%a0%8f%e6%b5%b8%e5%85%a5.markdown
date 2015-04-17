---
layout: post
status: publish
published: true
title: 让你的网站在lollipop+chrome上实现theme-color(状态栏浸入)
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 144
wordpress_url: http://libdll.so/?p=144
date: !binary |-
  MjAxNS0wMS0yMCAxNDowMTo0OCArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wMS0yMCAwNjowMTo0OCArMDgwMA==
categories:
- Other
tags: []
comments:
- id: 49
  author: 科技爱好者
  author_email: lxx19950227@163.com
  author_url: http://blog.lxx1.com
  date: !binary |-
    MjAxNS0wMi0wMiAxMToyMzo1MCArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMi0wMiAwMzoyMzo1MCArMDgwMA==
  content: 网站速度现在很快啊~
- id: 50
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMi0wMiAxNTo1NjoyMyArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMi0wMiAwNzo1NjoyMyArMDgwMA==
  content: 是啊，最近使用了360的骨骼字体cnd，快了很多
- id: 51
  author: JohnnySun
  author_email: bmy001@gmail.com
  author_url: http://libdll.so
  date: !binary |-
    MjAxNS0wMi0wMiAxNTo1ODoyMSArMDgwMA==
  date_gmt: !binary |-
    MjAxNS0wMi0wMiAwNzo1ODoyMSArMDgwMA==
  content: 交换个友链吗？
---
<p>在chrome39+版本的浏览器上，对最新的安卓萝莉炮（lollipop）多了一项新支持，theme-color(状态栏浸入)。很多人还不知道怎么用，前几天去逛v2ex时候发现v2ex已经在国内率先支持了（这个必须给个赞）</p>
<p>实现方法也很简单，在代码的head里加入</p>
<pre><meta name="theme-color" content="#db5945"><br />
<&#47;pre><br />
同时，还可以选择一个192x192px的png图片，chrome会把他当作任务切换的logo，代码如下</p>
<pre>
<link rel="icon" sizes="192x192" href="nice-highres.png"><&#47;pre></p>
<p>我们也希望越来越多的网站支持呀，那或许会带来很好的体验，webapp也是以后的大方向呢。</p>
<p>做好了效果还是很好的，我这几天就在我的网站上折腾了一番，现在已经很好的支持了，看看效果<br />
<a href="http:&#47;&#47;libdll.so&#47;wp-content&#47;uploads&#47;2015&#47;01&#47;Screenshot_2015-01-19-10-27-12.png"><img src="http:&#47;&#47;libdll.so&#47;wp-content&#47;uploads&#47;2015&#47;01&#47;Screenshot_2015-01-19-10-27-12.png" alt="Screenshot_2015-01-19-10-27-12" width="1080" height="1920" class="alignnone size-full wp-image-145" &#47;><&#47;a><br />
<a href="http:&#47;&#47;libdll.so&#47;wp-content&#47;uploads&#47;2015&#47;01&#47;Screenshot_2015-01-19-10-27-08.png"><img src="http:&#47;&#47;libdll.so&#47;wp-content&#47;uploads&#47;2015&#47;01&#47;Screenshot_2015-01-19-10-27-08.png" alt="Screenshot_2015-01-19-10-27-08" width="1080" height="1920" class="alignnone size-full wp-image-146" &#47;><&#47;a><br />
<a href="http:&#47;&#47;libdll.so&#47;wp-content&#47;uploads&#47;2015&#47;01&#47;Screenshot_2015-01-19-01-57-31.png"><img src="http:&#47;&#47;libdll.so&#47;wp-content&#47;uploads&#47;2015&#47;01&#47;Screenshot_2015-01-19-01-57-31.png" alt="Screenshot_2015-01-19-01-57-31" width="1080" height="1920" class="alignnone size-full wp-image-147" &#47;><&#47;a></p>
