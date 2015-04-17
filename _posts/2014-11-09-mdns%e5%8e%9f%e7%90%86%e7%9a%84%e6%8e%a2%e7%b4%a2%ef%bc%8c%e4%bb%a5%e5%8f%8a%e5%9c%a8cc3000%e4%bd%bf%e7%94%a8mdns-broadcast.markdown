---
layout: post
status: publish
published: true
title: MDNS原理的探索，以及在CC3000使用MDNS Broadcast
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 93
wordpress_url: http://www.bmysoft.org/?p=93
date: !binary |-
  MjAxNC0xMS0wOSAxNTo0NzozNiArMDgwMA==
date_gmt: !binary |-
  MjAxNC0xMS0wOSAwNzo0NzozNiArMDgwMA==
categories:
- C&#47;CPP
- Adruino
tags: []
comments: []
---
<p>关于CC3000进行MDNS Boardcast的方法进行进行了下基本的研究，Ti官方给出了一个API去进行MDNS广播</p>
<pre>Calling mdnsAdvertiser() with a specific device service name will cause the CC3000 to send one mDNS packet to a multicast address and port number.</p>
<p>mDNS port - 5353<br />
mDNS multicast address - 224.0.0.251</p>
<p>Other devices which listens to the same multicast IP and port can parse the packet and pull out all the CC3000 required information (device name, IP address...) .<br />
CC3000 Host Driver supplies one API to support basic mDNS implementation:<br />
 <span class="coMULTI">&#47;* mdnsEnabled - flag to enable&#47;disable the mDNS feature *&#47;<br />
<&#47;span> <span class="coMULTI">&#47;* deviceServiceName - the service name as part of the published canonical domain name *&#47;<br />
<&#47;span> <span class="coMULTI">&#47;* deviceServiceNameLength - the length of the service name *&#47;<br />
<&#47;span> <span class="coMULTI">&#47;* return On success, zero is returned, return SOC_ERROR if socket was not opened successfully, or if an error occurred. *&#47;<br />
<&#47;span>&nbsp;<br />
<span class="kw4">int<&#47;span> mdnsAdvertiser<span class="br0">(<&#47;span><span class="kw4">unsigned<&#47;span> <span class="kw4">short<&#47;span> mdnsEnabled<span class="sy0">,<&#47;span> <span class="kw4">char<&#47;span> <span class="sy0">*<&#47;span> deviceServiceName<span class="sy0">,<&#47;span> <span class="kw4">unsigned<&#47;span> <span class="kw4">short<&#47;span> deviceServiceNameLength<span class="br0">)<&#47;span><&#47;pre><br />
由于我们在MEGA2560上一直使用adafruit所提供的CC3000库文件，而adafruit也提供了CC3000的MDNS的Arduino的库文件，为了方便，我们直接使用了adafriut的库文件。</p>
<pre>库文件下载地址：https:&#47;&#47;github.com&#47;adafruit&#47;CC3000_MDNS<&#47;pre><br />
关键使用方法很简单:</p>
<ol>
<li>#include <Adafruit_CC3000.h><br />
#include <SPI.h><br />
#include <CC3000_MDNS.h><&#47;li></p>
<li>Adafruit_CC3000 cc3000 = Adafruit_CC3000(ADAFRUIT_CC3000_CS, ADAFRUIT_CC3000_IRQ, ADAFRUIT_CC3000_VBAT,<br />
SPI_CLOCK_DIV2);<&#47;li></p>
<li>mdns.begin("cc3kserver", cc3000)<&#47;li>
<li>mdns.update();<&#47;li><br />
<&#47;ol><br />
简单的这样的几步就可以发送mdns broadcast了。</p>
<p>其实发送mdns broadcast的原理非常简单，向224.0.0.251:5353使用socket发送一个response</p>
<p>关键代码原理如下</p>
<pre>uint8_t respHeader[] = { 0x00, 0x00, &#47;&#47; ID = 0<br />
 0x84, 0x00, &#47;&#47; Flags = response + authoritative answer<br />
 0x00, 0x00, &#47;&#47; Question count = 0<br />
 0x00, 0x01, &#47;&#47; Answer count = 1<br />
 0x00, 0x00, &#47;&#47; Name server records = 0<br />
 0x00, 0x01 &#47;&#47; Additional records = 1<br />
};<br />
uint8_taRecord[] = { 0x00, 0x01, &#47;&#47; Type = 1, A record&#47;IPV4 address<br />
 0x80, 0x01, &#47;&#47; Class = Internet, with cache flush bit<br />
 0x00, 0x00, 0x00, 0x00, &#47;&#47; TTL in seconds, to be filled in later<br />
 0x00, 0x04, &#47;&#47; Length of record<br />
 0x00, 0x00, 0x00, 0x00 &#47;&#47; IP address, to be filled in later<br />
 };<br />
