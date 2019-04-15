
## DNS
正常用户都是通过域名访问服务器，如www.baidu.com。DNS服务器解析www.baidu.com的服务器ip返回给浏览器，浏览器再和服务器ip进行TCP连接。  
使用**nslookup**查看DNS解析结果,比如查看百度的  
```
C:\Users\Administrator>nslookup www.baidu.com
服务器:  public1.114dns.com
Address:  114.114.114.114

非权威应答:
名称:    www.a.shifen.com
Addresses:  14.215.177.39
          14.215.177.38
Aliases:  www.baidu.com
```
百度有两台服务器，随便在浏览器上访问一台：http://14.215.177.38。访问是没有问题的。http默认端口是80，https默认端口是443。   
从上面可以看到www.baidu.com实际是www.a.shifen.com的别名

## DNS劫持