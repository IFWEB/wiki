# 新console的map.js菜单系统介绍

标签（空格分隔）： console 前端

---

[TOC]
### map.js 来源
新console系统，菜单由后台consoleapi/menus接口下发,数据结构如下：
```` 
{
    "code": 0,
    "message": "操作成功",
    "data": {
        "resources": [
            {
                "id": 1,
                "parentId": 0,
                "label": "首页",
                "path": "homePage",
                "children": []
            },
            {
                "label": "财务管理",
                "path": "finance",
                "children": [
                    {
                        "label": "资金管理",
                        "path": "crjmanage",
                        "children": [
                            {
                                "id": 321,
                                "parentId": 300,
                                "label": "充值管理",
                                "path": "czgl",
                                "children": []
                            },
                            {
                                "id": 301,
                                "parentId": 300,
                                "label": "提现管理",
                                "path": "txgl",
                                "children": []
                            }
                        ]
                    }
                ]
            }
        }]
    }
}
````

每个菜单需要对应一个单文件组件入口，这个入口是一个单文件组件。由于webpack打包时不会执行js语句，无法识别js语句拼装，所以必须要完整路径的资源地址。
````
// 系统管理
map['_sysManage_jsgl_glygl'] = function() {
    return import ('index/sysManage/userControl/role.vue');
};
map['_sysManage_jsgl_yhzgl'] = function() {
    return import ('index/sysManage/userControl/tabtest.vue');
};
````
_sysManage_jsgl_yhzgl指的是后台下发的path拼接，分成三段:sysManage/jsgl/yhzgl，这些菜单最终是在数据库中配置的，配置数据库的我们自己做。稍后补充配置方法介绍的地址。






