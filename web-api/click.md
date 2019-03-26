## IOS safari 点击失效详解
出现情况：
	使用 zepto 或 jQuery 的情况下，给一个元素添加click事件，如果事件是委托到 document 或 body 上，并且委托的元素是默认不可点击的（如 div, span 等），此时 click 事件在IOS safari中会失效。

#### 案例如下：
```html
<!DOCTYPE html>
<html>
<head>
    <title>ios click test</title>
</head>
<body>
    <div id="main">
        <div id="text">test test</br>test test</br>test test</br></div>
        <a id="link">this is a link</a>
        <button id="btn">hello world</button>
        <input type="text" name="name" id="name" placeholder="click input"/>
    </div>
    <script type="text/javascript" src="./deps/zepto.js"></script>
    <script type="text/javascript">
        $(document).on('click', '#text', function() {
            alert('text');      //IOS中点击失效
        }).on('click', '#link', function() {
            alert('link');      //IOS中点击有效
        }).on('click', '#btn', function() {
            alert('btn');      //IOS中点击有效
        }).on('click', '#name', function() {
            alert('input');    //IOS中点击有效
        });
    </script>
</body>
</html>
```

#### 解决方案
1. 将 `click` 事件直接绑定到目标​元素（​​即 `target`）上。
2. 将目标​元素换成 `&lt; a>` 或者 `button` 等可点击的元素。
3. 将 `click` 事件委托到非 `document` 或 `body` 的父级元素上。
4. 给目标元素加一条样式规则 `cursor: pointer`。
5. 使用原生方式委托事件。


## click呼起电话异常分析

#### 现象和原因分析
呼起电话链接\<a href="tel:11111">联系客服\</a>，呼起电话是这个标签的默认功能，响应click事件时触发。当文案"联系客服"的父节点（除开document外）绑定click事件的处理中执行e.preventDefault()，会导致呼起电话失败。

#### 解决方案
1. 去除 e.preventDefault()。 
2. 使用其他事件，比如tap事件代替click来执行想要的处理，保证click通畅。
