---
title: 一个不需要将View添加到Display的View欺骗截图框架
author: JohnnySun
tags: Android
categories: Android
date: '2018-11-18'

---

摘要：随着富媒体社交的兴起，越来越多的App开始在社交分享中拼接分享的内容，本文的主要内容就是在不添加到AttachToViewTree的情况下将一个ViewGroup里的内容完成加载并截图。

最近我们的App开始加入了越来越多的内容图片分享，比如小程序分享卡片 要做成这样的
![MiniProgramShare](https://i.imgur.com/kf5B7A4.png)
还有一些微信朋友圈长图片分享的
![Long Img Share](https://i.imgur.com/foCdJrW.png)

这里面的最大问题是，现在很多的View会在onAttachToWindow 或者Drawable为visiable的时候才会做很多工作（这里比如我们网络图片加载用的Fresco）

之前为了这个问题，通用的做法是，把这个要分享的Layout写好，添加到当前Activity的最底下，并且设置为invisible或者用别的View盖住，这样这个view就会正常的加载图片，可以正常的getBitmap。

但是这种通用的做法也有很大的弊端：

 1. 被截图的View要放在屏幕里，每次measure layout draw 都回去绘制一遍，增大了cpu开销
 2. 初始化的时候会多出时候很多的View，使页面的打开速度变慢
 3. 并不知道这个View里所有的网络图片是否都加载完成，如果图片没有加载完成，则可能导致截出来的图是空白或者展位图
 4. 需要写到Activity或者Fragment里，代码侵入性太强，这种需求多了后很难维护
 
 由于有以上这几个问题的存在，最近在当这种需求越来越多的之后，我想出了一个方案来欺骗View，让其在不添加到Window上的情况下也能正常的走完所有的逻辑，从而在不添加到Window的时候进行View截图

这种方式的好处如下：

 1. 由于不添加到Act或者Frag，系统每次的measure layout draw 都与被截图View无关
 2. 初始化的时候可以使用AsyncLayoutInflate，可以异步进行View初始化
 3. 对Fresco的图片加载进行了监听，保证图片全部加载完成后再进行截图
 4. 对现有Activity Fragment侵入性差
 
 ---
 说了这么多，现在也来讲一下原理：
 
```kotlin
/**
 * 本View用于处理制作view的bitmap的需求，例如小程序分享的view的bitmap，以前的方式由于一些控件的问题，
 * 需要将被截取的view放置于viewtree中，首先来说它会影响代码的结构，降低执行的效率，由于view被add到了tree中
 * 即使他在界面中被盖住了，实际上每次也仍然在绘制，大大了gpu的负担，并且当这类需求增多后，如果每一个都加到viewtree里，
 * 页面会非常混乱不好维护
 *
 * 推荐使用AddView的方式将完全设置好的View直接add到该layout中直接capture，如果先add进来在后续改变visiblity状态啥的，
 * onAttach逻辑可能会出现问题，就有可能出现WebImageView不加载的问题。
 */
class ViewCaptureLayout: RelativeLayout {

    private var viewHeight = 0
    private var viewWidth = 0
    private var measureHeightMode = View.MeasureSpec.UNSPECIFIED
    private var measureWidthMode = View.MeasureSpec.UNSPECIFIED
    private var imageIndex = ArrayList<ImageStatus>()

    private var viewHolder = RelativeLayout(context)

    private var needAsyncShot = false
    private var imageLoadCompleted = false
    private var hasFoundWebImageView = false
    private var isPrepared = false

    private var onShotCallback : ((Bitmap) -> Unit)? = null

    constructor(context: Context?) : super(context)
    constructor(context: Context?, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr)

    fun setViewSize(width: Int, height: Int) {
       when(height) {
           ViewGroup.LayoutParams.MATCH_PARENT -> {
               measureHeightMode = View.MeasureSpec.AT_MOST
               viewHeight = 0
           }
           ViewGroup.LayoutParams.WRAP_CONTENT -> {
               measureHeightMode = View.MeasureSpec.UNSPECIFIED
               viewHeight = 0
           }
           else -> {
               measureHeightMode = View.MeasureSpec.EXACTLY
               viewHeight = height
           }
       }

        when(width) {
            ViewGroup.LayoutParams.MATCH_PARENT -> {
                measureWidthMode = View.MeasureSpec.AT_MOST
                viewWidth = 0
            }
            ViewGroup.LayoutParams.WRAP_CONTENT -> {
                measureWidthMode = View.MeasureSpec.UNSPECIFIED
                viewWidth = 0
            }
            else -> {
                measureWidthMode = View.MeasureSpec.EXACTLY
                viewWidth = width
            }
        }
    }


    override fun removeAllViews() {
        super.removeAllViews()
        viewHolder.removeAllViews()
        imageIndex.clear()
    }

    /**
     * 这里先不去真正的addView，而是添加到以一个ViewHolder中，等待所有的add完成后zaiprepare时统一add
     */
    override fun addView(child: View?, index: Int, params: ViewGroup.LayoutParams?) {
        viewHolder.addView(child, index, params)
    }

    /**
     * 这里没有使用addView， 使用了轻量的attachViewToParent方法，目前看来带来的副作用也是有的，
     * 某些复杂的布局可能会不生效，这里推荐在再裹一层ViewGroup的方式，直接放复杂布局包裹在内部就好。
     * 这里之所以不用addView是因为这个ShotLayout并不需要添加到ViewTree，所以需要骗过系统，这里是
     * 用了dispatchVisibilityAggregated这个方法来通知系统 我的View和Drawable都是可见的。
     */
    private fun attachViewHolder() {
        imageIndex.clear()
        super.removeAllViews()
        insertImageListener(viewHolder)
        var params = viewHolder.layoutParams
        params.whenNull {
            params = RelativeLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT)
        }
        attachViewToParent(viewHolder, 0, params)
        requestLayout()
        invalidate()
    }

    private fun insertImageListener(viewGroup: ViewGroup) {
        if (viewGroup.childCount <= 0) {
            return
        }
        for (x in 0 until viewGroup.childCount) {
            var child = viewGroup.getChildAt(x)
            if (child is WebImageView) {
                // 这里取消所有WebIamgeView的动画，防止获取bitmap的时候正在做动画导致图片是白的
                child.setFadeDuration(0)
                //增加对webimageView加载的判断
                if (child.visibility == View.VISIBLE && !child.imageUrl.isNullOrBlank()) {
                    hasFoundWebImageView = true
                    var status = ImageStatus(child.toString(), false)
                    imageIndex.add(status)
                    var pos = imageIndex.size - 1
                    child.setOnControllerListener(object : BaseControllerListener<ImageInfo>() {
                        override fun onFailure(id: String?, throwable: Throwable?) {
                            super.onFailure(id, throwable)
                            imageIndex[pos].loadStatus = true
                            checkImageLoadingStatus()
                        }

                        override fun onFinalImageSet(id: String?, imageInfo: ImageInfo?, animatable: Animatable?) {
                            super.onFinalImageSet(id, imageInfo, animatable)
                            imageIndex[pos].loadStatus = true
                            checkImageLoadingStatus()
                        }
                    })
                }
            } else if (child is ViewGroup && child.visibility == View.VISIBLE) {
                insertImageListener(child)
            }
        }
    }

    /**
     * 用于检查所有的WebimageView是不是都加载完了
     */
    private fun checkImageLoadingStatus() {
        var completed = true
        imageIndex.forEach {
            if (!it.loadStatus) {
                completed = false
            }
        }

        if (completed) {
            imageLoadCompleted = true
            if (needAsyncShot) {
                getViewShotAndCallback()
            }
        }
    }

    /**
     * 在初始化完成后调用该方法
     */
    fun onPrepare(prepareCompleteCallback: (() -> Unit)) {
        if (Looper.getMainLooper().thread == Thread.currentThread()) {
            onPrepareThreadUnSafe()
            prepareCompleteCallback.invoke()
        } else {
            Observable.just(null).observeOn(AndroidSchedulers.mainThread())
                    .map { onPrepareThreadUnSafe() }
                    .subscribe { prepareCompleteCallback.invoke() }
        }

    }

    // 不要再UI线程之外调用，如有异步需求，请使用onPrepre
    @Suppress("MemberVisibilityCanBePrivate")
    @UiThread
    fun onPrepareThreadUnSafe(forcePrepare: Boolean = false) {
        if (isPrepared && !forcePrepare) {
            // 这里默认不会prepare两次，连续prepare两次可能会造成recycler的逻辑出现问题。如有需要，请使用forcePrepare
            return
        }
        isPrepared = true
        imageLoadCompleted = false
        attachViewHolder()
        if (!hasFoundWebImageView) {
            imageLoadCompleted = true
        }
        measure(View.MeasureSpec.makeMeasureSpec(viewWidth, measureWidthMode),
                View.MeasureSpec.makeMeasureSpec(viewHeight, measureHeightMode))
        // 这里这么做是为了解决在某些嵌套RecyclerView的时候，如果没有layout之后没有完全重新添加一次的话，vh会被重新create。
        // 这样的话 截图就不会正常显示了
        layout(0, 0, measuredWidth, measuredHeight)
        super.removeAllViews()
        imageIndex.clear()
        attachViewHolder()
        onDetachedFromWindow()
        onAttachedToWindow()
        //requestLayout()
        // 在开始不执行这个步骤的话就画不出来 很迷的操作
        val bitmap = Bitmap.createBitmap(measuredWidth, measuredHeight, Bitmap.Config.ARGB_4444)
        val canvas = Canvas(bitmap)
        canvas.drawColor(ContextCompat.getColor(context, R.color.c_ffffff))
        draw(canvas)
        bitmap.recycle()
    }

    // 调用该方法不一定会马上返回Bitmap，需等待所有图片加载完成才会通过回调返回bitmap
    fun getViewShot(onShotCallback : ((Bitmap) -> Unit)) {
        //this.onShotCallback = onShotCallback
        //getViewShotAndCallback()
        this.onShotCallback = onShotCallback
        if (imageLoadCompleted) {
            needAsyncShot = false
            getViewShotAndCallback()
        } else {
            needAsyncShot = true
        }
    }

    // 用于发送回调Bitmap
    private fun getViewShotAndCallback() {
        if (Looper.getMainLooper().thread == Thread.currentThread()) {
            getViewShotAndCallbackThreadUnSafe()
        } else {
            Observable.just(null).observeOn(AndroidSchedulers.mainThread())
                    .subscribe { getViewShotAndCallbackThreadUnSafe() }
        }
    }

    // 请在UI线程调用，非UI线程请调用getViewShotAndCallback()
    @Suppress("MemberVisibilityCanBePrivate")
    @UiThread
    fun getViewShotAndCallbackThreadUnSafe() {
        onShotCallback?.let {
            //invalidate()
            val bitmap = Bitmap.createBitmap(measuredWidth, measuredHeight, Bitmap.Config.ARGB_4444)
            val canvas = Canvas(bitmap)
            canvas.drawColor(ContextCompat.getColor(context, R.color.c_ffffff))

            draw(canvas)
            //onDetachedFromWindow()
            it.invoke(bitmap)
            onShotCallback = null
        }
    }

    // 以下也是为了骗WebimageView这类的View，让他认为我在window里了，我已经可见了，展示了等等

    override fun isAttachedToWindow(): Boolean {
        return true
    }

    override fun getWindowVisibility(): Int {
        return View.VISIBLE
    }

    override fun isShown(): Boolean {
        return true
    }

    class ImageStatus(val id: String, var loadStatus: Boolean)
}
```
 
 其实原理也很简单，由于本身这就是个ViewGroup，所以他是可以通过下发`dispatchVisibilityAggregated()` 的方式去通知内部的child atachToWindow，同时由于自己本身也复写了`isAttachToWindow()` `getWindowVisibility()` `isShown()` 方法，所以内部的child只要自己不是gone的状态，在加入到这个ViewGroup之后，就都会认为自己是Attachtowidow了，这样 我们的很多View，就可以在不添加到屏幕的情况下正常的工作，就可以截取出正常的View图片

这里之所以去复写`addView()` 方法， 也是为了方便使用，由于`dispatchVisibilityAggregated()`无法直接调用，这里使用了`attachViewToParent()`代替`addView()`，并且还能调用`dispatchVisibilityAggregated()`.并且 在复写`addView()`之后，也可以直接把写个View作为父View直接写到XML里，然后通过LayoutInflate Inflate出来，直接获取BItmap即可.


> Written with [StackEdit](https://stackedit.io/).


