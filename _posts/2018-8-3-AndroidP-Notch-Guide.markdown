---
layout: post
title: "AndroidP刘海适配指南"
categories: Android
---

*Google于前几天发布了AndroidP的DP5， 随着DP5的发布，API28的文档算是定型了，这里来探讨下AndroidP的刘海适配方法*

关于各大厂商Andord8.X的刘海适配这里不再多说，参考个大厂商文档即可

主要还是说下AndroidP适配和之前给8.x刘海适配的不同之处：
* 在8.x时 各大厂商判断刘海基本都是用的getSysprop 或者是反射，在获取刘海的时候不需要获取Window，但是在AndroidP中， 用户想获取刘海的信息，需要使用`getRootWindowInsets().getDisplayCutout()` 这就需要获取window
下面给出在AndoridP中判断设备是否是刘海屏的方法
```java
DisplayCutout cutout = attachedView.getRootWindowInsets().getDisplayCutout();
return cutout != null;
```
需要注意的是，这里的attachedView不能在onCreate的时候随便传一个view，那时候view还没有Attach到window，拿到的RootWindowInsets是null，这里推荐使用OnApplyWindowInsetesListener方法，在应用Insetes时判断刘海状态并保存，示例如下
```java
if (StatusBarUtils.isAndroidM()) {
    getWindow().getDecorView().setOnApplyWindowInsetsListener( new View.OnApplyWindowInsetsListener() {
        @SuppressLint("NewApi")
        @Override
        public WindowInsets onApplyWindowInsets(View v, WindowInsets insets) {
            //用这里的View判断刘海状态
            LoginCommon.HAS_NOTCH = DeviceUtils.hasNotch(v) ? DeviceUtils.HAS_NOTCH : DeviceUtils.NO_NOTCH;
            getWindow().getDecorView().setOnApplyWindowInsetsListener(null);
            return v.onApplyWindowInsets(insets);
        }
    });
}
```

* AndroidP 的默认逻辑是LayoutFullScrren的WindowFlag可以侵入刘海，默认的状态和FullScrren的情况下，系统会禁止DecorView侵入刘海，但是有些时候我们有需要让FullScreen的情况下侵入刘海区域，这时候就需要设置`WindowManager.LayoutParams`的`layoutInDisplayCutoutMode`
```java 
// AndoridP 中定义了三种类型

// DEFAULT就是上面说的默认 LayoutFullScrren可以侵入，其余不会侵入刘海区域
public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT = 0;
// 这个状态是永远不会在刘海上绘制
public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER = 2;
//使用这个状态可以让LayoutFullScreen和FullScrren的时候在刘海区域绘制
public static final int LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES = 1;
```

这里给出一个修改绘制区域的例子 
```java
WindowManager.LayoutParams lp = act.getWindow().getAttributes();
lp.layoutInDisplayCutoutMode =
                        WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES;
act.getWindow().setAttributes(lp);
```

当然 如果你在启动页设置了默认背景，并且设置了Fullscreen，那么在启动的时候刘海会是一个大黑条，很难看，解决的方法是在StartAct的theme中加入`<item name="android:windowLayoutInDisplayCutoutMode">shortEdges</item>`
