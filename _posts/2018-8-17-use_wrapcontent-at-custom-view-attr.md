---
title: 在自定义View中使用wrap_content字段
author: JohnnySun
tags: Android View
categories: Android
date: '2018-08-18'

---

今天这个不是啥干货， 就是踩坑的一个记录，干货我最近还没想好写啥

在我们自定义View的过程中，常常会用到自定义字段，很多时候也会使用 ```getDimensionPixelSize()``` 方法来获取自定义属性的大小

今天在做一个需求的时候突然在想，为什么在Android的```layout_width``` 这样的方法里可以使用```wrap_content```, ```match_parent``` 这样的方法，于是参考了下官方的代码，发现如果在在自己的工程中使用这个需要如下定义attr
```xml
<attr name="indicator_width" format="dimension">  
	<enum name="fill_parent" value="-1" />  
	<enum name="match_parent" value="-1" />  
	<enum name="wrap_content" value="-2" />  
</attr>
```

这之后 我就想当然的去使用平时常用的```getDimensionPixelSize()``` 去获取取值了，结果跑起来就crash了，
``` java
throw new UnsupportedOperationException("Can't convert value at index " + attrIndex  
  + " to dimension: type=0x" + Integer.toHexString(type));
```
原始的错误堆栈没了，反正就是抛了这个

分析源代码如下
```java
/**  
 * Retrieve a dimensional unit attribute at <var>index</var> for use * as a size in raw pixels.  This is the same as * {@link #getDimension}, except the returned value is converted to  
 * integer pixels for use as a size.  A size conversion involves * rounding the base value, and ensuring that a non-zero base value * is at least one pixel in size. * <p> * This method will throw an exception if the attribute is defined but is * not a dimension. * * @param index Index of attribute to retrieve.  
 * @param defValue Value to return if the attribute is not defined or  
 *                 not a resource. * * @return Attribute dimension value multiplied by the appropriate  
 *         metric and truncated to integer pixels, or defValue if not defined. * @throws RuntimeException if the TypedArray has already been recycled.  
 * @throws UnsupportedOperationException if the attribute is defined but is  
 *         not a dimension. * * @see #getDimension  
  * @see #getDimensionPixelOffset  
  */  
public int getDimensionPixelSize(@StyleableRes int index, int defValue) {  
	if (mRecycled) {  
	throw new RuntimeException("Cannot make calls to a recycled instance!");  
 }  
	final int attrIndex = index;  
	index *= AssetManager.STYLE_NUM_ENTRIES;  

	final int[] data = mData;  
	final int type = data[index+AssetManager.STYLE_TYPE];  
	if (type == TypedValue.TYPE_NULL) {  
		return defValue;  
	} else if (type == TypedValue.TYPE_DIMENSION) {  
		return TypedValue.complexToDimensionPixelSize(  
  data[index+AssetManager.STYLE_DATA], mMetrics);  
	} else if (type == TypedValue.TYPE_ATTRIBUTE) {  
		final TypedValue value = mValue;  
		getValueAt(index, value);  
		throw new UnsupportedOperationException(  
  "Failed to resolve attribute at index " + attrIndex + ": " + value);  
	}  
	throw new UnsupportedOperationException("Can't convert value at index " + attrIndex  
  + " to dimension: type=0x" + Integer.toHexString(type));  
	}
```

由于这里之前```<enum name="wrap_content" value="-2" /> ```定义的是个枚举类型，这里的 ```getDimensionPixelSize()``` 无法处理，就抛出了之前的异常，看了android的实现，发现他是使用```getLayoutDimension()```实现的，遂使用该方法代替， 成功拿到大小，并且通过```ViewGroup.LayoutParams.WRAP_CONTENT```进行判断即可。

好奇的我看了下```getLayoutDimension()```的实现方法，发现 他是刻意对TYPE_ENUM进行了判断
```java
/**  
 * Special version of {@link #getDimensionPixelSize} for retrieving  
 * {@link android.view.ViewGroup}'s layout_width and layout_height  
 * attributes.  This is only here for performance reasons; applications * should use {@link #getDimensionPixelSize}.  
 * * @param index Index of the attribute to retrieve.  
 * @param defValue The default value to return if this attribute is not  
 *                 default or contains the wrong type of data. * * @return Attribute dimension value multiplied by the appropriate  
 *         metric and truncated to integer pixels. * @throws RuntimeException if the TypedArray has already been recycled.  
 */
 public int getLayoutDimension(@StyleableRes int index, int defValue) {  
	if (mRecycled) {  
		throw new RuntimeException("Cannot make calls to a recycled instance!");  
	}  
	index *= AssetManager.STYLE_NUM_ENTRIES;  
	final int[] data = mData;  
	final int type = data[index+AssetManager.STYLE_TYPE];  
	if (type >= TypedValue.TYPE_FIRST_INT  
		&& type <= TypedValue.TYPE_LAST_INT) {  
		return data[index+AssetManager.STYLE_DATA];  
	} else if (type == TypedValue.TYPE_DIMENSION) {  
		return TypedValue.complexToDimensionPixelSize(data[index + AssetManager.STYLE_DATA], mMetrics);  
	}  
	return defValue;  
}
```

> Written with [StackEdit](https://stackedit.io/).


