## webpack打包后结构
在配置中设置模式为开发模式（mode: "development"），代码打包后就不会压缩。先看一下待打包的代码
```
//webpack.config.js 打包配置文件
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  }
};

//main.js
var path = require('path');
console.log(path.join(__dirname,''));
```
代码非常简单，打包后将main.js打包到bundle.js中。

打包结果bundle.js中先把非结构相关的代码隐藏掉。将eval变量中的代码释放出来。
```
 (function(modules) { // webpackBootstrap
    //模块缓存
    var installedModules = {};

    //加载方法。类似node中的require方法
    function __webpack_require__(moduleId) {
        //...
    }
    //...

    //开始加载模块入口并返回exports
    return __webpack_require__(__webpack_require__.s = 0);
 })({

    "./main.js": (function(module, exports, __webpack_require__) {
        (function(__dirname) {
            var path = __webpack_require__( "C:\\nvm\v6.12.2\\node_modules\\webpack\\node_modules\\_path-browserify@0.0.0@path-browserify\\index.js");
            console.log(path.join(__dirname,''))
        }.call(this, "/"))
    }),

    0: (function(module, exports, __webpack_require__) {
        __webpack_require__("./main.js");
        //...
    }),

    "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_path-browserify@0.0.0@path-browserify\\index.js": (function(module, exports, __webpack_require__) {
        //...
    }),

    "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_process@0.11.10@process\\browser.js": (function(module, exports) {
        //...
    })
});
```
实际上就是一个匿名执行函数。参数是一个对象modules。modules中每一个属性（除开"0"之外）的key都是该目标文件依赖的资源的路径（依赖是通过require或者import等匹配收集的，main.js中依赖path模块，path模块依赖browser.js模块），属性key对应的值就是资源的执行内容。而属性"0"的值是整个webpack执行的额入口。正如代码片段
```
    ...
    //开始加载模块入口并返回exports
    return __webpack_require__(__webpack_require__.s = 0);
```
匿名函数最终会执行属性"0"的方法。而“0”属性的内容很简单，就是加载我们的入口文件“main.js”。再看“main.js”对应的文件可以看到，我们的require被改写成了**“__webpack_require__”**，这个就是webpack的模块加载方法。详细查看：[webpack模块方法](https://www.webpackjs.com/api/module-methods/)   
  
具体来看一下这个模块加载方法

```
    //加载方法。类似node中的require方法
    function __webpack_require__(moduleId) {

        // 检查模块是否在缓存中
        if(installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        // 创建新的模块(并且将其加入缓存)
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };

        // 执行模块的方法
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

        // 标记模块已经被加载
        module.l = true;

        // Return the exports of the module
        return module.exports;
    }
```

代码加注释应该能看的比较清楚了。模块的执行顺序就是：__webpack_require__(0)  -->  __webpack_require__("./main.js")  -->  __webpack_require__( "..._path-browserify@0.0.0@path-browserify\\index.js")  -->  __webpack_require__( "..._process@0.11.10@process\\browser.js")。在执行到有依赖是就执行依赖。  

fis3打包生成的那张异步依赖map图实际上是一种比webpack更好更清晰的一张依赖图。

下面是结构比较完整的代码
```
 (function(modules) { // webpackBootstrap
    //模块缓存
    var installedModules = {};

    //加载方法。类似node中的require方法
    function __webpack_require__(moduleId) {

        // 检查模块是否在缓存中
        if(installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        // 创建新的模块(并且将其加入缓存)
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };

        // 执行模块的方法
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

        // 标记模块已经被加载
        module.l = true;

        // Return the exports of the module
        return module.exports;
    }
    //...

    //开始加载模块入口并返回exports
    return __webpack_require__(__webpack_require__.s = 0);
 })({

    "./main.js": (function(module, exports, __webpack_require__) {
        (function(__dirname) {
            var path = __webpack_require__( "C:\\nvm\v6.12.2\\node_modules\\webpack\\node_modules\\_path-browserify@0.0.0@path-browserify\\index.js");

            console.log(path.join(__dirname,''))


        }.call(this, "/"))

        //# sourceURL=webpack:///./main.js?
    }),

    0: (function(module, exports, __webpack_require__) {
        __webpack_require__("./main.js");
        !(function webpackMissingModule() { var e = new Error("Cannot find module 'bundle.js'"); e.code = 'MODULE_NOT_FOUND'; throw e; }());

        //# sourceURL=webpack:///multi_./main.js_bundle.js?
    }),

    "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_path-browserify@0.0.0@path-browserify\\index.js": (function(module, exports, __webpack_require__) {
        (function(process) {// Copyright Joyent, Inc. and other Node contributors.
            //...
        }.call(this, __webpack_require__( "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_process@0.11.10@process\\browser.js")))
    }),

    "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_process@0.11.10@process\\browser.js": (function(module, exports) {
        //...
    })
});
```

>注：最新版本webpack4中key为0的属性没有了，初始化直接调用入口  

```
 (function(modules) { // webpackBootstrap
    //模块缓存
    var installedModules = {};

    //加载方法。类似node中的require方法
    function __webpack_require__(moduleId) {
        //...
    }
    //...

    //开始加载模块入口并返回exports
    return __webpack_require__(__webpack_require__.s = './main.js');
 })({

    "./main.js": (function(module, exports, __webpack_require__) {
        //...
    }),
    "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_path-browserify@0.0.0@path-browserify\\index.js": (function(module, exports, __webpack_require__) {
        //...
    }),
    "C:\\nvm\\v6.12.2\\node_modules\\webpack\\node_modules\\_process@0.11.10@process\\browser.js": (function(module, exports) {
        //...
    })
});
```