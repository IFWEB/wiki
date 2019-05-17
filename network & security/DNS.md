
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
从上面可以看到www.baidu.com实际是www.a.shifen.com的别名,这个别名也叫CName。CName记录允许您将多个名字映射到同一台计算机。通常用于同时提供WWW和MAIL服务的计算机。例如，有一台计算机名为kkkssmm.。 它同时提供WWW和MAIL服务，为了便于用户访问服务。可以为该计算机设置两个别名：A和B。  

这样的好处是，变更服务器IP地址的时候，只要修改www.a.shifen.com这个域名就可以了，用户的www.baidu.com域名不用修改。  

由于CNAME记录就是一个替换，所以域名一旦设置CNAME记录以后，就不能再设置其他记录了  

（比如A记录和MX记录），这是为了防止产生冲突。 

## DNS劫持
DNS劫持又称域名劫持,是指通过某些手段取得某域名的解析控制权，修改此域名的解析结果，导致对该域名的访问由原IP地址转入到修改后的指定IP，其结果就是对特定的网址不能访问或访问的是假网址。  

如果可以冒充域名服务器，然后把查询的IP地址设为攻击者的IP地址，这样的话，用户上网就只能看到攻击者的主页，而不是用户想要取得的网站的主页了，这就是DNS劫持的基本原理。  

DNS劫持其实并不是真的“黑掉”了对方的网站，而是冒名顶替、招摇撞骗罢了。  


