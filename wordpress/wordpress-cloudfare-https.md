# wordpress使用cloudfare开启https后重定向过多

使用letsEncrypt为网站申请https证书，就想着开始http自动重定向到https
参考网上教程都是修改301重定向，

### 修改 nginx 配置文件

添加 两行

```
server_name iruach.com www.iruach.com;
return 301 https://$server_name$request_uri;
```

如下

```
server {
        listen 80;
        #server_name _;
        server_name iruach.com www.iruach.com;
        return 301 https://$server_name$request_uri;
        #rewrite ^(.*) https://$server_name$1 permanent;
        }
```

### 重启nginx 

`nginx -s reload`

### 访问网站
http://iruach.com

显示重定向次数过多 注释掉`return 301 https://$server_name$request_uri;`

一切正常 反复很多次 清除cookie 也不行 后来想起可能是cloudfare引起的，网上一查，果然如此

## CloudFlare 造成重定向的次数过多的原因

当网站开启了 CloudFlare 服务，用户访问我们的网站时，其实访问的离用户比较近的 Cloudflare 服务器，Cloudflare 再代理用户请求我们的源服务器，以达到加速和保护源服务器的目的。Cloudflare 代理用户请求我们源服务器获取网页资源的过程叫回源。

Cloudflare 造成循环重定向的错误就出在了回源的过程中，造成这种错误的原因就是 http 和 https 之间的重定向。

Cloudflare Crypto 的 SSL 中有 4 个选项（如下），其中 Off 就是不启用 SSL，通过 HTTP 协议访问网站。另外 3 种是通过 HTTPS 协议访问网站。

### Cloudflare CDN 配置

* Flexible：当我们的源网站没有配置 HTTPS 支持时，启用这个选项，Cloudflare 会在回源的时候通过 HTTP 协议访问我们的网站。
* Full：当我们的源网站支持 HTTPS，但是 HTTPS 证书和域名不匹配或者是自签名证书时，Cloudflare 会通过 HTTPS 协议访问源网站，但不会验证证书，也就是说，即使我们的源网站提供的 HTTPS 证书不受浏览器信任，Cloudflare 也会通过 HTTPS 回源网站。
* Full(strict)：当我们的源网站支持 HTTP ，并且证书有效时（未过期且受信任）。Cloudflare 会通过 HTTPS 协议访问源网站，并在每个请求过程中验证证书。


了解了上面各个设置的功能，我们来看一下 Cloudflare 的循环重定向问题是怎么出现的，在 Cloudflare 中开启了 SSL 后，访问网站时出现循环重定向需满足下面两个条件：


* SSL 中设置了 Flexible，CDN 以 HTTP 协议回源网站。
* 源网站支持 HTTPS，并且设置了通过 HTTP 协议访问时，自动跳转到 HTTPS 协议。

到这里，可能就有朋友发现问题了，我们访问 Cloudflare 的 CDN 服务器的时候，是通过 HTTPS 访问的，CDN 访问源网站的时候，是通过 HTTP 访问的，源网站上 HTTP 又自动跳转了 HTTPS，完美的一个循环重定向。重定向的次数多了，浏览器就撂挑子报出了 ERR_TOO_MANY_REDIRECTS 的错误。

### CloudFlare 造成重定向的次数过多问题的解决办法

知道了循环重定向的原因，我们也就知道了怎么解决这个问题，通过测试，下面的两种设置方法都可以解决 Cloudflare 循环重定向的问题。

* SSL 中选择 Full(strict) 或者 Full(strict)，让 CDN 回源的时候使用 HTTPS 的方式回源，没有 HTTP 什么事了，就不会跳来跳去了
* 源网站不设置 HTTPS 支持或者 不设置 HTTP 跳转 HTTPS，让 Cloudflare 回源的时候使用 HTTP 方式获取资源。

修改了 CloudFlare 设置后，可能需要过几分钟或清理浏览器缓存后才能生效。

除了 Cloudflare，使用其他 CDN 提供商的时候，也可能会出现这个问题，如果设置了 CDN 后，遇到了 Chrome 报重定向次数过多的问题，可以通过上面的思路查找问题。如果你在使用其他 CDN 的时候也遇到了类似的问题，欢迎在评论中提出，让更多的朋友看到。

#### 参考链接

1 [HTTPS 升级指南](http://www.ruanyifeng.com/blog/2016/08/migrate-from-http-to-https.html)

2 [WordPress 网站使用 CloudFlare 后提示“将您重定向的次数过多” 的原因及解决办法](https://www.wpzhiku.com/wordpress-wang-zhan-shi-yong-cloudflare-hou-ti-shi-jiang-nin-chong-ding-xiang-de-ci-shu-guo-duo-de-yuan-yin-ji-jie-jue-ban-fa/)

