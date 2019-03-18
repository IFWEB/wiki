网页授权流程分为四步：

1、引导用户进入授权页面同意授权，获取code

2、通过code换取网页授权access_token（与基础支持中的access_token不同）

3、如果需要，开发者可以刷新网页授权access_token，避免过期

4、通过网页授权access_token和openid获取用户基本信息（支持UnionID机制）

第一步：用户同意授权，获取code（前端跳转）

https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx2ebfe5f20389c64a&redirect_uri=encodeURIComponent('http:')%2F%2Fm.lcfarm.com%2Factivity%2F2019fourYears%2Fziweiguan%2Fziweiguan.html%3F&response_type=code&scope=snsapi_userinfo&state=STATE&connect_redirect=1#wechat_redirect

参数 	    是否必须	 说明
appid	        是	 公众号的唯一标识
redirect_uri	是	 授权后重定向的回调链接地址， 请使用 urlEncode 对链接进行处理
response_type	是	 返回类型，请填写code
scope	        是	 应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo 													（弹出授权页面，可通过openid拿到昵称、性别、所在地。并且， 即使在未关注的情况下，只要用户授权，也能获取其信息 ）
state	        否   重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
#wechat_redirec 是	 无论直接打开还是做页面302重定向时候，必须带此参数

如果用户同意授权，页面将跳转至回跳地址
code说明 ： code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。


第二步：通过code换取网页授权access_token（后台处理）
首先请注意，这里通过code换取的是一个特殊的网页授权access_token,与基础支持中的access_token（该access_token用于调用其他接口）不同。公众号可通过下述接口来获取网页授权access_token。如果网页授权的作用域为snsapi_base，则本步骤中获取到网页授权access_token的同时，也获取到了openid，snsapi_base式的网页授权流程即到此为止。

尤其注意：由于公众号的secret和获取到的access_token安全级别都非常高，必须只保存在服务器，不允许传给客户端。后续刷新access_token、通过access_token获取用户信息等步骤，也必须从服务器发起。

第三步：刷新access_token（如果需要、后台处理）

由于access_token拥有较短的有效期，当access_token超时后，可以使用refresh_token进行刷新，refresh_token有效期为30天，当refresh_token失效之后，需要用户重新授权。

请求方法
获取第二步的refresh_token后，请求以下链接获取access_token：
https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN

第四步：拉取用户信息(前端请求AJAX)

通过access_token、openid就可以请求拿到用户信息

参数	描述
access_token	网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
openid	用户的唯一标识
lang	返回国家地区语言版本，zh_CN 简体，zh_TW 繁体，en 英语

返回的用户信息如下

参数			描述
openid		用户的唯一标识
nickname	用户昵称
sex			用户的性别，值为1时是男性，值为2时是女性，值为0时是未知
province	用户个人资料填写的省份
city		普通用户个人资料填写的城市
country		国家，如中国为CN
headimgurl	用户头像，最后一个数值代表正方形头像大小（有0、46、64、96、132数值可选，0代表<640>																	</640>640正方形头像），用户没有头像时该项为空。若用户更换头像，原有头像URL将失效。
privilege	用户特权信息，json 数组，如微信沃卡用户为（chinaunicom）
unionid		只有在用户将公众号绑定到微信开放平台帐号后，才会出现该字段。