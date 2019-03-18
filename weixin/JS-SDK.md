微信JS-SDK是微信公众平台 面向网页开发者提供的基于微信内的网页开发工具包。

通过使用微信JS-SDK，网页开发者可借助微信高效地使用拍照、选图、语音、位置等手机系统的能力，同时可以直接使用微信分享、扫一扫、卡券、支付等微信特有的能力，为微信用户提供更优质的网页体验。

JSSDK使用步骤

步骤一：绑定域名(这一步在公众号设置)

     先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”。
     备注：登录后可在“开发者中心”查看对应的接口权限。
     （weixin.lcfarm.com）

步骤二：引入JS文件（前端引入）
      我们平台通过html中引入下面link来完成
      <link rel="import" href="/common/html/wxShareInit/wxScript.html?__inline">
      最终引入的是1.4.0版本
	  <script type="text/javascript" src="//res.wx.qq.com/open/js/jweixin-1.4.0.js"></script>

步骤三：通过config接口注入权限验证配置(后台完成)

		所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用）。
		wx.config({
		    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
		    appId: '', // 必填，公众号的唯一标识
		    timestamp: , // 必填，生成签名的时间戳
		    nonceStr: '', // 必填，生成签名的随机串
		    signature: '',// 必填，签名
		    jsApiList: [] // 必填，需要使用的JS接口列表
		});

步骤四：通过ready接口处理成功验证（前端完成）

	   我们平台通过/common/js/mod/share.js进行了组件封装,以便在应用时可以直接调用
	   // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
	   
	   wx.ready(function() {
            wx.hideMenuItems({ //在这里配置各个分享列表，配置好的就是可以分享的
                menuList: data.hideMenuItems || [
                        "menuItem:share:qq",
                        "menuItem:share:QZone",
                        "menuItem:share:weiboApp",
                        "menuItem:jsDebug",
                        "menuItem:editTag",
                        "menuItem:delete",
                        "menuItem:originPage",
                        "menuItem:readMode",
                        "menuItem:share:email",
                        "menuItem:share:brand",
                        "menuItem:favorite"
                    ] // 要隐藏的菜单项，只能隐藏“传播类”和“保护类”按钮，所有menu项见附录3
            });
            exports.bindWeixinEvent(data);
        });

        bindWeixinEvent: function(data) {//这里传递具体的分享内容，标题，图片，描述等
        var share = data;
        share.timeline = data.timeline;
        share.title = data.title;
        share.timeline = data.timeline;
        share.des = data.des;
        share.imgUrl = data.imgUrl;
        share.linkUrl = data.linkUrl; //未登录分享出去的是一个新的宣传页面

        data.channels = data.channels || {
            'Timeline': '',
            'AppMessage': '',
            'QQ': '',
            'QZone': ''
        };
        wx.onMenuShareTimeline({
            title: share.timeline || share.title, // 分享标题
            link: addParam(share.linkUrl, {
                'channel': data.channels.Timeline
            }), // 分享链接
            imgUrl: share.imgUrl, // 分享图标
            success: function() {
                // 用户确认分享后执行的回调函数
                data.success && data.success();
                report.send('H5-share', 'H5-成功分享到朋友圈', 'H5-share3');
            },
            cancel: function() {
                // 原来没有，不知道有没有效
                data.cancel && data.cancel();
            }
        });

        wx.onMenuShareQQ({ 
            title: share.title, // 分享标题
            desc: "我在布谷农场投资，给你发加金福利和新手红包啦！", // 分享描述
            link: addParam(share.linkUrl, {
                'channel': data.channels.QQ
            }), // 分享链接
            imgUrl: share.imgUrl, // 分享图标
            success: function() {
                // 用户确认分享后执行的回调函数
                data.success && data.success();
                report.send('H5-share', 'H5-成功分享到朋友圈', 'H5-share3');
            },
            cancel: function() {
                // 用户取消分享后执行的回调函数
                data.cancel && data.cancel();
            }
        });
    }

步骤五：通过error接口处理失败验证

wx.error(function(res){
    alert(JSON.stringify(res));
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
});