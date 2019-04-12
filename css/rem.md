## rem简介
rem是先对与html根节点的字体大小。

## rem的运用

## 使用时注意
在使用rem的时候需要注意的是当rem计算出来的值不是一个整数（主要是不是手机的物理像素的整数倍），那么就会出现取舍。  

异常状况举例（rem布局）： 
```
.invitefriend-top {
    .invitefriend-top-header{
        position: relative;
        height: 5.64rem;
        background: url('/img/lcstwo/header_pt.jpg') no-repeat;
        background-size: 100%;
    }
}
```
图片宽高比为：750*564，对应在手机端为：7.5rem*5.64rem。
正常展示：


background-size：100%;相当于background-size：100% auto;由于5.64rem对应到部分手机上会有取舍，和7.5rem真正的比例不是750*564，而是有细微差别。比如手机分辨率为2160x1080,1080为横向物理像素点数量，那么invitefriend-top-header元素的真实高度像素为1080*564/750 = 812.16px，可能被处理为813px。而图片比例依然用750*564那么图像显示时的宽高比为