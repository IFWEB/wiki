## http请求


## post请求

### get请求和post的请求的区别

### post请求服务如何知道数据内容传输完毕
两种方式有区别：  
**没有使用分块编码**：content-length就标明body的长度。
**如果使用分块编码**：  
1. 在头部加入 Transfer-Encoding: chunked 之后，就代表这个报文采用了分块编码。这时，报文中的实体需要改为用一系列分块来传输。
2. 每个分块包含十六进制的长度值和数据，长度值独占一行，长度不包括它结尾的 CRLF(\r\n)，也不包括分块数据结尾的 CRLF。
3. 最后一个分块长度值必须为 0，对应的分块数据没有内容，表示实体结束。