uint8_tnsecRecord[] = { 0xC0, 0x0C, &#47;&#47; Name offset<br />
 0x00, 0x2F, &#47;&#47; Type = 47, NSEC (overloaded by MDNS)<br />
 0x80, 0x01, &#47;&#47; Class = Internet, with cache flush bit<br />
 0x00, 0x00, 0x00, 0x00, &#47;&#47; TTL in seconds, to be filled in later<br />
 0x00, 0x08, &#47;&#47; Length of record<br />
 0xC0, 0x0C, &#47;&#47; Next domain = offset to FQDN<br />
 0x00, &#47;&#47; Block number = 0<br />
 0x04, &#47;&#47; Length of bitmap = 4 bytes<br />
 0x40, 0x00, 0x00, 0x00 &#47;&#47; Bitmap value = Only first bit (A record&#47;IPV4) is set<br />
 };<br />
#####################################################<br />
#计算respons的长度<br />
#####################################################<br />
queryFQDNLen = _expectedLen - 4;<br />
_responseLen =HEADER_SIZE + queryFQDNLen + A_RECORD_SIZE + NSEC_RECORD_SIZE;<br />
#####################################################<br />
#拷贝之前的预格式化数据到_resopnse<br />
#####################################################<br />
memcpy(_response, respHeader, HEADER_SIZE);<br />
 memcpy(_response + HEADER_SIZE, _expected, queryFQDNLen);<br />
 uint8_t* records = _response + HEADER_SIZE + queryFQDNLen;<br />
 memcpy(records, aRecord, A_RECORD_SIZE);<br />
 memcpy(records + A_RECORD_SIZE, nsecRecord, NSEC_RECORD_SIZE);<br />
#####################################################<br />
# Add TTL to records.<br />
#####################################################<br />
 uint8_t ttl[4] = { (uint8_t)(ttlSeconds >> 24), (uint8_t)(ttlSeconds >> 16), (uint8_t)(ttlSeconds >> 8), (uint8_t)ttlSeconds };<br />
 memcpy(records + TTL_OFFSET, ttl, 4);<br />
 memcpy(records + A_RECORD_SIZE + 2 + TTL_OFFSET, ttl, 4);<br />
######################################################<br />
ipAddress = 获取CC3000 ip地址</p>
<p>records[IP_OFFSET] = (uint8_t)(ipAddress >> 24);<br />
 records[IP_OFFSET + 1] = (uint8_t)(ipAddress >> 16);<br />
 records[IP_OFFSET + 2] = (uint8_t)(ipAddress >> 8);<br />
 records[IP_OFFSET + 3] = (uint8_t) ipAddress;</p>
<p>#####################################################<br />
#最后通过socket向224.0.0.251:5353发送_response<br />
#####################################################<br />
#怎么创建socket这里就不阐述了<br />
send(_mdnsSocket, _response, _responseLen, 0);<&#47;pre><br />
&nbsp;</p>
