## 背景
andriod手机webview中调用h5的页面的时候注入了一个native变量(js访问这个变量提供的方法可以调用手机app的功能)。但有时js访问native提示访问未定义的变量错误，原因是可能该变量还没有注入成功导致。  
ios不存在这个问题，iOS webview不需要注入native变量，而是js和ios webview约定好native变量的调用方式，js中默认就会有native这个全局变量。  

看一下android webview注入这个变量的代码
```
@Override
public void onProgressChanged (WebView view, int newProgress) {
    BaseWebView baseWebView = (BaseWebView)view;
    //为什么要在这里注入JS
    //1 OnPageStarted中注入有可能全局注入不成功，导致页面脚本上所有接口任何时候都不可用
    //2 OnPageFinished中注入，虽然最后都会全局注入成功，但是完成时间有可能太晚，当页面在初始化调用接口函数时会等待时间过长
    //3 在进度变化时注入，刚好可以在上面两个问题中得到一个折中处理
    //为什么是进度大于25%才进行注入，因为从测试看来只有进度大于这个数字页面才真正得到框架刷新加载，保证100%注入成功
    if (newProgress > 25 && !baseWebView.isInjectedJS()) {
        baseWebView.loadJS(JsCallJava.PRELOAD_INTERFACE_JS);
        baseWebView.setIsInjectedJS(true);
        Log.d(" inject js interface completely on progress " + newProgress);
    }
    super.onProgressChanged(view, newProgress);
}
```
可以看到，Android手机是在页面加载进度到25%的时候注入变量的。大部分手机在这个时候注入都是能访问到native的。目前主要集中在红米手机上可能会访问不到native。

## 处理方案
1. 在当跳转发生在Android webview到h5页面，并且js检测native不存在的情况下做后面的处理  
1. js自定义native全局变量并定义native对象所拥有的所有方法，这些方法中将方法名称和参数打包插入一个队列callList。  
1. 循环检测直到检测到webview注入了native为止。遍历callList，执行每一个元素对应的native方法。  
###### 代码如下

```
//处理部分Android手机在访问webview注入的native变量的时候改变量不存在的问题.
if(!window.native && $.bom.query("from") === "android") {//检测native
	var callList = [],
		timer = 0;

	window.native = {
		front:true//用来识别js自定义的native。
	};
	//模拟native的方法
	$.each("getHtmlAttribute openView share copy openWeChat openAppLoading closeAppLoading closeCurrentView openCurrent openProductDetail".split(" "), function(i, name) {
		window.native[name] = function(){
			callList.push({
				funcName: name,//获取函数自己的名称
				args: arguments
			});
		};
　　});

	function checkNative(){//检查Android注入的native是否可用
		if(window.native.front){
			timer = setTimeout(checkNative, 100);
		}else{
			for(var i = 0; i < callList.length; i++){
				window.native[callList[i].funcName].apply(window, callList[i].args);
			}
		}
	}
	checkNative();
}
```



