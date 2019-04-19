# node 6.3以下版本 
使用node-inspector调试node后台程序

环境：chrome浏览器    
安装node-inspector
```
$ npm install -g node-inspector
```

启动inspector服务：
```
$ node-inspector &  
```

以debug模式运行node.js应用：
```
$ node --debug app.js  
```

>需要注意，如果你主进程fork多个子进程，需要为子进程添加不同的debug监听端口,如下  
```
//普通的fock
childProcess.fork('./child-p.js');
//解决监听--debug 5858端口冲突,主进程监听的也是5858端口，所以子进程需要另外一个端口
childProcess.fork('./child-p.js', [], {execArgv: ['--debug=5859']});
```

通过URL http://127.0.0.1:8080/debug?port=5858 就可以进行调试主进程了，调试子进程使用你配置的debug监听端口即可。

可以使用chrome插件调试工具在chrome上调试代码 [NIM--Node Inspector Manager](https://chrome.google.com/webstore/detail/nim-node-inspector-manage/gnhhdgbaldcilmgcpfddgdbkhjohddkj)  

# node 6.3以上版本
Node.js提供了一个内置的基于DevTools的调试器

调试模式启动node
```
$ node --inspect app.js
Debugger listening on 127.0.0.1:9229.
To start debugging, open the following URL in Chrome:
    chrome-devtools://devtools/bundled/js_app.html?experiments=true&v8only=true&ws=127.0.0.1:9229/dc9010dd-f8b8-4ac5-a510-c1a114ec7d29
```

打开chrome的调试模式`chrome://inspect`,
![打开inspect](https://github.com/IFWEB/wiki/blob/master/node/img/inspect-chrome.jpg)   

二次进入可以在chrome程序员开发工具下面进入  
![打开inspect](https://github.com/IFWEB/wiki/blob/master/node/img/inspect-devTools.jpg)  

选择sources打断点调试  
![调试inspect](https://github.com/IFWEB/wiki/blob/master/node/img/inspect-sources.jpg)

# 参考
[--inspect](https://nodejs.org/en/docs/inspector/)语法

