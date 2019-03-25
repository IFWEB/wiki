所有的布局方法规定左侧固定宽度为50px。右侧自动适应。  
左右侧通用样式如下
```
body{
    color: #fff;
    margin: 0;
    padding: 0;
}
.left{
    width: 50px;
    background: red;
}
.right{
    background: blue;
}
```
## 方法一：左侧浮动方案,右侧利用div自动基充父元素特性自适应
主要有两种子方案（后续可能需要清除浮动）   
```
//父元素使用来清除浮动，撑开父元素
.clearFloat:before,.clearFloat:after{
    content: ".";
    clear: both;
    display: block;
    font-size: 0;/*或者height: 0px*/
}
```
### 右侧设置margin-left
```
.float1 .left{
    float: left;           
}
.float1 .right{
    margin-left: 50px;         
}
```
### 右侧设置overflow: auto;利用BFC（块级格式化上下文）来实现
什么是BFC?
```
.float2 .left{
    float: left;
}
.float2 .right{
    overflow: auto;/*这是一个种让BFC中右侧不包裹左侧的方式（当右侧高度高于左侧会出现的包裹情况），overflow: hidden也可以*/
}
```

## 方法二：左侧绝对定位方案，右侧利用div自动基充父元素特性自适应
这个方案和第一个方案很类似,需要注意的是父元素的高度需要自己定义，脱离文档流的元素无法将父元素撑开
```
/*父元素定义一个最小高度，这个高度应该不小于左侧元素高度*/
.position{
    min-height: 295px;
}
.position .left{
    position: absolute;
}
.position .right{
    margin-left: 50px;
}
```

## 方法三：父元素设置为table,两个子元素分别为表格列，第一列固定宽度
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

## 方法四：双浮动布局，右侧使用calc来计算实现
```
.calc .left{
    float: left;
}
.calc .right{
    float: right;
    width:calc(100% - 50px);
}
```
calc是css3属性，ie9+支持。
>实际上两边可以不用浮动，用双内联块display:inline-block也可以，但是需要解决幽灵空白问题，更复杂

## 方法五：flex布局
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

## 方法六： grid布局
```
.grid {
    display: grid;
    grid-template-columns: 50px 1fr;
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
