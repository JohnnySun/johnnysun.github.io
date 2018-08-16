---
title: Rxjava简单的源码分析
author: JohnnySun
tags: 'RxJava,Android,Thread'
categories: Android
date: 2018-8-16

---

<p>Rxjava create 调用流程</p>
<p><img src="blob:https://stackedit.io/1f80cb2f-b3dd-4d38-ac39-a7999624338f" alt="Image.png"></p>
<p><strong>这里来看<strong><strong>Rxjva</strong></strong>如何来做<strong><strong>线程切换，以及</strong></strong>observeOn<strong><strong>和</strong></strong>subscribeOn****的区别</strong></p>
<p><strong>我<strong><strong>们先来看</strong></strong>observeOn****的</strong></p>
<ul>
<li>observeOn的本质是lift变换</li>
</ul>
<p><img src="blob:https://stackedit.io/6ce31bdc-b2e1-4330-aac2-9e4819d6571a" alt="Image_1.png"></p>
<ul>
<li>lift本质是不考虑背压的unsafeCreate</li>
</ul>
<p><img src="blob:https://stackedit.io/721fa430-056e-48ef-b705-8a600d1c1fc8" alt="Image_2.png"></p>
<ul>
<li>这里来看一下OnSubScribeLift方法实现</li>
</ul>
<p><img src="blob:https://stackedit.io/d11d2f27-eeea-4762-8cb5-801ceb7376a6" alt="Pasted Graphic.tiff"></p>
<p><strong>结论：这里由上面看出，<strong><strong>observeOn</strong></strong>的本<strong><strong>质是影响</strong></strong>Subscribe<strong><strong>的</strong></strong>调用线程</strong></p>
<p><strong>我<strong><strong>们再来看</strong></strong>subscribeOn****的</strong></p>
<p>本质上也是个create 也是创建一个OnSubscribe</p>
<p><img src="blob:https://stackedit.io/b97a5490-2f3d-47f4-8bbb-0d51a8328354" alt="Image_3.png"></p>
<p><img src="blob:https://stackedit.io/26a95143-53cc-4cab-b24e-b41b0372195a" alt="Image_4.png"></p>
<p>来看下OperatorSubscribeOn</p>
<p><img src="blob:https://stackedit.io/d14e16d1-957c-43ce-a405-e3da6ee95c3c" alt="Image_5.png"></p>
<p>所以 SubscribeOn 和 ObserveOn 切换线程并不是完全像网上所说的那样 SubscribeOn切换前面的方法执行线程 ObserveOn切换后面的方法执行线程</p>
<p><strong>应该是</strong> <strong>SubscribeOn<strong><strong>切换前面的</strong></strong>onSubscribe.call</strong>**()**<strong>方法的执行线程</strong></p>
<p><strong>而</strong> <strong>ObserveOn****切换他后面</strong> <strong>Observe<strong><strong>方法中</strong></strong>OnNext OnComplete****登方法切换的线程</strong></p>
<p><strong>至于为什么<strong><strong>ObserveOn</strong></strong>能够切换<strong><strong>map flat</strong></strong>等方法的线程</strong>  <strong>而<strong><strong>SubscribeOn</strong></strong>不行</strong>  <strong>是因为<strong><strong>map float</strong></strong>后面的<strong><strong>action</strong></strong>回调是在他们创建的新的<strong><strong>Observeable</strong></strong>的<strong><strong>subscribe</strong></strong>方法中回调回来的</strong>  <strong>而不是在<strong><strong>onSubscribe.call</strong></strong>中</strong></p>
<p>这里是对subscribe的执行方法 他会回调最后一个Observable.java中的onSubscribe的call方法使得rxjava的调用流程就像开始的第一张图那样。</p>
<p><img src="blob:https://stackedit.io/615d25b0-ac3e-459d-b257-cef8be94c646" alt="Pasted Graphic 1.tiff"></p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

