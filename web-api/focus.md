## ios手机通过js实现focus异常分析

####  现象和原因分析
页面的js代码中执行 $("#inputElement").focus() 发现无效。iOS下，通过JS调用键盘或设置focus是被禁止的，亦或许是一个bug。在这种情况下，
其实focus事件已经执行了，input框没有获得焦点，iPhone键盘跳不出来。但是可以监听其他点击事件，在回调中添加这个命令就可以了。

```javascript
//用户点击#element元素可以激活小键盘
$("#element").on("click",function(){
	$("#inputElement").focus();
});
```
#### 解决方案
1. 绑定一个click事件，在click事件响应中执行$("#inputElement").focus()。
 
