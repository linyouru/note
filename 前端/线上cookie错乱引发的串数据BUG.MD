# AWStudio线上cookie错乱引发的串数据BUG

## 现象：
用户登录后随机点击其他页面，有概率会获取其他用户的数据，一旦获取到其他用户的数据后请求到的都是该用户的数据；刷新页面会导致掉线；该现象只在web页面出现，使用swagger API或postmanm都不能复现该现象；

## 原因分析：
1、观察浏览器控制台，登录后响应的token一开始是正确的，但点击其他页面后发现请求上报cookie中的token变化，原因在静态资源响应头中存在**Set-Cookie**，又因为使用了nignx缓存，导致最开始登录的用户的静态资源被缓存；后续登录的用户在切换页面时若最后一个请求为被缓存的静态资源，浏览器就会因为该静态资源响应头中的**Set-Cookie**覆盖当前cookie中的token，从而导致请求到其他用户数据，刷新被登出同理；  
2、为什么静态资源响应头中会存在**Set-Cookie**?因为AWStudio的静态资源是使用koa-static作为静态资源服务器，请求静态资源需经过网关。而网关中使用的JWTSession会给每个请求加上**Set-Cookie**。

## 处理方案：
1、nignx缓存静态资源中加上“proxy_hide_header Set-Cookie;”配置，不传输网关设置的“Set-Cookie”；  
2、只针对帮助文档设置缓存，AWStudio的缓存一开始只是为了解决帮助文档加载缓慢的问题；  
3、使用nginx作为静态资源服务器，静态文件不过网关，自然就没有上述问题；  

考虑时间成本，目前采用方案1处理该bug。

## 参考资料：
https://www.nginx.cn/doc/standard/httpproxy.html  
https://serverfault.com/questions/641417/nginx-not-processing-proxy-hide-header-and-proxy-ignore-header-directives#:~:text=Directive%20proxy_ignore_headers%20tells%20nginx%20to%20ignore%20the%20content,performs%20an%20internal%20redirect%20to%20the%20specified%20URI%3B  