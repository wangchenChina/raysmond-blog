---
title: Ubuntu下Nginx配置和使用SSL证书
date: 2015-10-02 23:57:02
tags:
  - Linux
  - SSL
  - Ubuntu
  - 安全
---

最近SSL证书真的是越来越便宜，想当初买Rapidssl证书的时候一年40几刀。现在我通过 https://www.gogetssl.com 渠道购买Rapidssl三年只要20刀，简直白菜价。

Secure Sockets Layer (SSL)的作用就是在网站和用户之间建立一个安全可靠的会话。HTTP协议默认是不加密的，什么密码，邮件，个人信息都明文传输，好可怕啊。从去年起，我就有SSL情节，不是HTTPS的网站都不怎么敢注册。现在我的两个网站( https://raysmond.com 和 https://raysnote.com )上了HTTPS。

不多说了，要买SSL证书的同学可以到 https://www.gogetssl.com 这里买，比较便宜。下面说一下如何在Ubuntu服务器上给Nginx配置和使用SSL证书。

购买和配置证书

首先生成CSR文件。

```bash
sudo mkdir /etc/nginx/ssl
cd /etc/nginx/ssl

# generate private key, please remember the pass pharse you input
sudo openssl genrsa -des3 -out server.key 2048

# generate csr
sudo openssl req -new -key server.key -out server.csr
```

然后就是购买SSL证书，这个不多说了，一步步跟着网站走。不过好像要使用到自定义域名邮箱，比如admin@raysmond.com，用于接收邮件。假设你已经收到了发来的SSL证书邮件。把证书内容输入到`server.crt`中。大概内容就是：

```
-----BEGIN CERTIFICATE-----
证书内容
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
证书内容
-----END CERTIFICATE-----
```

然后把private key中的passpharse去掉。

```bash
sudo cp server.key server.key.original
sudo openssl rsa -in server.key.original -out server.key
```

Nginx中配置和使用SSL证书

修改Nginx配置文件``/etc/nginx/sites-enabled/default`，加上：

```
server {
    listen 80;
    listen [::]:80;
    server_name raysmond.com; # 改成你的域名
    client_max_body_size 4G;

    # enable ssl
    listen 443 default ssl;
    ssl on;
    ssl_certificate /etc/nginx/server.crt;
    ssl_certificate_key /etc/nginx/server.key;

    # 如果是http的话，强制转跳到https
    if ($scheme = http) {
       return 301 https://$server_name$request_uri;
    }

    ssl_session_cache    shared:SSL:5m;
    ssl_session_timeout  5m;
    keepalive_timeout   70;
    ssl_protocols       SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    # ssl_ciphers  HIGH:!aNULL:!MD5;
    # ssl_prefer_server_ciphers  on;

    # end of ssl


    # 下面的网站配置内容根据自己的实际情况配置
    root /usr/share/nginx/html;
	index index.html index.htm;
}
```
