### 使用popstate监听back按钮的普通处理流程
```
var timer = 0,
  bool = false;

function init() {
    clearTimeout(timer);
    timer = setTimeout(function() {
        bool = true;
    }, 500);    
    pushHistory();
    bindEvent();
};

function pushHistory() {
    var state = {
        title: "title",
        url: "#backtip"
    };
    window.history.pushState(state, "title", "#backtip");
};

function bindEvent() {
    function popstate(e) {
        if (bool) {
            //这里做响应处理
        }
    }
    window.addEventListener("popstate", popstate, false);
};

init();//初始化
```
假定从index.html页面跳转到charge_bind.html页面，charge_bind.html使用了上面这段代码初始化，url变成了charge_bind.html#backtip。 
如果此时使用浏览器back，url会跳转到charge_bind.html，popstate正常响应。这一块是没有问题的。问题是如果在charge_bind.html#backtip页面跳转 
到新的protocol.html页面。history的记录如下。 
![Alt text](../img/history.png)
 
 
此时如果从protocol.html使用浏览器返回Android和ios表现不一致。 
#### ios
url返回到charge_bind.html#backtip，表现为：页面返回但不刷新（不会初始化），响应popstate   
如果再back  
url返回到charge_bind.html，表现为：页面返回但不刷新（不会初始化），响应popstate  
如果再back  
返回到index.html  

#### android
url返回到charge_bind.html#backtip，表现为：页面返回并刷新（会初始化），不响应popstate  
这里因为执行了init，在history中又新添加了一个记录charge_bind.html#backtip  
如果再back   
url返回到charge_bind.html#backtip，表现为：页面返回但不刷新（不会初始化）,响应popstate  
再back  
url返回到charge_bind.html,表现为:页面返回并刷新（会初始化）,不响应popstate  
这里因为执行了init，在history中又新添加了一个记录charge_bind.html#backtip  
再back  
url返回到charge_bind.html,表现为:页面返回但不刷新（不会初始化）,响应popstate  
在back  
返回到index.html  
  
整体的back过程如下
![Alt text](../img/history-back.png)