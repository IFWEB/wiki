## 清除浮动实例
![clear的详细描述](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear)  
目前在使用中主要用在清除浮动，撑开父元素。主要有两种方式：**伪类方式**和 **清除浮动节点方式**  


## 伪类清除浮动  
当子元素设置为浮动，要撑开父元素，父元素使用伪类清除浮动。**推荐使用这种方式**，最好定义成全局样式，便于后面使用
```
//css样式,正常来说伪元素使用"::",但是老版本ie8及以下不支持,所以使用了":"方式
.clearfix:before,.clearfix:after{
    content: "";
    display: block; 
    clear: both;
}
.float{
    float: left;
    width: 50px;
    background: red;
}

//DOM节点
<div class="parent clearfix">
    <div class="float">左侧浮动宽为50</div>
</div>
```
需要注意的是css3标准中"::"才表示伪元素，":"表示伪类。在css2中伪类和伪元素并没有区分。IE8及以下的版本中对"::"支持都不是很好。所以使用":"表示伪类兼容性更好以下。不过不排除后面的浏览器严格区分伪类和伪元素。建议两者都写上  
当然content的用法还有更复杂的，如下,原理是让content的内容区域高度为0  
```
.clearfix:before,.clearfix:after{
    content: ".";
    clear: both;
    display: block;
    font-size: 0;/*或者height: 0px*/
}
```

## 专用节点清除浮动
在浮动节点的后面添加一个清除浮动的节点也是一种方式。个人不推荐使用（额外的节点影响代码可读性）
```
//css样式,正常来说伪元素使用"::",但是老版本ie8及以下不支持,所以使用了":"方式
.clearfix{
    clear: both;
    display: block;/*防止样式套用时，该元素不是快级元素*/
}
.float{
    float: left;
    width: 50px;
    background: red;
}

//DOM节点
<div class="parent">
    <div class="float">左侧浮动宽为50</div>
    <div class="clearfix"></div>
</div>

```
