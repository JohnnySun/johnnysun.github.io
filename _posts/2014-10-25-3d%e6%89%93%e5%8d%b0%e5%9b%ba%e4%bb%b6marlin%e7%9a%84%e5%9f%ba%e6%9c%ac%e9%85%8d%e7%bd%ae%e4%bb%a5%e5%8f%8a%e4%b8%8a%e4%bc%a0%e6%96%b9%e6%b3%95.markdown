---
layout: post
status: publish
published: true
title: 3D打印固件Marlin的基本配置以及上传方法
author:
  display_name: JohnnySun
  login: JohnnySun
  email: bmy001@gmail.com
  url: http://libdll.so
author_login: JohnnySun
author_email: bmy001@gmail.com
author_url: http://libdll.so
wordpress_id: 71
wordpress_url: http://www.bmysoft.org/?p=71
date: !binary |-
  MjAxNC0xMC0yNSAyMTo0OTozNSArMDgwMA==
date_gmt: !binary |-
  MjAxNC0xMC0yNSAxMzo0OTozNSArMDgwMA==
categories:
- Adruino
- 3D打印
tags: []
comments: []
---
<p>使用的单片机：&nbsp;mega 2560</p>
<p>Marlin固件下载地址：&nbsp;https:&#47;&#47;github.com&#47;ErikZalm&#47;Marlin&#47;releases （在这里下载最新版本的Marlin， 个人不建议直接clone，在开发版本可能会出现bug）</p>
<p>Marlin固件的设置存在Configure.h 文件中，下面摘录网上对部分配置的解读和自己的一些解读</p>
<p><code>#define BAUDRATE 250000<&#47;code>这是配置串口波特率的，只有上位机波特率和固件波特率相同来能通讯成功，一定需要注意。当然也不能随便改，常见的波特率为：2400，9600，19200，38400，57600，115200，250000。在3d打印机中常用的是后3个。</p>
<p>关于波特率的概念可以参见我的博客里的另一篇文章：<a href="http:&#47;&#47;www.bmysoft.org&#47;?p=44" rel="bookmark">浅谈关于串口通信中的波特率<&#47;a></p>
<p><code>#define MOTHERBOARD 33<&#47;code><br />
这个参数是配置板子类型的，3d打印机主控板类型非常多，每个板子的io配置不尽相同，所以这个参数必须要跟你自己的板子类型相同，否则无法正常使用。我们定的板子是MKS Gen V1.1,因为是兼容Ramps1.4的，所以对应的配置应该为33（单打印头配置），和34（双打印头配置）。如果你使用的是其它板子，请参考旁边的注释并选择合适的配置。</p>
<p><code>#define TEMP_SENSOR_0 1<&#47;code><br />
<code>#define TEMP_SENSOR_BED 1<br />
<&#47;code>这两个参数分别配置温度传感器的类型。这是读取温度是否正常的重要参数，如果读取的温度不正常将不能工作甚至有很大的潜在危险（烧毁器件等）。配置为1说明两个都是100K ntc热敏电阻。如果你使用了其它温度传感器需要根据情况自行更改。</p>
<p><code>#define EXTRUDE_MINTEMP 170<br />
<&#47;code>这个参数是为了防止温度未达到而进行挤出操作时带来的潜在风险，如果你做其它3d打印机，比如有朋友做巧克力打印机，挤出温度只需要45度，那么这个参数需要配置为较低数值，比如40度。</p>
<p><code>const bool X_ENDSTOPS_INVERTING = true;<&#47;code><br />
<code>const bool Y_ENDSTOPS_INVERTING = true;<&#47;code><br />
<code>const bool Z_ENDSTOPS_INVERTING = true;<&#47;code><br />
这里的三个参数是配置3各轴的限位开关类型的，配置为true，限位开关默认状态输出为1，触发状态输出为0，也就是机械限位应该接常开端子。如果你接常闭端子，则将true改为false。</p>
<p><code>#define INVERT_X_DIR false<&#47;code><br />
<code>#define INVERT_Y_DIR true<br />
<&#47;code>这两个参数是比较容易错的。根据自己机械的类型不通，两个的配置不尽相同。但是原则就是要保证原点应该在打印平台的左下角（原点位置为[0,0]），或右上角（原点位置为[max,max]）。只有这样打印出来的模型才是正确的，否则会是某个轴的镜像而造成模型方位不对。参考下图坐标。<img src="http:&#47;&#47;ww4.sinaimg.cn&#47;mw690&#47;78d6d1a7gw1e8f2mnswbqj203j03o743.jpg" alt="" data-bd-imgshare-binded="1" &#47;><br />
<code>#define X_HOME_DIR -1<&#47;code><br />
<code>#define Y_HOME_DIR -1<&#47;code><br />
<code>#define Z_HOME_DIR -1<&#47;code><br />
如果原点位置为最小值参数为-1，如果原点位置为最大值配置为1.</p>
<p><code>#define X_MAX_POS 205<&#47;code><br />
<code>#define X_MIN_POS 0<&#47;code><br />
<code>#define Y_MAX_POS 205<&#47;code><br />
<code>#define Y_MIN_POS 0<&#47;code><br />
<code>#define Z_MAX_POS 200<&#47;code><br />
<code>#define Z_MIN_POS 0<&#47;code><br />
这几个参数是配置打印尺寸的重要参数，参考上面的坐标系图来填写，这里需要说明的是坐标原点并不是打印中心，真正的打印中心一般在[(x.max-x.min)&#47;2,(y.max-y.min)&#47;2]的位置。中心位置的坐标需要在后面的切片工具中使用到，打印中心坐标应该与这里的参数配置匹配，否则很可能会打印到平台以外。</p>
<p><code>#define HOMING_FEEDRATE {50*60, 50*60, 4*60, 0}<&#47;code><br />
配置回原点的速率，单位为毫米每分钟，如果你使用的是xy轴同步带传动，z轴螺杆传动，这个参数可以使用默认值。</p>
<p><code>#define DEFAULT_AXIS_STEPS_PER_UNIT {85.3333, 85.3333,2560,158.8308}<&#47;code><br />
这个参数是打印机打印尺寸是否正确的最重要参数，参数含义为运行1mm各轴所需要的脉冲数，分别对应x,y,z,e四轴。多数情况下这个数字都需要自己计算才可以。计算公式可以参考的文章<a href="http:&#47;&#47;forums.makerlab.me&#47;t&#47;5" target="_blank">3d打印机各轴脉冲数计算方法<&#47;a>。如果你不想自己计算可以用计算器：<a href="http:&#47;&#47;www.makerlab.me&#47;calc&#47;index.html" target="_blank" rel="nofollow">3d打印机脉冲数计算器<br />
<&#47;a><br />
至此，最常用的参数都已经配置完成，可以开始使用了。</p>
<p>下面介绍一下Marlin的编译方法：<br />
编译方法有两种：</p>
<ol>
<li>使用Arduino IDE 直接打开INO文件，选择board的类型为Arduino Mega 2560点击Verify校验代码
<p><img src="http:&#47;&#47;ww3.sinaimg.cn&#47;mw690&#47;78d6d1a7gw1e8f3zztbhej20dw0godid.jpg" alt="" data-bd-imgshare-binded="1" &#47;></p>
<p>再点击向右的箭头按钮来上传固件，如图</p>
<p><img src="http:&#47;&#47;ww2.sinaimg.cn&#47;mw690&#47;78d6d1a7gw1e8f400qju9j20dw0gomzp.jpg" alt="" data-bd-imgshare-binded="1" &#47;><&#47;li></p>
<li>第二种方法：进入目录执行ake<&#47;li><br />
<&#47;ol></p>
