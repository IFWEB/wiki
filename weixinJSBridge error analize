首先字面上理解的意思是WeixinJSBridge这个对象没有被定义，这种报错一般出现时需要满足以下几个条件：

1. WeixinJSBridge变量没有声明或者声明后没有赋值。
2. WeixinJSBridge被调用，比如WeixinJSBridge.name。

因此，我们逐个分析上面两条原则：

1. 这个变量由谁声明赋值及为何没有被定义赋值？
2. 在哪里被谁调用？

首先，我们来看第一个问题：
>WeixinJSBridge是由微信app定义的js对象，在微信webview初始化之后添加。添加的时间点，不同的设备有不同的时间，时间的早晚主要取决于设备的类型和性能，一般滞后于html页面中js执行的时间。低版本安卓设备要更晚一些，在window.onload事件之后才初始化出来。
在微信中刚打开html,js代码执行时多数情况WeixinJSBridge对象都还未创建，此时调用就会报错。

其次，调用者主要分为两类：
>1.开发者的js调用;
早期，微信没有完整的权限控制时，开发者可以直接调用WeixinJSBridge此api；
当控制了权限时，无法直接调用，WeixinJSBridge，必须经过微信的js封装并通过授权后才可以调用WeixinJSBridge。页面开发者通过植入：

``` 
<script src="//res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
``` 

>此段微信脚本，并且通过服务器端实时注册微信授权后才可调用。我们平台是通过这个nodeserver来实现的：

``` 
<script>
(function() {
var GLOBAL_FOOT = "my";
var hm = document.createElement("script");
hm.src = "//weixin.nongfadai.com/handler/weixin/sign.js?v="+new Date().getTime();
var s = document.getElementsByTagName("script")[0];
s.parentNode.insertBefore(hm, s);
})();
</script>
``` 


>当我们调用时，WeixinJSBridge还未被初始化出来，就会报错。这种情况出现在我们有应用微信分享功能的页面。

>2.微信安卓版x5内核中会自己添加节点来调用。
在安卓机已验证，html页面在微信上允许时，微信都会给html页面最后自动添加三个节点，具体功能未知，但是会调用到WeixinJSBridge，此时WeixinJSBridge对象还未初始化时就会报错，但并不是所有页面都会报错。ios不会出现此问题。
