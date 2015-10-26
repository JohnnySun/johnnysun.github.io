<p>做一些总结，今天大致看了下GCM的使用方法，现在来稍微总结一下，作为一个温习。</p>
<h2 id="gcmlistenerservice"><strong>GcmListenerService</strong></h2>
<hr>
<p><em>GCM Receiver</em></p>
<p>要想使用GCM，首先要<br>
<code>import com.google.android.gms.gcm.GcmListenerService</code></p>
<p>之后从GcmListenerService继承一个新的类</p>
<pre><code>public class MyGcmListenerService extends GcmListenerService {
...
}
</code></pre>
<p>类中重载onMessageReceived<br>
函数原型</p>
<pre><code>public void onMessageReceived(String from, Bundle data) {
}
</code></pre>
<p>重载成如下</p>
<pre><code>@Override
public void onMessageReceived(String from, Bundle data) {
    String message = data.getString("message");
    Log.d(TAG, "From: " + from);
    Log.d(TAG, "Message: " + message);
</code></pre>
<hr>
<p><em>Gcm Sender</em></p>
<p>从服务端发送消息到GCM服务器<br>
使用http协议的方法如下</p>
<pre><code>Content-Type:application/json
Authorization:key=AIzaSyZ-1u...0GBYzPu7Udno5aA
{
  "to";/topics/foo-bar
  "data": {
  "message":"This is GCM Topic Message",
  }
 }
</code></pre>