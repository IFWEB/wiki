#### 写在前面

由于笔者所在的团队使用fis3打包工具搭配modJS来解决js模块化，并且最近也在研究js模块化方案，故写下这篇文章来解读modJS的实现细节。

限于笔者水平，如果有错误或不严谨的地方，请给予指正，十分感谢。


#### 一、JS中的模块规范（AMD/CMD/CommonJS/ES6）
---

1. CommonJS
- 使用的是同步风格的require
- node.js的模块系统，是参照CommonJS规范定义的

2. [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88))（Asynchronous Module Definition）异步模块定义
- AMD规范 https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88)
- 使用的是回调风格的支持异步
- 以RequireJS 为代表

3. CMD (Common Module Definition) 通用模块定义
- CMD规范 https://github.com/seajs/seajs/issues/242
- 以SeaJS为代表

4. ES6模块化
- ES6 模块化是欧洲计算机制造联合会 ECMA 提出的 JavaScript 模块化规范，它在语言的层面上实现了模块化。浏览器厂商和 Node.js 都宣布要原生支持该规范。它将逐渐取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案
- 缺点在于目前无法直接运行在大部分 JavaScript 运行环境下，必须通过工具转换成标准的 ES5 后才能正常运行

#### 二、js模块化方案 [modJS](https://github.com/fex-team/mod)的使用
---

首先，来看一下modJS的简介以及使用方法：

1. modJS简介
> modJS是一个精简版的AMD/CMD规范，并不完全遵守AMD/CMD规范，目的在于希望给使用者提供一个类似nodeJS一样的开发体验，同时具备很好的线上性能。

    
2. 使用
- 使用defined(id,factory)来定义一个模块
>在平常开发中，只需写factory中的代码即可，无需手动定义模块。打包工具fis3会自动将模块代码嵌入factory的闭包里。 factory提供了3个参数：require, exports, module，用于模块的引用和导出。

典型的例子
```
// a.js 文件
define('js/a', function(require, exports, module) {
    function init() {
        console.log('模块a被引用')
    }
    return { init: init }
    // or 
    // exports.init = init
    // or
    // modules.exports = { init : init }
})
```

- 使用require (id) 来引用已预先加载完成的模块
> 和NodeJS里获取模块的方式一样，非常简单。因为所需的模块都已预先加载，因此require可以立即返回该模块。

典型的例子
```
// index.html 文件
<script src="./mod.js" type="text/javascript"></script> 
<script src="./js/a.js" type="text/javascript"></script> 
 <script type="text/javascript">
    require('js/a').init();
</script> 
```
- 使用require.async (ids, onload, onerror) 来引用异步加载的模块
>考虑到有些模块无需在启动时载入，因此modJS提供了可以在运行时异步加载模块的接口。ids可以是一个模块名，或者是数组形式的模块名列表。当所有都加载都完成时，onload被调用，ids对应的所有模块实例将作为参数传入。如果加载错误或者网络超时，onerror将被触发。超时时间通过require.timeout设置，默认为5000(ms)。

> 使用require.async获取的模块不会被打包工具安排在预加载中，因此在完成回调之前require将会抛出模块未定义错误。

典型的例子:
```
// index.html 文件
 <script src="./mod.js" type="text/javascript"></script> 
 <script type="text/javascript">
    require.async('js/a',function(mod){
        mod.init()
    },function(id){
        console.error("模块" + id + "加载失败")
    });
</script> 
```


- 使用require.resourceMap(obj) 解析模块依赖树

> 通过require.resourceMap(obj) 解析模块依赖树，并获取模块对应的url。由打包工具自动完成。

典型的例子
```
// resource_map.js 文件
require.resourceMap({
    "pkg": {},
    "res": {
        "js/a": {
            "url": "js/a.js",
            "type": "js"
        },
        "js/b": {
            "url": "js/b.js",
            "type": "js"
        },
        "js/c": {
            "url": "js/c.js",
            "type": "js",
            "deps": ["js/a", "js/b"]
        }
    }
})
```

