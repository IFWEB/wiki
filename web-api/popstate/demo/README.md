## popstate注意事项
demo中页面进入顺序是first>a>b  
在a中做了pushState插入一个#backtip的记录，并监听popstate事件。做了两个兼容  
```
if(history.state !== 'backtip'){//第一个兼容
    window.history.pushState('backtip', "这是backtip处理", "#backtip");
}
```
**兼容原因是：**浏览器从b返回到a#backtip时大部分浏览器会刷新页面，避免二次pushState。  
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
**兼容原因是：**在ios上（微信和浏览器）从b页面返回a#backtip时也会进入popstate的监听。这个监听需要屏蔽掉。只处理a#backtip返回a的情况。  
>处理函数中有一个特别的处理：点击confirm弹出框取消按钮时，history.forward();让当前浏览器重新进入到a#backtip。避免popstate监听失效。  

history的状态如下  
![history状态图](https://github.com/IFWEB/wiki/blob/master/web-api/popstate/popstate-process.png)  