---
layout: post
title: "2018-8-3-AndroidP刘海适配指南"
categories: Android
---

*Google于前几天发布了AndroidP的DP5， 随着DP5的发布，API28的文档算是定型了，这里来探讨下AndroidP的刘海适配方法*

关于各大厂商Andord8.X的刘海适配这里不再多说，参考个大厂商文档即可

这里说下AndroidP适配和之前给8.x刘海适配的不同之处：
* 在8.x时 各大厂商判断刘海基本都是用的getSysprop 或者是反射，在获取刘海的时候不需要获取Window，但是在AndroidP中， 用户想获取刘海的信息，需要使用`getRootWindowInsets().getDisplayCutout()` 这就需要获取window
下面给出在AndoridP中判断设备是否是刘海屏的方法
```java
DisplayCutout cutout = attachedView.getRootWindowInsets().getDisplayCutout();
return cutout != null;
```
需要注意的是，
