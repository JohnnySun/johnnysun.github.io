---
title: 源码解读Rxjava实现的基本方法
author: JohnnySun
tags: 'RxJava, Android, Thread'
categories: Android
date: 2018-8-16

---

<p><em>本文对rxjava最常用的方法进行一部分源码解读，便于大家理解rxjava的工作原理</em></p>
<p>Rxjava create 调用流程</p>
<p><img src="https://i.imgur.com/6kp4Rzt.png" alt="Imgur"></p>
<p><strong>这里来看Rxjva如何来做线程切换，以及observeOn和subscribeOn的区别</strong></p>
<p><strong>我们先来看observeOn的</strong></p>
<ul>
<li>observeOn的本质是lift变换</li>
</ul>
<p><img src="https://i.imgur.com/y12GvOc.png" alt="Imgur"></p>
<ul>
<li>lift本质是不考虑背压的unsafeCreate</li>
</ul>
<p><img src="https://i.imgur.com/doYmNsw.png" alt="Imgur"></p>
<ul>
<li>这里来看一下OnSubScribeLift方法实现</li>
</ul>
<p><img src="https://i.imgur.com/jLmf8mS.png" alt="Imgur"></p>
<h2 id="结论：这里由上面看出，observeon的本质是影响subscribe的调用线程"><strong>结论：这里由上面看出，observeOn的本质是影响Subscribe的调用线程</strong></h2>
<p><strong>我们再来看subscribeOn的</strong><br>
本质上也是个create 也是创建一个OnSubscribe</p>
<p><img src="https://i.imgur.com/zHQ7rNy.png" alt="Imgur"><br>
来看下OperatorSubscribeOn<br>
<img src="https://i.imgur.com/oEtuyqe.png" alt="Imgur"><br>
所以 SubscribeOn 和 ObserveOn 切换线程并不是完全像网上所说的那样 SubscribeOn切换前面的方法执行线程 ObserveOn切换后面的方法执行线程<br>
<strong>应该是SubscribeOn切换前面的onSubscribe.call()方法的执行线程</strong></p>
<p><strong>而ObserveOn切换他后面Observe方法中OnNext OnComplete登方法切换的线程</strong></p>
<p><strong>至于为什么ObserveOn能够切换map flat等方法的线程而SubscribeOn不行是因为map float后面的action回调是在他们创建的新的Observeable的subscribe方法中回调回来的而不是在onSubscribe.call中</strong></p>
<hr>
<p>这里是对subscribe的执行方法 他会回调最后一个Observable.java中的onSubscribe的call方法使得rxjava的调用流程就像开始的第一张图那样。</p>
<p><img src="https://i.imgur.com/aLdfvaV.png" alt="Imgur"></p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>
