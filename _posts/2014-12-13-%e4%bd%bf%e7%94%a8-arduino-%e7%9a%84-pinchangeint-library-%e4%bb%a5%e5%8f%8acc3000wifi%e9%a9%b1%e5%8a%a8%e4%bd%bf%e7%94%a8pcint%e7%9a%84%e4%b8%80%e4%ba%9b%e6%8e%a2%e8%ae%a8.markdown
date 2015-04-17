---
layout: post
status: publish
published: true
title: 使用 Arduino 的 PinChangeInt Library 以及cc3000wifi驱动使用pcint的一些探讨
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 124
wordpress_url: http://www.bmysoft.org/?p=124
date: !binary |-
  MjAxNC0xMi0xMyAyMDowODoyMCArMDgwMA==
date_gmt: !binary |-
  MjAxNC0xMi0xMyAxMjowODoyMCArMDgwMA==
categories:
- Adruino
- 3D打印
tags: []
comments: []
---
<p>这周一直在MKS的mega2560上移植CC3000的wifi驱动，与其说是移植，不如说是改写吧，CC3000需要SPI总线，int， dig 这几个引脚才能正常工作。可是我们的板子上没有多余的int中断线了。没办法，自己动手，丰衣足食。</p>
<p>其实arduino上同时提供了两种中断：&ldquo;external&rdquo;, and &ldquo;pin change&rdquo;，只有两个external中断，在uno里，他们被映射到了arduino pin2 pin3。但同时还有另一种中断，pin change int。</p>
<p>external int可以在引脚的RISING or FALLING时被触发，或者是更加低层的模式，而PCINT顾名思义，当pin chage时候被触发也可以接受RISING or FALLING，所以打算使用pcint代替int来移植cc3000到我们的板子上</p>
<pre>引用一段关于中断的介绍：<br />
●中断源<br />
中断源是指能够向单片机发出中断请求信号的部件和设备。对于单片机来讲，往往存在多个中断源。中断源一般可分为内部中断源和外部中断源。<br />
单片机内部集成的许多功能模块，如定时器、串行通讯口、模&#47;数转换器等，它们在正常工作时往往无需CPU参与，而当处于某种状态或达到某个规定值需要程序 控制时，会通过发出中断请求信号通知CPU。这一类的中断源位于单片机内部，称作内部中断源。内部中断源在中断条件成立时，一般通过片内硬件会自动产生中 断请求信号，无须用户介入，使用方便。内部中断是CPU管理片内资源的一种高效的途径。<br />
系统中的外部设备也可以用作中断源，这时要求它们能够产生一个中断信号（通常是高（低）电平或者电平跳变的上升（下降）沿），送到单片机的外部中断请求引 脚供CPU检测。这些中断源位于单片机外部，称为外部中断源。通常用作外部中断源的有输入输出设备、控制对象、以及故障源等。<&#47;pre><br />
下面说说cc3000的驱动那部分吧：</p>
<p>cc3000需要一个int引脚，但是板子上没有多余的int引脚了，于是我使用了Arduino 的 PinChangeInt Library。</p>
<p>我使用了Adafruit_CC3000的库文件。</p>
<p>代码里判断了中断针脚，我们在加一个pcint的测试</p>
<pre>#elif defined(__AVR_ATmega1281__) || defined(__AVR_ATmega2561__) || defined(__AVR_ATmega2560__) || defined(__AVR_ATmega1280__)<br />
 2, 0,<br />
 3, 1,<br />
 21, 2,<br />
 20, 3,<br />
 19, 4,<br />
 18, 5,<br />
 11, 11,&#47;&#47;PCINT Test<&#47;pre></p>
<pre>将ccspi里面的所有deachInterrupt和attachInterrupt改为<br />
PCintPort:detachInterrupt(g_IRQnum);<br />
PCintPort:attachInterrupt(g_IRQnum, SPI_IRQ, FALLING);</p>
<p>开起了全部调试后，还是卡在了这个循环。<br />
 if (sSpiInformation.ulSpiState == eSPI_STATE_POWERUP)<br />
 {<br />
 while (sSpiInformation.ulSpiState != eSPI_STATE_INITIALIZED);<br />
 }</p>
<p>eSPI_STATE_INITIALIZED 在中断处理函数里被修改了，所以说中断还是没有执行，卡在了等待中断上，这个问题还没有解决。不知道这是否是一个内部中断呢。<&#47;pre></p>
