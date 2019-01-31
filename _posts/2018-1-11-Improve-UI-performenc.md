---
title: 提升安卓App的UI流畅度
author: JohnnySun
tags: Android
categories: 'Android, UI'
date: 2019-1-10

---

最近在优化app整体的流畅度， 对这部分做一个总结和整理

1月31号更新一个想法
PS：最近在考虑将统计学或者机器学习应用于Res_layout的Inflate和View的初始化以及band的时间统计上，通过机器学习或者统计学的方式使用大量样本在每次初始化VH BindVH时计算耗时，然后通过延后bind的方式减少单帧内bind viewholder的数量，从而减少手机的峰值压力，通过这样的方法减少手机的卡顿情况发生，

## Android App绘制View的流程:

(XML)->ViewGroup Object -> Measure -> Layout - >  Draw

## Android Sys绘制View的流程:
![Imgur](https://i.imgur.com/aQS9u5R.png)

## 造成安卓卡顿的几大原因

 1. View的Inflate
 2. 绘制流程开销过大（这里分为初始化开销，和更新绘制的开销）
 3. 不必要的动画太多 （大量的位移以及大量改变界面的绘制还会同时造成gpu绘制的开销，也会造成耗时增加）
 4. 自定义View在OnAttachToWindow时做了过多的工作

复杂View的Inflate耗时  解决办法
1.  脱离主线程Inflate，主线程在Inflate时使用加载动画过度。
2.  减少View的嵌套层级 使用Constrainlayout 拉平层级

绘制流程的开销主要在于View的初始化开销过大，Layout，Measure，Draw流程过于复杂，以及ViewGroup层级嵌套问题

> 这里不介绍关于View初始化开销过大的原因，各种自定义View太多，这里我们下面会讲述两个解决这部分问题的思路

为什么层级的嵌套会影响性能：

1.  首先是在XML的层面，  因为层级嵌套的很多，在Parse XML的时候会相对于少层级或者单层级的布局多创建很多的ViewGroup，每多一个ViewGrop，而每一个ViewGourp创建都是有开销的
2.  其次是在Measure中，这里我们先说一下单一ViewGroup的Measure流程，ViewGroup在确定其中的所有子View的大小， margain后根据情况确定自己的大小，然后完成Measure的流程，但是在多层级嵌套的View中，ViewGroup会递归的调用子ViewGroup的Measure方法，  当层级嵌套非常多后，这里的开销就会变得异常的大。
3.  在Layout中的原因和在Measure中类似，这里就不再讲述一遍了。
4.  Draw这里引入一个新的概念，过度绘制，  系统会按照View的层级依次进行绘制，只要是View有背景色，这里都会进行一次绘制，刷新像素点。如果当前视图存在很多被盖住的View。系统也会依次把他们在屏幕上进行绘制。这里有个例外，如果背景是全透明的，那这部分的区域就不会被重新绘制。  绘制的时间开销也是影响流畅度的很大原因。


这里来说一下动画印象性能的问题：
这一点是平时最容易被忽略的，我以Fresco为例，我们的工程中在Fresco中封装了WebImageView，在这里我们为了实现在网络图片加载出来的时候由默认图渐变为网络图片，在封装中直接给WebImageView设置了渐变效果，但是我们的WebImageView被大量的应用在了ViewHolder中，而且这两年富媒体是大趋势，我们的App中WebImageView的使用率大大提升，由于Fresco的缓存机制，有很多图片都能够在缓存中命中。这样就出现了一个很严重的性能问题。当app在稍微快速滑动的时候，系统的性能很大程度上都被用于无用的，用户无法感知的渐变动画上了。每一个在ViewHolder的中的WebImageView在被重新bind的时候，基本都会换图片，然而不管怎么样，渐变动画基本在现在的逻辑上都会执行，而从缓存中取得的图片基本速度很快，用户感知不到，导致在屏幕下不可见bind的时候就已经开始在执行动画了， 而当用户网速很快的时候，快速滑动也没有关闭Fresco的图片加载管道，导致了大量的图片完成后更新Ui的动画。

Fresco会使用渐变的方式进行图片的更新，这会导致两个很大的问题，一个是由于我们大量的使用了Fresco进行图片加载，导致用户在滑动的时候会有大量的动画同时进行，大量的更新的UI回掉，在调试中表现出消耗了大量的CPU时间，这部分可能是fresco渐变实现的问题，这里我没有深入探究， 那我说下他导致的第二个问题，在大量做动画的同时，也导致了大量的GPU区域重绘由于渐变效果会更新图片覆盖的整个区域，在做渐变的同时GPU也会去重新刷新着部分绘制的区域。

下面我们来看该问题在Hisilicon Kirin 980上的表现。

![动画问题在Hisilicon Kirin 980上的表现](https://i.imgur.com/EKwI4Sw.png)

途中浅绿色的部分就是980上的在中速滑动时做动画的耗时，红色部分是GPU绘制耗时，由于这是跑在2k分辨率上的，而且目前来看980的cpu单核心性能应该是当年前千元机的2-3倍，那么实际上该问题在主流的千元机（这里之所以用千元机做标准是因为千元机出货量最大）造成的耗时应该在10ms左右，老机器可能更多。


这里我们换一个Snapdragon820的机器看一下Fresco动画在问答中的表现
![Imgur](https://i.imgur.com/8hk9pFG.png)

上图中我们首先加载好了viewholder，然后保证所有的WebimageVeiw都处于加载完成的状态，这时我们向上滑动，使屏幕最上部的上一个VH进行bind， 这时候出现了严重的性能问题，大量的动画在主线程的开销以及大量的OpenGL驱动开销（表现为红色） 

我们修改代码屏蔽掉不必要的Fresco动画，由于我已经封装好了WebImageView的异步请求外加仅网络图片回来启动过度动画，我们使用`WebImageView.setAsyncRequest(true)`开启该功能
同样的步骤，这回看看效果 
![Imgur](https://i.imgur.com/0GYQ7ow.png)


（这里还要补充下动画不关对性能的影响，和耗电的影响）
---


在优化的过程中，我们首先对导致卡顿的原因进行分析

这里我们使用Profile GPU Rendring工具

> In the enlarged image of the Profile GPU Rendering graph shown in figure 1, you can see the colored section, as displayed on Android 6.0 (API level 23).
>![](https://developer.android.com/images/tools/performance/profile-gpu-rendering/gettingstarted_image003.png)
>**Figure 1.** Enlarged Profile GPU Rendering graph.
The following are a few things to note about the output:
> -   For each visible application, the tool displays a graph.
> -   Each vertical bar along the horizontal axis represents a > -frame, and the height of each vertical bar represents the > -amount of time the frame took to render (in milliseconds).
> -   The horizontal green line represents 16 milliseconds. To achieve 60 frames per second, the vertical bar for each frame needs to stay below this line. Any time a bar surpasses this line, there may be pauses in the animations.
> -   The tool highlights frames that exceed the 16 millisecond threshold by making the corresponding bar wider and less transparent.
> -   Each bar has colored components that map to a stage in the rendering pipeline. The number of components vary depending on the API level of the device.
>
> The following table provides descriptions of each segment of a vertical bar in the profiler output when using a device running Android 6.0 and higher.
>
> ![](https://i.imgur.com/AATn87Y.png)

在网页上看着这些颜色差距还是很明显的，可是真正在手机里看，这些颜色根本分出清哪个是哪个，这里面我发现了一个小技巧，可以快速的分辨出是什么在耗时

 - 这些颜色都是按顺序排列的，每个颜色在直方图里一定会出现，所以我们大可直接去找直方图里最明显的颜色，然后去比对它所在的位置，就可以很轻松的找到对应的耗时是什么。

 使用上面的方法，可以很方便的的查出来app在何时发生了掉帧，以及掉帧的大致原因是什么， 这里我使用我们app的首页举个例子（测试设备为Snapdragon 835 Android P)：

![](https://i.imgur.com/fXmZjcI.jpg)
 
![](https://i.imgur.com/wq839si.jpg)

上面两张图中分别是第一次bindHolder和复用后的bind操作。
我们先来看第一次bindHolder的这种情况
 
 下图是OnCreateVH的耗时（PS： 这里由于调试影响性能， 这里的耗时仅供参考，但是也能反映出大致每个操作的整体耗时）
![](https://i.imgur.com/mixU6Eu.png)
再上图中我们发现，在create的操作中 Lottie和UserIcon初始化占用了大量的时间，这里可以考虑对整体UserIcon的初始化和Lottie的初始化进行异步处理

> Written with [StackEdit](https://stackedit.io/).



