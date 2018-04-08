# 微信Web开发踩坑 2018-04-08

### 微信jssdk

- 微信jssdk授权，注意授权时候的url不能够和页面访问的url不一致。
    - 域名需要添加到公众号后台的授权列表中。
    - 注意不要在wx.ready之前调用historyAPI修改url, 否则会导致`config:invalid signature`错误
    - 如果WebApp中有使用hash路由进行场景管理，在url中至少需要添加一个空白`?`(queryString)，否则可能出现微信支付调用失败

### 开发调试

- 静态资源和联调，把精通资源特别是`javascript`设置到一个前端可以快速更新的地址下，例如github的Page服务，coding的Page服务等
