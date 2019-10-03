---
title: 使用 Certbot 安装 Letsencrypt 证书
date: 2019-01-05 13:33:25
tags:
- Nginx
- HTTPS
---

**概述**
本文介绍如何通过 Certbot 安装 Https Letsencrypt 证书

**先决条件**
* 拥有一个域名，例如 pachiuba.com
* 在域名服务器创建一条A记录，指向云主机的公网IP地址。例如 pachiuba.com 指向 192.168.0.1 的IP地址
* 要等到新创建的域名解析能在公网上被解析到

**安装 Certbot**
![Certbot](https://pachiuba.com/img/Certbot.png)

<section class="rnrn"></section>

或者直接获取自动安装脚本，然后在按如下两种模式生成证书

```js
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto # 给脚本执行权限
```

## **Certbot 两种生成证书的方式**

**certbot 模式（推荐）**

certbot 会启动自带的 nginx（如果服务器上已经有nginx或apache运行，需要停止已有的nginx或apache）生成证书

```js
./certbot-auto certonly -d example.com -d www.example.com
//或者
./certbot-auto certonly -d www.example.com
```

**webroot 模式**
* 1、配置验证目录
```js
server {
  listen 80;
  server_name 127.0.0.1;
  location / {
    root   /var/www/example;
    index  index.html;
  }
}
```

* 2、重启 nginx
```js
nginx -t // 检查nginx配置文件是否正确
nginx -s reload // 使配置生效
service nginx restart // 重启 nginx
```

* 3、执行 certbot 脚本生成证书
```js
certbot certonly --webroot -w /var/www/example/ -d www.example.com -d example.com -w /var/www/other -d other.example.net -d another.other.example.net
```

certbot会生成随机文件到给定目录(nginx配置的网页目录)下的/.well-known/acme-challenge/目录里面，并通过已经启动的nginx验证随机文件，生成证书

**证书应用**
通过以上方式生的成证书及 privkey 等文件一般位于 /etc/letsencrypt/live/example.com/ 下：

| 文件 | 描述 |
| ------ | ------ |
| cert.pem | 服务器证书 |
| chain.pem | 包含Web浏览器为验证服务器而需要的证书或附加中间证书 |
| fullchain.pem | cert.pem+chain.pem |
| privkey.pem | 证书的私钥 |

**打开防火墙443端口**
```js
//打开
firewall-cmd --permanent --add-port=443/tcp
//检查443是否开启
firewall-cmd --permanent --query-port=443/tcp
//重启防火墙
firewall-cmd --reload
```

### Nginx配置
在http{}中写入两个server
```js
server
    {
        listen 80;
        server_name pachiuba.com;
        //强制重置到443端口
        rewrite ^(.*)$  https://$host$1 permanent;
    }
    server
  	{
        #监听443端口
        listen 443 ssl;
        server_name pachiuba.com;
        #ssl on;
        // 安装证书的位置
        ssl_certificate /etc/letsencrypt/live/pachiuba.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/pachiuba.com/privkey.pem;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
        location / {
            root /www/wwwroot/www.pachiuba.com;
            index index.html;
        }
    }
```