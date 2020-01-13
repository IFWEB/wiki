## ios中设置为readonly的input框还是能上光标

#### 现象和原因分析
input节点readonly的input框能上光标，而且光标总是显示在页面最顶层（z-index超大）。

#### 解决方案
1. \<input type="text" readonly unselectable="on" onfocus="this.blur()">。
2. 使用disabled属性代替，不过disabled样式和readonly会有差别需要处理。
