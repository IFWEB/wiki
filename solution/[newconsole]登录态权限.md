# 登录态
  nodeConsole依然使用redis使用session方式实现登录态。
## 什么是session？
  1、存在于服务器端，不同于Cookie
  2、浏览器在访问服务器的时候，将自己的sessionid给服务器，服务器就可以根据这个ID去匹配查找Session信息，，Session的存储机制是key=value的形式。
  3、如果Session的实现是依赖于Cookie的实现，浏览器在多次访问服务器端时，会在请求报文中的Cookie属性中添加JESSIONID=123xxx，作为到服务器端检索属于自己的Session信息的唯一ID.
## 什么是redis？
  Redis的的是完全开源免费的，遵守BSD协议，是一个高性能的键值数据库。称为数据结构服务器。这个问题在大并发，高负载的网站中必须考虑.redis数据库中的所有数据都存储在内存中。由于内存的读写速度远快于硬盘，因此Redis的的的在性能上对比其他基于硬盘存储的数据库有非常明显的优势大并发的情况下，所有的请求直接访问数据库，数据库会出现连接异常。这个时候，就需要使用的的Redis的做一个缓冲操作，让请求先访问到的Redis的的，而不是直接访问数据库。
  1，运行在内存，
  2，数据虽在内存，但是提供了持久化的支持，即可以将内存中的数据异步写入到硬盘中，同时不影响继续提供服务
  3，支持数据结构丰富（string（字符串），list（链表），set（集合），zset（sorted set - 有序集合））和Hash（哈希类型，md5加密出来的那个串）
# 用户登录
## step1: 登录。 
  前端提交userName、password和验证码信息，校验验证码正确后，调用dubbo登录服务判断是否登录;若登录失败，返回给前端密码错误；若登录成功，dubbo 返回用户个人信息useInfo和用户权限列表uriList，进行下一步。

## step2: 存储userInfo、uriList、isLogin
  产生一个通用唯一标识符作为session; 将userInfo、usiList、isLogin存储至redis，并将过期时间设置为30分钟。存储的key和value分别为：
  
  
  
| key   | value   |    | 
|:----|:----|:----|
| userInfo:userId   | userInfo   | userId为用户id,userInfo为dubbo返回的用户个人信息，其中包含插入的uuid   | 
| uriList:userId   | uriList   | userId为用户id,userList为dubbo返回的用户权限列表   | 
| isLogin:userId   | session   | userId为用户id,session为产生的唯一标识符   | 

 
## step3: 签名session并设置cookie。
  使用secretKey签名session等到签名之后的signedSession。这里的secretKey存储在node-console服务内存中。设置cookie,并将cookie过期时间设为30分钟,set-cookie:session=signedSession;HttpOnly;maxAge:30*60*1000，返回给前端：登录成功。

# 用户鉴权
## step1: 验证session是否有效且未过期
  前端cookie中携带session请求node-cosole服务；先使用secretKey验证session。若验证失败，表示sessin无效，返回给前端：未登录，若验证成功，表示session有效，查询redis中是否存在userInfo:session，若存在，表示session未过期，继续进行下一步，若不存在，表示session已过期，返回前端：登录已过期。
 
## step2: 验证当前登录是否是最近一次登录
  session验证成功后，从redis读取到userInfo,就能获取当前userId,查询isLogin:userId的session与验证过的session是否相等，若相等，表示当前登录是最近一次登录，进行下一步，若不相等，表示当前登录不是最近一次登录，返回给前端:账号在另一个客户端登录，当前登录被迫下线。

## step3: 判读当前用户是否有权限访问该接口
  查询redis中该uriList:session是否包含该请求的req.path；若包含，则表示用户有权限访问该接口，进行相应的业务处理，再进入下一步；若不包含，则表示用户没有权限访问该接口，不进行业务处理，直接进行下一步。

## step4: 更新redis中userInfo、uriList和isLogin过期时间，更新cookie中session的过期时间   

![图片](https://github.com/IFWEB/wiki/blob/master/img/%E7%99%BB%E5%BD%95%E6%80%81/console.png)
