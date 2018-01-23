---
title: 微信小程序本地开发环境搭建
date: 2018-01-23 11:39:28
tags:
- 微信小程序
categories:
- 微信小程序
---

> 前段时间开发一个微信小程序项目，需要本地开发接口，而微信开发工具只能用HTTPS请求，所以不得不配置本地代理；开始在网上找了个教程用Charles这个代理工具，感觉用着有点麻烦，所以换成nginx；

<!-- more -->

#####  申请AppID

​    这个直接在[公众号平台](https://mp.weixin.qq.com)申请就可以了；

##### 配置域名

  在[公众号平台](https://mp.weixin.qq.com)开发者设置>服务器配置中填写域名信息，这个只能填写真实域名，还需要SSL支持；

#### 本地开发环境配置

   **这里主要说的是Windows环境，其他Lunix、MAC也大同小异**

#####1.修改Hosts

​     首先找到Hosts文件： C:\Windows\System32\drivers\etc\hosts

然后添加下面这行到hosts，将域名映射到本地：

```
127.0.0.1 www.yourdomain.com
```

##### 2.生成自签名SSL证书

首先需要安装openssl [【点击这里下载】](http://slproweb.com/download/Win64OpenSSL-1_1_0g.exe)，安装好后需要将其配置到环境变量（如果已经安装过openssl自行忽略...）；

__打开命令行工具__

1. 生成private.key

   输入命令：

   ```tcl
   openssl genrsa -des3 -out server.key 1024
   ```

2. 生成CSR (Certificate Signing Request)

   输入命令：

   ```
   openssl req -new -key server.key -out server.csr
   ```
   这里会要求输入一些地址信息，看着写就好：

   ```
   Loading 'screen' into random state - done
   You are about to be asked to enter information that will be incorporated
   into your certificate request.
   What you are about to enter is what is called a Distinguished Name or a DN.
   There are quite a few fields but you can leave some blank
   For some fields there will be a default value,
   If you enter '.', the field will be left blank.
   -----
   Country Name (2 letter code) [AU]:ZH  【国家代码】
   State or Province Name (full name) [Some-State]:HB【省份】
   Locality Name (eg, city) []:WH【城市】
   Organization Name (eg, company) [Internet Widgits Pty Ltd]:cxsky【组织】
   Organizational Unit Name (eg, section) []:cxsky【组织】
   Common Name (e.g. server FQDN or YOUR name) []:*.yourdomain.com 【可以填域名】
   Email Address []:【Enter】

   Please enter the following 'extra' attributes
   to be sent with your certificate request
   A challenge password []:【Enter】
   An optional company name []:【Enter】
   ```

3. 移除Passphrase

   输入命令：

   ```
   cp server.key server.key.org
   openssl rsa -in server.key.org -out server.key 
   ```

4. 生成自签名证书

   输入命令

   ```
   openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
   ```

   ​


完成了以上4步后，将server.crt和server.key移到你想要存放证书的地方。

##### 3.安装和配置nginx

   完成上面步骤后，接下来配置nginx支持HTTPS，[【下载nginx Windows版】](http://nginx.org/en/download.html)，修改 nginx配置文件nginx-1.13.8\conf\nginx.conf，

```
server {
        listen       443 ssl;
        server_name  www.yourdomain.com;

        ssl_certificate      D:/nginx-1.13.8/ssl/server.crt;
        ssl_certificate_key  D:/nginx-1.13.8/ssl/server.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

		ssl_protocols SSLv3 TLSv1 TLSv1.2 TLSv1.1;
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

        location / {
            proxy_pass http://127.0.0.1:8080; # 本地服务器地址及端口
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header Host $host;
			proxy_set_header X-Forward-Proto https;
			proxy_http_version 1.1;
			# for websocket
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
        }
    }
```

##### 4.测试

​    这个时候在微信小程序IDE，就可以直接向本地发送HTTPS请求了，请求最终指向的是http://127.0.0.1:8080 

最后如果小程序IDE提示TLS版本过低，请查看nginx.conf 中server下的```ssl_protocols``` 是否配置正确，获取直接在小程序IDE中去掉验证TLS版本的勾选

![weixinapp](http://osassnq6x.bkt.clouddn.com/winxinapp.png)

 