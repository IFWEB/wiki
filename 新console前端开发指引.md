# 新console前端开发指引
## 开发环境配置
参照[新手指引](https://github.com/IFWEB/wiki/blob/master/%E5%89%8D%E7%AB%AF%E6%96%B0%E4%BA%BA%E6%8C%87%E5%BC%95.md) 新console前端代码运行
## 目录结构
- build、config 中放置webpack打包配置，node-master服务器相关程序
- data (非必须，默认没有) 放node master录制到本地的数据
- node_modules npm install 下来的项目依赖包
- src console前端主要代码，主要有：assets 资源（防止css、js、img等静态资源）、components（公共组件）、pages（页面，每个页面对应一个文件夹）、pubulic（公共依赖第三方插件）、store（vuex状态管理代码）
## 怎样开发一个新页面
### 配置新菜单
1. 与后台确定新菜单的对应路径，比如：busiManage/cpgl/lcjhgl代表业务管理、产品管理、理财计划管理。
2. 此时还没有vue文件，在busiManage下新建productControl文件夹，再在productControl下新建operate.vue文件。
>可以自由定义命名，注意尽量使用能对应上意义的名称。

3. 在src/assets/js/map.js文件中配置相对应菜单指向的vue文件路径:
```javascript
map['_busiManage_cpgl_lcjhgl'] = function() {
    return import ('index/busiManage/productControl/operate.vue');
};
```
4. 编写组件对应vue文件代码：
>注：
1. style统一放到最后，调试的时候为了方便可以放在最前，提交都放在最后。
2. css3尽量使用bootstrap排版，定制化的才写style。
3. components对应放的是组件，以上代码用的是es6写法，统一用es6语法编写。
4. beforeRouteEnter钩子函数会在路由进入后触发，只有写在route-view直接装载的vue文件才有，本例中pageHead.vue文件中就无法出发此钩子。
5. mounted钩子在组件被计算时就会触发，无法多次触发。
在build中的配置文件有配置路径@、src、index、assets、components
```javascript
            '@': resolve('src'),
            'src': path.resolve(__dirname, '../src'),
            'index': path.resolve(__dirname, '../src/pages/index'),
            'assets': path.resolve(__dirname, '../src/assets'),
            'components': path.resolve(__dirname, '../src/components')
```

```html
<template>
    <div class="container-section col-sm-11 col-md-12" id="containerSection">
        <page-head></page-head>
    </div>
</template>
<script>
//es6语法，相当于var pageHead=require('components/pageHead')
import pageHead from 'components/pageHead';
//es6语法，相当于module.exports = {}
export default {
    data: function() {
        return {
        }
    },
    components: {
    	//es6语法，相当于pageHead:pageHead
        pageHead
    },
    beforeRouteEnter: function(to, from, next) {
        next(vm => {
                vm.beforeroute=Math.random();
        })
    },
    mounted: function() {
        // this.beforeroute = Math.random();
    },
    watch: {
        '$route': function(from, to) {
            this.beforeroute = Math.random();
            this.content = this.getType();
        }
    },
	methods:{
        getType: function() {
	    //es6语法，var query = this.$route.query;
            let query = this.$route.query;
            return query.type || 'list';
        },
        checkType: function(type) {
            if (type === this.content) {
                return true;
            }
            return false;
        }
    }
}
</script>
<style>
.main-section {
    color: green;
}
</style>
```
5. 组件的写法与上面的vue文件一致，.vue文件就是一个单文件组件。
现在已经有一些公共组件，放在components目录下，pageHead（小标题）、table（表格）、datepicker（日历）、item（筛选框和按钮）、tablePage(table+item组合）
