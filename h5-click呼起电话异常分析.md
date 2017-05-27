### 呼起电话链接\<a href="tel:11111">联系客服\</a>
呼起电话是这个标签的默认功能，响应click事件时触发。当文案"联系客服"的父节点（除开document外）绑定click事件的处理中执行e.preventDefault();会导致呼起电话失败。
### 解决方案
1.去除e.preventDefault()
2.使用其他事件，比如tap事件代替click来执行想要的处理，保证click通畅。
