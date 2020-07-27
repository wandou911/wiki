# Let's Encrypt 入门教程

## 证书安装与更新

### 1 安装客户端

```
[root@vultr ~]# yum install letsencrypt
```

Let's Encrypt 客户端名称可能是 letsencrypt 或 certbot. 请用 which 指令看看你属于哪一种情形。后文将统一使用 letsencrypt 指令，如有需要请自行替换成 certbot.


### 2 获取证书

```
[root@vultr certbot]# service nginx stop
Stoping nginx...  done
[root@vultr certbot]# letsencrypt certonly --standalone
[root@vultr certbot]# service nginx start
Starting nginx...  done
```

假设你已经获取了 example.com 的域名，并且其 DNS A Record (不能是CNAME) 指向了当前服务器的 IP 地址。Let's Encrypt 客户端会通过查询 DNS 信息验证服务器的身份。在这里，我推荐使用独立模式（standalone）获取和更新证书。

由于客户端要使用 80 或 443 端口，请首先关闭占用此端口的程序。通常停止 Nginx 即可。

证书的位置:`/etc/letsencrypt/live`

### 3 配置nginx

如果你已经配置好了 HTTP 服务，/etc/nginx/nginx.conf 文件中应该有类似下面的片段。

```
http {
    server {
        listen 80; 
        listen [::]:80;
        server_name example.com www.example.com;
        location / {
            root /var/www/html;
        }
        ...
    }
    ...
}
```

你需要再加上一个 server 条目用于 HTTPS 服务。改完之后的结果是这个样子。

```
http {
    server {
        listen 80; 
        listen [::]:80;
        server_name example.com www.example.com;
        location / {
            root /var/www/html;
        }
        ...
    }
    server {
        listen 443 ssl;
        server_name example.com www.example.com;
        ssl_certificate /etc/letsencrypt/live/example.com/cert.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
        location / {
            root /var/www/html;
        }
        ...
    }
    ...
}
```

记得把所有 example.com 替换成你自己的域名。

保存配置文件，启动 Nginx.

`sudo service nginx start`


恭喜你已经成功部署了 Let's Encrypt 证书！

### 4 certbot更新证书 

Let's Encrypt 证书有效期为三个月，你需要在证书到期之前续约。官方会在证书到期前向你发送邮件，因此不必担心错过日期。你可以运用 cron 实现证书的自动更新，不过手动操作已经足够简单了。

```
[root@vultr certbot]# service nginx stop
Stoping nginx...  done
[root@vultr certbot]# certbot renew
Saving debug log to /var/log/letsencrypt/letsencrypt.log

[root@vultr certbot]# service nginx start
Starting nginx...  done
```

### 5  使用certbot命令删除证书

进入 /etc/letsencrypt/renewal 目录

```
[root@vultr ~]# cd /etc/letsencrypt/renewal

[root@vultr live]# ls
README  www.xiaokeli.me

[root@vultr renewal]# certbot delete --cert-name www.xiaokeli.me
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Deleted all files relating to certificate www.xiaokeli.me
```

## 进阶：HTTPS 跳转

现在，你的服务器同时接受 HTTP 和 HTTPS 请求。如果你希望只受理 HTTPS 请求，可以在 nginx.conf 中添加一个 301 跳转规则，告知浏览器将 HTTP 变成 HTTPS.

```
http {
    server {
        listen 80; 
        listen [::]:80;
        server_name example.com www.example.com;
        return 301 https://$server_name$request_uri;
    }
    server {
        listen 443 ssl;
        server_name example.com www.example.com;
        ssl_certificate /etc/letsencrypt/live/example.com/cert.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
        location / {
            root /var/www/html;
        }
        ...
    }
    ...
}
```

#### 参考链接：

[Let's Encrypt 入门教程](https://bitmingw.com/2017/02/02/letsencrypt-tutorial/)
