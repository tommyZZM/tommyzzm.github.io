# 微信Web开发踩坑 2018-04-08

### 微信jssdk

- 微信jssdk授权，注意授权时候的url不能够和页面访问的url不一致。
    - 注意不要在wx.ready之前调用historyAPI修改url, 否则会导致`config:invalid signature`错误
    - 如果WebApp中有使用hash路由进行场景管理，在url中至少需要添加一个空白`queryString`(`?`)，否则可能出现微信支付调用失败

- 微信分享时hash路由有可能被服务器过滤掉
    - 可以通过分享链接带上`queryString`(`?myHash=xxxx`), 然后
        - 进行服务端重定向
        - 或者页面跳转之后再读取url信息在页端跳转

- 常用的微信跳转其他APP方式, (1)应用宝剪切版, (2)魔窗等跳转服务, (*)自定义schema的方式在微信不可用

### 开发调试

- 静态资源和联调，把静态资源特别是`javascript`设置到一个前端可以快速更新的地址下，例如github的Page服务，coding的Page服务等