- require.loadJs (url)

>异步加载脚本文件，不做任何回调

- require.loadCss ({url: cssfile})

> 异步加载CSS文件，并添加到页面

- require.loadCss ({content: csstext})

> 创建一个样式列表并将css内容写入

在这篇文章中，只讨论js的模块化方案，故，不讨论require.loadCss以及没有回调的require.loadJs。 

#### 三、modJS的实现细节
---

从modJS的使用上可以看出，modJS暴露了两个全局变量define、require,现在跟随modJS源代码研究一下实现细节。 [点击查看modJS的仓库地址](https://github.com/fex-team/mod)

1. define(id,factory)

> 用define函数包裹js模块来完成模块的定义,包裹操作由打包工具fis3自动完成。

```
// mod.js 文件
var require, define;

(function(global) {
    if (require) return; // 避免重复加载mod.js而导致已定义模块丢失

    var factoryMap = {},
        modulesMap = {},
        loadingMap = {},
        resMap = {},
        pkgMap = {};
        
    /**
     * @desc 定义js模块, 用define函数包裹模块，由打包工具自动完成
     * @param {String} id 模块唯一标识
     * @param {Function} factory 工厂函数，接受三个参数require、exports、modules,其中exports只是modules.exports的引用
     * @return void
     * @example define('js/a',function(require,exports,module){ return { init: function init(){} } })
     */
    define = function(id, factory) {
        id = alias(id);
        factoryMap[id] = factory;

        var queue = loadingMap[id]; // 异步加载模块,回调函数依次执行
        if (queue) {
            for (var i = 0, len = queue.length; i < len; i++) {
                queue[i]()
            }
            delete loadingMap[id]; // 从正在加载中移除
        }
    }
    
    function alias(id) {
        return id.replace(/\.js$/i, '');
    }
    
})(this) // 使用函数包裹，避免污染全局变量
```
比如，当我们有一个js文件a.js，文件内容如下：
```
// a.js 文件
console.log('模块a');

function init() {
    console.log('模块a被引用')
}

return { init: init }
// or 
// exports.init=init
// or
// modules.exports={init:init}
```
用打包工具进行define函数包裹后，a.js文件就变成了如下内容，此时我们就完成了对一个标识为“js/a”的模块的包裹:
```
// a.js 文件
define('js/a', function(require, exports, module) {
    console.log('模块a');

    function init() {
        console.log('模块a被引用')
    }

    return { init: init }

    // or 
    // exports.init=init
    // or
    // modules.exports={init:init}
})
```
当检测到模块被引用，打包工具会将该模块对应的srcipt标签自动嵌入HTML文档中进行预加载，加载完成后浏览器会立即执行，这样就完成了一个模块的定义。

```
// index.html 文件
<script src="./mod.js" type="text/javascript"></script> 
<script src="./js/a.js" type="text/javascript"></script>
```

2. require(id)

在上一步操作中，完成了对模块标识为“js/a”的模块的定义，现在可以通过require(id)对已定义的模块进行引用了。 require(id)所需要做的就是初始化factory。
```
// mod.js 文件
var require, define;

(function(global) {

     /** 此处省略部分代码 **/ 

    /**
     * @desc 同步引用已定义的js模块，若该模块未定义，则抛出 “Can not find module”错误
     * @param {String} id 模块唯一标识
     * @return {Object|String} 返回模块内部执行的return语句，如果模块内部没有执行return，则返回模块内部调用的 moduls.exoprts； return 优先级高于 module.exports 
     * @example require('js/a')
     */
    require = function(id) {
        id = alias(id);
    
        var module = modulesMap[id];
    
        // 避免重复初始化factory
        if (module) {
            return module.exports
        }
    
        // 初始化factory
        var factory = factoryMap[id];
        if (!factory) {
            throw "Can not find module `" + id + "`";
        }
    
        module = modulesMap[id] = { exports: {} };
        var result = typeof factory === "function" ? factory.apply(module, [require, module.exports, module]) : factory;
    
        if (result) { // return 优先级高于 module.exports 
            module.exports = result;
        }
        return module.exports
    }
    
    function alias(id) {
        return id.replace(/\.js$/i, '');
    }

})(this)
```

3. requier.asyn(ids,onload,onerror)

在上面的介绍中，我们知道：通过define(id,factory)函数包裹一个模块，并使用打包工具fis3自动将该模块对应的script内嵌至HTML文档中完成模块的预加载，然后require(id)函数再引用已经预加载好的模块。 但考虑到有些模块无需在启动时载入，所以需要通过requier.async(ids,onload,onerror)进行运行时异步加载模块

那么，运行时异步加载模块需要解决那些问题呢?
- 模块内部依赖解析
- 模块资源定位
- 通过DOM操作动态的往HTML head标签里插入HTML script标签来异步加载模块
- 模块及模块内部依赖异步加载完成后的执行onload回调，如果加载失败或超时执行onerror回调

对于模块内部依赖解析和模块资源定位这个两个问题，modJS是通过require.resourceMap函数解析打包工具fis3生成的rerource_map对象实现的。

比如，js目录下有三个js文件a.js、b.js、c.js，c.js引用了a.js和b.js,那么打包工具就会解析文件之间的依赖关系以及资源定位，生成一个json对象:
```
"pkg": {},
"res": {
    "js/a": {
        "url": "js/a.js",
        "type": "js"
    },
    "js/b": {
        "url": "js/b.js",
        "type": "js"
    },
    "js/c": {
        "url": "js/c.js",
        "type": "js",
        "deps": ["js/a", "js/b"]
    }
}
```
再使用require.resourceMap(obj)函数进行包裹，生成一个resource_map.js文件,内嵌至HTML文档中,浏览器加载完resource_map.js文件后，执行require.resourceMap函数就完成了模块内部依赖解析以及模块资源定位

```
// resource_map.js 文件
require.resourceMap({
    "pkg": {},
    "res": {
        "js/a": {
            "url": "js/a.js",
            "type": "js"
        },
        "js/b": {
            "url": "js/b.js",
            "type": "js"
        },
        "js/c": {
            "url": "js/c.js",
            "type": "js",
            "deps": ["js/a", "js/b"]
        }
    }
})


// mod.js 文件
var require, define;

(function(global) {

    /** 此处省略部分代码 **/ 
    
    /** 
     * @desc js模块依赖解析
     * @param {Object} obj js模块依赖对象： { pkg: {}, res: { 'js/a': { url: 'js/a.js', type: 'js' }, 'js/b': { url: 'js/b.js', type: 'js', deps: ['js/a'] } } }
     * @return void
     */
    require.resourceMap = function(obj) {
        var k, col;
    
        // merge `res` & `pkg` fields
        col = obj.res;
        for (k in col) {
            if (col.hasOwnProperty(k)) {
                resMap[k] = col[k];
            }
        }
        
        col = obj.pkg;
        for (k in col) {
            if (col.hasOwnProperty(k)) {
                pkgMap[k] = col[k];
            }
        }
    }

})(this)

// index.html
<script src="./mod.js" type="text/javascript"></script>
<script src="./resource_map.js" type="text/javascript"></script>
<script type="text/javascript">
    require.async('js/c', function(mod) {
        mod.init()
    });
</script>
    
```
现在，解决了模块内部依赖解析和资源定位的问题，就可以通过DOM操作动态的往HTML head标签里插入HTML script标签来异步加载模块，并在模块及模块内部依赖异步加载完成后的执行onload回调，如果异步加载失败或超时的执行onerror回调，异步加载超时时间，modJS通过require.timeout来设置，默认为5s

```
// mod.js 文件
var require, define;

(function(global) {
    
    /** 此处省略部分代码 **/ 
        
    var head = document.getElementsByTagName('head')[0];

    /**
     * @desc 异步加载js模块
     * @param {String} id 模块唯一标识
     * @param {Function} onload 所有的模块(包括模块内部依赖)都加载完成后执行回调函数
     * @param {Function} onerror 模块加载错误或超时时执行的回调函数,超时时间通过require.timeout设置，默认5s
     * @example require.async(id,onload,onerror)
     * @example require.async([id1,id2,...],onload,onerror)
     * @tips 先异步加载该模块，再异步加载该模块的依赖，为什么这种顺序不会出现问题？ 因为会等待所有的异步模块加载完毕之后才会执行onload函数
     */
    require.async = function(ids, onload, onerror) {
        if (typeof ids === 'string') {
            ids = [ids]
        }

        var needMap = {},
            needNum = 0;

        function findDependence(depArr) {
            for (var i = 0, len = depArr.length; i < len; i++) {
                var dep = alias(depArr[i]);

                if (dep in factoryMap) { // skip loaded
                    var child = resMap[dep] || resMap[dep + '.js']
                    if (child && 'deps' in child) { // 通过resource_map.js检查模块是否存在内部依赖，若存在，且不依赖本身，则递归内部依赖
                        (child.deps !== depArr) && findDependence(child.deps)
                    }

                    continue;
                }

                if (dep in needMap) { // skip loading
                    continue;
                }

                needMap[dep] = 1;
                needNum++;
                loadScript(dep, updateNeed, onerror) // 动态加载脚本。 updateNeed函数有权访问外部函数的变量(needNum，ids，onload)，并只能得到这些变量的最后一个值（闭包）

                var child = resMap[dep] || resMap[dep + '.js']
                if (child && 'deps' in child) { // 通过resource_map.js检查模块是否存在内部依赖，若存在，且不依赖本身，则递归内部依赖
                    (child.deps !== depArr) && findDependence(child.deps)
                }
            }
        }

        
        function updateNeed() {
            if (0 == needNum--) { // 等待所有的模块以及模块的内部依赖加载成功，再执行回调函数onload
                var args = [];
                for (var i = 0, n = ids.length; i < n; i++) {
                    args[i] = require(ids[i]); // 将加载完成的模块作为参数传递给onload回调函数，如果有模块为加载成功，将抛出Can not find module异常
                }
                typeof onload === 'function' && onload.apply(global, args) // onload函数的作用域指向全局
            }
        }

        findDependence(ids);
        updateNeed(); 
    }

    /** 
     * @desc 加载异步js脚本超时时间,默认5s
     */
    require.timeout = 5000;

    /** 
     * @desc 通过script标签动态加载脚本
     * @param {String} id 模块唯一标识
     * @param {Function} calback js模块loaded的回调函数
     * @param {Function} onerror: js模块errored的回调函数
     * @return void
     */
    function loadScript(id, callback, onerror) {
        var queue = loadingMap[id] || (loadingMap[id] = []);
        queue.push(callback)

        var res = resMap[id] || resMap[id + ".jd"]; // 通过resource_map.js获取模块对应的url
        var pkg = res.pkg;

        if (!res.url) return;
        if (pkg) { 
            url = pkgMap[pkg].url;
        } else {
            url = res.url || id;
        }

        createScript(url, onerror && function() {
            onerror(id)
        });

    }

    function createScript(url, onerror) {
        var script = document.createElement('script');

        if (onerror) {
            var tid = setTimeout(onerror, require.timeout); // 超时执行onerror

            function onload() {
                clearTimeout(tid) // loaded 清除定时器
            }

            if ('onload' in script) {
                script.onload = onload
            } else {
                script.onreadystatechange = function() {
                    if (this.readyState === 'loaded' || this.readyState === 'complete') {
                        onload();
                    }
                }
            }

            script.onerror = function() {
                clearTimeout(tid);  // errored 清除定时器
                onerror()
            };
        }

        script.src = url;
        script.type = "text/javascript";
        head.appendChild(script);

        return script;
    }

})(this);

```

#### 四：总结
---

通过以上，可以总结出modJS实现js模块化解决方案的6个要点:
- define(id,factory)，定义模块，对模块进行define函数包裹，由打包工具完成。
- require(id)，同步加载已定义的js模块，若该模块未定义，则抛出 “Can not find module”错误。
- require.resourceMap，通过resource_map.js 解析js模块依赖树，以及模块的资源定位，resource_map.js由打包工具解析文件依赖和资源定位并包裹require.resourceMap函数完成。
- require.timeout，设置异步加载模块的超时时间，默认5s。
- require.async(ids,onload,onerror)，通过DOM操作动态的往HTML head标签里插入HTML script标签来异步加载模块以及模块的内部依赖,script标签的src通过resourceMap取得。
- 异步加载模块以及模块的内部依赖完成后，通过require引入该模块，并作为参数传递给require.async的回调函数onload；异步加载失败或超时，执行onerror回调。


#### 五：附上完整的带注释modJS代码
---
```
var require, define;

(function(global) {
    if (require) return; // 避免重复加载mod.js而导致已定义模块丢失

    var factoryMap = {},
        modulesMap = {},
        loadingMap = {},
        resMap = {},
        pkgMap = {},
        head = document.getElementsByTagName('head')[0];

    /**
     * @desc 定义js模块, 用define函数包裹模块，由打包工具自动完成
     * @param {String} id 模块唯一标识
     * @param {Function} factory 工厂函数，接受三个参数require、exports、modules,其中exports只是modules.exports的引用
     * @return void
     * @example define('js/a',function(require,exports,module){ return { init: function init(){} } })
     */
    define = function(id, factory) {
        id = alias(id);
        factoryMap[id] = factory;

        var queue = loadingMap[id]; // 异步加载,回调函数依次执行
        if (queue) {
            for (var i = 0, len = queue.length; i < len; i++) {
                queue[i]()
            }
            delete loadingMap[id]; // 从正在加载中移除
        }
    }

    /**
     * @desc 同步加载已定义的js模块，若该模块未定义，则抛出 “Can not find module”错误
     * @param {String} id 模块唯一标识
     * @return {Object|String} 返回模块内部执行的return语句，如果模块内部没有执行return，则返回模块内部调用的 moduls.exoprts； return 优先级高于 module.exports 
     * @example require('js/a')
     */
    require = function(id) {
        id = alias(id);

        var module = modulesMap[id];

        // 避免重复初始化factory
        if (module) {
            return module.exports
        }

        // 初始化factory
        var factory = factoryMap[id];
        if (!factory) {
            throw "Can not find module `" + id + "`";
        }

        module = modulesMap[id] = { exports: {} };
        var result = typeof factory === "function" ? factory.apply(module, [require, module.exports, module]) : factory;

        if (result) { // return 优先级高于 module.exports 
            module.exports = result;
        }
        return module.exports
    }

    /**
     * @desc 异步加载js模块
     * @param {String|Array} ids 模块唯一标识
     * @param {Function} onload 所有的模块(包括模块内部依赖)都加载完毕后执行回调函数
     * @param {Function} onerror 模块加载错误或超时时执行的回调函数,超时时间通过require.timeout设置，默认5s
     * @example require.async(id,onload,onerror)
     * @example require.async([id1,id2,...],onload,onerror)
     * @tips 先异步加载该模块，再异步加载该模块的依赖，为什么这种顺序不会出现问题？ 因为会等待所有的异步模块加载完毕之后才会执行onload函数
     */
    require.async = function(ids, onload, onerror) {
        if (typeof ids === 'string') {
            ids = [ids]
        }

        var needMap = {},
            needNum = 0;

        function findDependence(depArr) {
            for (var i = 0, len = depArr.length; i < len; i++) {
                var dep = alias(depArr[i]);

                if (dep in factoryMap) { // skip loaded
                    var child = resMap[dep] || resMap[dep + '.js']
                    if (child && 'deps' in child) { // 通过resource_map.js检查模块是否存在内部依赖，若存在，且不依赖本身，则递归内部依赖
                        (child.deps !== depArr) && findDependence(child.deps)
                    }
                    continue;
                }

                if (dep in needMap) { // skip loading
                    continue;
                }

                needMap[dep] = 1;
                needNum++;
                loadScript(dep, updateNeed, onerror) // 动态加载脚本。 updateNeed函数有权访问外部函数的变量（needNum，ids，onload），并只能得到这些变量的最后一个值（闭包）

                var child = resMap[dep] || resMap[dep + '.js']
                if (child && 'deps' in child) { // 通过resource_map.js检查模块是否存在内部依赖，若存在，且不依赖本身，则递归内部依赖
                    (child.deps !== depArr) && findDependence(child.deps)
                }
            }
        }

        function updateNeed() {
            if (0 == needNum--) { // 等待所有的模块以及模块的内部依赖加载完成，再执行回调函数onload
                var args = [];
                for (var i = 0, n = ids.length; i < n; i++) {
                    args[i] = require(ids[i]); // 将加载完成的模块作为参数传递给onload回调函数，如果有模块未加载成功，将抛出Can not find module异常
                }
                typeof onload === 'function' && onload.apply(global, args) // onload函数的作用域指向全局
            }
        }

        findDependence(ids);
        updateNeed();
    }

    /** 
     * @desc 加载异步js脚本超时时间,默认5s
     */
    require.timeout = 5000;

    /** 
     * @desc js模块依赖解析
     * @param {Object} obj js模块依赖对象： { pkg: {}, res: { 'js/a': { url: 'js/a.js', type: 'js' }, 'js/b': { url: 'js/b.js', type: 'js', deps: ['js/a'] } } }
     * @return void
     */
    require.resourceMap = function(obj) {
        var k, col;

        // merge `res` & `pkg` fields
        col = obj.res;
        for (k in col) {
            if (col.hasOwnProperty(k)) {
                resMap[k] = col[k];
            }
        }

        col = obj.pkg;
        for (k in col) {
            if (col.hasOwnProperty(k)) {
                pkgMap[k] = col[k];
            }
        }
    }


    /** 
     * @desc 通过script标签动态加载脚本
     * @param {String} id 模块唯一标识
     * @param {Function} calback js模块loaded的回调函数
     * @param {Function} onerror: js模块errored的回调函数
     * @return void
     */
    function loadScript(id, callback, onerror) {
        var queue = loadingMap[id] || (loadingMap[id] = []);
        queue.push(callback)

        var res = resMap[id] || resMap[id + ".jd"]; // 通过resource_map.js获取模块对应的url
        var pkg = res.pkg;

        if (!res.url) return;
        if (pkg) {
            url = pkgMap[pkg].url;
        } else {
            url = res.url || id;
        }

        createScript(url, onerror && function() {
            onerror(id)
        });

    }

    function createScript(url, onerror) {
        var script = document.createElement('script');

        if (onerror) {
            var tid = setTimeout(onerror, require.timeout); // 超时执行onerror

            function onload() {
                clearTimeout(tid) // loaded 清除定时器
            }

            if ('onload' in script) {
                script.onload = onload
            } else {
                script.onreadystatechange = function() {
                    if (this.readyState === 'loaded' || this.readyState === 'complete') {
                        onload();
                    }
                }
            }

            script.onerror = function() {
                clearTimeout(tid); // errored 清除定时器
                onerror()
            };
        }

        script.src = url;
        script.type = "text/javascript";
        head.appendChild(script);

        return script;
    }

    function alias(id) {
        return id.replace(/\.js$/i, '');
    }

})(this); // 使用函数包裹，避免污染全局变量
```

限于笔者水平，如果有错误或不严谨的地方，请给予指正，十分感谢。

##### 参考：

- [#modJS](https://github.com/fex-team/mod)
- [fis3](http://fis.baidu.com/)
- [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88))
- [CMD](https://github.com/seajs/seajs/issues/242)