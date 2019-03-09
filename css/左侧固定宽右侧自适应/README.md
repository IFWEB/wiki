所有的布局方法规定左侧固定宽度为200px。右侧自动适应。  
左右侧通用样式如下
```
body{
    color: #fff;
}
.left{
    width: 200px;
    background: red;
}
.right{
    background: blue;
}
```
# 方法一：左侧浮动方案,右侧利用div自动基充父元素特性自适应
主要有两种子方案
## 右侧设置margin-left
```
.float1 .left{
    float: left;           
}
.float1 .right{
    margin-left: 200px;         
}
```
## 右侧设置overflow: auto;利用BFC（块级格式化上下文）来实现
什么是BFC?
```
.float2 .left{
    float: left;
}
.float2 .right{
    overflow: auto;/*这一块需要添加，否则当float2-right整体宽度是全屏的宽度，虽然文字不会和左边块重叠。在IE和Firefox上问题比较明显*/
}
```

# 方法二：左侧绝对定位方案，右侧利用div自动基充父元素特性自适应
这个方案和第一个方案很类似
```
.position .left{
    position: absolute;
}
.position .right{
    margin-left: 200px;
}
```

# 方法三：父元素设置为table,两个子元素分别为表格列，第一列固定宽度
```
.table{
    display: table;
    width: 100%;
}
.table .left{           
    display: table-cell;
}
.table .right{
    display: table-cell;
}
```

# 方法四：双浮动布局，右侧使用calc来计算实现
```
.calc .left{
    float: left;
}
.calc .right{
    float: right;
    width:calc(100% - 200px);
}
```
calc是css3属性，ie9+支持。
>实际上两边可以不用浮动，用双内联块display:inline-block也可以，但是需要解决幽灵空白问题，更复杂

# 方法五：flex布局
```
.flex {
    display: flex;
    align-items: flex-start;
}
.flex .left { 
    flex: 0 0 auto;
}
.flex .right {
    flex: 1 1 auto;
}
```
flex布局ie10+支持

# 方法六： grid布局
```
.grid {
    display: grid;
    grid-template-columns: 200px 1fr;
    align-items: start;
}
.grid .left {
    box-sizing: border-box;
    grid-column: 1;
}
.grid .right {
    box-sizing: border-box;
    grid-column: 2;
}
```
grid布局ie支持比较差，必须edge16+才支持的比较好  


<a href="https://github.com/IFWEB/wiki/blob/master/css/左侧固定宽右侧自适应/demo.html" target="_blank">demo就在该目录下的demo.html</a>  
上面几个布局效果如下   
![左侧固定右侧自适应](https://github.com/IFWEB/wiki/blob/master/css/左侧固定宽右侧自适应/show.png)  
