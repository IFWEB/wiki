### -----svn新建文件总是被忽略
**原因分析**   
1. 全局忽略，查看Tortoise SVN--Settings--General 右边--Global ignore pattern，如果全局忽略中有配置则去除这个匹配。
（但是往往不是，普通情况没有人会去动这个配置）  
2. 局部忽略，查看Tortoise SVN--Properties查看属性，如果有忽略项，删除即可。
