---
title: "利用Nginx构建OpenAI的反向代理"
categories:
  - Blog
  - Tech
tags:
  - Nginx
  - OpenAI
  - 反向代理
  - SSL
---

## 1、安装Nginx

```bash
sudo apt install nginx
```

## 2、购买域名

阿里云、腾讯云都可以

## 3、购买SSL证书

阿里云有免费的，每年20个
证书要绑定域名

## 4、域名解析

阿里云免费

## 5、设置Nginx反向代理

```bash
sudo vim /etc/nginx/conf.d/proxy.conf
```

输入下面内容：

```bash
server {
 listen 443 ssl;
 server_name 这里输入购买的域名;
 ssl_certificate 这里输入SSL证书的.epm文件;
 ssl_certificate_key 这里输入SSL证书的.key文件;
 ssl_session_cache shared:le_nginx_SSL:1m;
 ssl_session_timeout 1440m;
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
 ssl_prefer_server_ciphers on;
 ssl_ciphers TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:!MD5;
 location / {
 proxy_pass  https://api.openai.com/;
 proxy_ssl_server_name on;
 proxy_set_header Host api.openai.com;
 proxy_set_header Connection '';
 proxy_http_version 1.1;
 chunked_transfer_encoding off;
 proxy_buffering off;
 proxy_cache off;
 proxy_set_header X-Forwarded-For $remote_addr;
 proxy_set_header X-Forwarded-Proto $scheme;
 }
}
```

## 6、测试

跟openai官方的使用方法完全相同，只是将原本直接指向 api.openai.com 的请求通过服务器转发了一次，服务器修改了请求头，避免openai检测到来自被封禁地区的ip。

例如使用chatgpt的api，请求方式就要修改为 https://「你购买的域名」/v1/chat/completions，而原本的请求是 https://api.openai.com/v1/chat/completions。

这样所有针对OpenAI的请求都可以通过自己的服务器进行转发，任意使用，不用担心被封。