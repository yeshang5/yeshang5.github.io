---
layout: post
title: 'Nginx开启Gzip压缩大幅提高页面加载速度'
date: 2018-12-12
author: 白皓
cover: ''
tags: nginx 页面加载
---


当项目中存在大量静态资源时，访问时会因此变得十分缓慢，在nginx中配置开启gzip对静态文件进行压缩可实现页面加载速度大幅度提高

1. Vim打开Nginx配置文件
> vim /usr/local/nginx/conf/nginx.conf

2. 在http段中加入以下代码

```python
#资源压缩
   gzip on; #开启gzip 
   gzip_min_length 1k; #低于1kb的资源不压缩
   gzip_comp_level 6; #压缩级别【1-9】，越大压缩率越高，同时消耗cpu资源也越多，建议设置在4左右。 
   gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css image/jpeg image/gif image/png; #需要压缩哪些响应类型的资源，多个空格隔开 
   gzip_disable "MSIE [1-6]\."; #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持） 
   gzip_vary on; #是否添加“Vary: Accept-Encoding”响应头
```

3. :wq保存退出，使用nginx -s reload 命令重新加载Nginx

> nginx -s reload

* 注意
  若提示nginx命令无法执行，换成以下命令
> ./nginx -s reload

5. 查看Gzip是否成功开启

在浏览器控制台中选中静态文件查看header信息

```python
HTTP/1.1 200 OK
Server: nginx/1.0.15
Date: Sun, 26 Aug 2012 18:13:09 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/5.2.17p1
X-Pingback: http://www.slyar.com/blog/xmlrpc.php
Content-Encoding: gzip
```
> 出现Content-Encoding: gzip 信息则gzip开启成功

6. conf文件如下


```python
user root;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;


    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #资源压缩
     gzip on; #开启gzip 
     gzip_min_length 1k; #低于1kb的资源不压缩
     gzip_comp_level 6; #压缩级别【1-9】，越大压缩率越高，同时消耗cpu资源也越多，建议设置在4左右。 
     gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css image/jpeg image/gif image/png; #需要压缩哪些响应类型的资源，多个空格隔开 
     gzip_disable "MSIE [1-6]\."; #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持） 
     gzip_vary on; #是否添加“Vary: Accept-Encoding”响应头


    server {
        listen       80;
        server_name  www.cdxc512.cn;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
           proxy_set_header Host $http_host;  #解决浏览器访问网站时跳转内网127.0.0.1的问题
           proxy_pass http://127.0.0.1:8080;
          #  root   html;
          #  index  index.html index.htm;
        }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

      
    }


}

著作权归作者所有。
商业转载请联系作者获得授权,非商业转载请注明出处。
原文: https://yeshang5.github.io/2018/12/10/nginx%E5%9F%BA%E7%A1%80%E9%85%8D%E7%BD%AE%E8%AF%A6%E8%A7%A3.html
```
