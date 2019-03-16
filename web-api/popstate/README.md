## pushState函数和popstate事件监听
详细参考：[pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)  、[popstate](https://developer.mozilla.org/zh-CN/docs/Web/Events/popstate)  
用途：用户离开当前页面前做挽留处理（popstate监听pushState插入的浏览记录pop）  

## 需要注意的问题
- 浏览器返回可能做刷新（重新初始化页面），所以pushState需要做兼容，避免多次pushState
- 不同浏览器对popstate监听的时机不同，正常浏览器只有从pushState的记录返回才会监听。但是ios上的微信和浏览器在返回到pushState的记录时还会做一次监听  

我们具体看一下浏览器记录的变更流程。页面顺序是first>a>b。其中a做了pushState处理  
![history状态图](https://github.com/IFWEB/wiki/blob/master/web-api/popstate/popstate-process.png)  
**注意：ios上浏览器的history记录的length和state的变化**

## demo中的兼容处理说明
[页面的demo](https://github.com/IFWEB/wiki/tree/master/web-api/popstate/demo)中页面进入顺序是first>a>b  
在a中做了pushState插入一个#backtip的记录，并监听popstate事件。做了两个兼容  
```
if(history.state !== 'backtip'){//第一个兼容
    window.history.pushState('backtip', "这是backtip处理", "#backtip");
}
```
**兼容原因是**： 浏览器从b返回到a#backtip时大部分浏览器会刷新页面，避免二次pushState。  
```
//添加监听,从a.html#backtip返回a.html时弹出对话框
window.addEventListener("popstate", function(){
    if(history.state && history.state === 'backtip'){//第二个兼容
        return;
    }else{
        var msg = confirm('...');
        if(msg == true){
            if(flag){//防重复,在手机上有时连续点击两次就会会触发两次,导致不对
                history.go(-1);
                flag = 0;
            }
        }else{
            //从"#backtip"返回,点击“继续”需要重新进入"#backtip"中去
            history.forward();
        }
    }     
});
```
**兼容原因是**： 在ios上（微信和浏览器）从b页面返回a#backtip时也会进入popstate的监听。这个监听需要屏蔽掉。只处理a#backtip返回a的情况。  
>处理函数中有一个特别的处理：点击confirm弹出框取消按钮时，history.forward();让当前浏览器重新进入到a#backtip。避免popstate监听失效。  

