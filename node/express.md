### express的结构
![express结构](https://github.com/IFWEB/wiki/blob/master/node/img/express.jpg)  

### Express的中间件顺序
Express的中间件是顺序执行，从第一个中间件执行到最后一个中间件，发出响应。  
![Express的中间件顺序](https://github.com/IFWEB/wiki/blob/master/node/img/express-process.jpg)

### koa中间件顺序
koa是从第一个中间件开始执行，遇到next进入下一个中间件，一直执行到最后一个中间件，再逆序，执行上一个中间件next之后的响应代码，一直到第一个中间件执行结束才发出响应。  
![koa中间件顺序](https://github.com/IFWEB/wiki/blob/master/node/img/koa-process.jpg)

