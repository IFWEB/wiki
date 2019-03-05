### 使用node-inspector调试node后台程序
#### 准备
chrome浏览器  
安装node-inspector：$ npm install -g node-inspector  
然后需要通过浏览器连接到node-inspector，需要启动inspector服务：$ node-inspector &  
最后以debug模式运行node.js应用：$ node --debug app.js  
**需要注意，如果你主进程fork多个子进程，需要为子进程添加不同的debug监听端口,如下**  
```
//普通的fock
childProcess.fork('./child-p.js');
//解决监听--debug 5858端口冲突,主进程监听的也是5858端口，所以子进程需要另外一个端口
childProcess.fork('./child-p.js', [], {execArgv: ['--debug=5859']});
```
通过URL http://127.0.0.1:8080/debug?port=5858 就可以进行调试主进程了，调试子进程使用你配置的debug监听端口即可。

**自6.3版本起，Node.js提供了一个内置的基于DevTools的调试器，大部分不推荐使用Node Inspector**
