---
title: FrameLayout让人困惑的wrap_content的Feature的研究
author: JohnnySun
tags: Android
categories: Android
date: '2018-11-18'

---

之前在用frameLayout的时候，踩过几次坑，简单来说，当framelayout里面是match_parent的时候，如果framelayout是wrap_content,则里面的view也会被当作wrapcontent处理

举个🌰：
```xml
<FrameLayout  
  android:layout_width="match_parent"  
  android:layout_height="wrap_content">  
 <TextView  
  android:layout_width="match_parent"  
  android:layout_height="match_parent"  
  android:gravity="center"  
  android:text="text"  
  android:background="@color/colorAccent"/>  
</FrameLayout>
```
我们这样写一个最简单的XML，那么他的表现是什么样呢？

如果最外层不是FrameLayout，而是RelativeLayout, 也就是我们的理想效果，那他应该是这样

![Imgur](https://i.imgur.com/X8wfc8e.png)

而实际上的效果呢？
是这样的
![Imgur](https://i.imgur.com/9lvkvYW.png)

而官方的描述看起来，应该就是第一张图的样子才对

> Child views are drawn in a stack, with the most recently added child on top. The size of the FrameLayout is the size of its largest child (plus padding), visible or not (if the FrameLayout's parent permits). Views that are [View.GONE](https://developer.android.com/reference/android/view/View.html#GONE) are used for sizing only if [setConsiderGoneChildrenWhenMeasuring()](https://developer.android.com/reference/android/widget/FrameLayout.html#setMeasureAllChildren(boolean)) is set to true.

为什么会有这样奇怪的Fature？ 踩了两次坑，今天我们来看看为啥FrameLayout会有这样的效果

---
## 问题探究
首先我们从FrameLayourt的`onMeasure()`入手
在FrameLayout的`onMeasure()`中， 最核心的是这段代码
```java
for (int i = 0; i < count; i++) {  
  final View child = getChildAt(i);  
  if (mMeasureAllChildren || child.getVisibility() != GONE) {  
  measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);  
  final LayoutParams lp = (LayoutParams) child.getLayoutParams();  
  maxWidth = Math.max(maxWidth,  
  child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin);  
  maxHeight = Math.max(maxHeight,  
  child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin);  
  childState = combineMeasuredStates(childState, child.getMeasuredState());  
  if (measureMatchParentChildren) {  
  if (lp.width == LayoutParams.MATCH_PARENT ||  
  lp.height == LayoutParams.MATCH_PARENT) {  
  mMatchParentChildren.add(child);  
 } } }}
```
这里面是通过`measureChildWithMargins(child, widthMeasureSpec,  0, heightMeasureSpec,  0);`这个方法来确定child的具体大小的， 而我们的FrameLayout的height由于是wrap_content， 所以heightMeasureSpec是`MeasureSpec.UNSPECIFIED`

知道了这些，下面让我们看看`measureChildWithMargins()`的实现方法
```java 
protected void measureChildWithMargins(View child,  
  int parentWidthMeasureSpec, int widthUsed,  
  int parentHeightMeasureSpec, int heightUsed) {  
  final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();  
  
  final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,  
  mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin  
  + widthUsed, lp.width);  
  final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,  
  mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin  
  + heightUsed, lp.height);  
  
  child.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
}
```
这里我们要注意的是`getChildMeasureSpec()`这个方法， 这个方法会根据传入的MeasureSpace返回child最后确定的MeasureSpace， 由于这里我们传入了`MeasureSpec.UNSPECIFIED`， 所以下面我们看看传入后会是什么样的返回结果（下面代码根据需要省略了部分，仅仅保留了用到的部分）
```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {  
  int specMode = MeasureSpec.getMode(spec);  
  int specSize = MeasureSpec.getSize(spec);  
  
  int size = Math.max(0, specSize - padding);  
  
  int resultSize = 0;  
  int resultMode = 0;  
  
  switch (specMode) {
  ...
  ...
  ...
   // Parent asked to see how big we want to be  
  case MeasureSpec.UNSPECIFIED:  
  if (childDimension >= 0) {  
      // Child wants a specific size... let him have it  
      resultSize = childDimension;  
      resultMode = MeasureSpec.EXACTLY;  
   } else if (childDimension == LayoutParams.MATCH_PARENT) {  
      // Child wants to be our size... find out how big it should  
      // be  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;  
      resultMode = MeasureSpec.UNSPECIFIED;  
   } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
      // Child wants to determine its own size.... find out how  
      // big it should be  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;  
      resultMode = MeasureSpec.UNSPECIFIED;  
   }  break;  
 }  //noinspection ResourceType  
  return MeasureSpec.makeMeasureSpec(resultSize, resultMode);  
}
```
```java
/**  
 * Always return a size of 0 for MeasureSpec values with a mode of UNSPECIFIED
 */
static boolean sUseZeroUnspecifiedMeasureSpec = false;
```
这里可以看出，由于child是match_parent的，所以`resultSize = size` 
`resultMode = MeasureSpec.UNSPECIFIED`
所以我们着重看size是如何计算的
```java
int specSize = MeasureSpec.getSize(spec);    
int size = Math.max(0, specSize - padding); 
```
关于这里，我引用网上的一张图
![Imgur](https://i.imgur.com/ozCeqW7.png)

这里问题就来了！ 所以 最后给View的MeasureSpace是啥呢？`MeasureSpec.UNSPECIFIED` size呢？ `WRAP_CONTENT`

这就是为什么会造成上吗那样的结果了。

开始以为这是一个Bug，后来想了想，都这么多版本了，为啥还在呢，后来又在Android的`R.id.design_bottom_sheet`中发现了利用这个所谓Bug的Feature...

总之，简单做包裹的话，乖乖去用`RelativeLayout`吧 复杂的就用`ConstraintLayout`, 这个FrameLayout，有点坑。
> Written with [StackEdit](https://stackedit.io/).


