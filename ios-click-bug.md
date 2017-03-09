###IOS safari 点击失效场景

出现情况：
	使用zepto 或 jQuery的情况下，给一个元素添加click事件，如果事件是委托到 document 或 body 上，并且委托的元素是默认不可点击的（如 div, span 等），此时 click 事件在IOS safari中会失效

####案例如下：
```
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

解决办法有五种可供选择
- 将 `click` 事件直接绑定到目标​元素（​​即 `target`）上
- 将目标​元素换成 `&lt; a>` 或者 `button` 等可点击的​元素
- 将 `click` 事件委托到​​​​​非 `document` 或 `body` 的​​父级元素上
- 给​目标元素加一条样式规则 `cursor: pointer`;
- 使用原生方式委托事件
