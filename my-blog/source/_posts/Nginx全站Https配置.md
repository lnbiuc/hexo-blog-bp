---
title: Nginx全站Https配置
date: 2022/9/28 23:31:39
updated: 2023/3/23 22:56:43
categories:
- DevOps
tags:
- Nginx
---

> 均已开启SSL，HSTS，静态文件压缩

# 80和443分开代理

> 80和443分开代理，保证绝大多数情况下都可用，用户仍然可以通过80端口访问。
>
> CDN回源协议可以为80也可以为443

![image-20221006191954258](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221006191954258.png)

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
    gzip on;
    gzip_static on;
        gzip_min_length 1k;
    gzip_comp_level 7;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary on;
	

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name beyondhorizon.top www.beyondhorizon.top;

    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
	location = /violet {
                root /nginx/html;
                index admin.html;


                try_files $uri $uri/ /admin.html;
        }

	location / {
                root /nginx/html;
                index index.html;
                try_files $uri $uri/ /index.html;
                }

	location ~ /api/ {
                add_header Cache-Control max-age=3600;
                proxy_pass http://127.0.0.1:8888;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }

		server {
                listen *:443 ssl http2;
        listen [::]:443  ssl http2;
        server_name www.beyondhorizon.top;
                ssl on;
        ssl_certificate beyondhorizon.top_bundle.crt;
        ssl_certificate_key beyondhorizon.top.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:ECDHE-RSA-DES-CBC3-SHA:ECDHE-ECDSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';

        ssl_prefer_server_ciphers on;

            ssl_session_cache   shared:SSL:10m;
        ssl_session_tickets on;
        ssl_stapling        on;
        ssl_stapling_verify on;
                add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        client_max_body_size 10m;
        include /etc/nginx/default.d/*.conf;
                keepalive_timeout   70;
        location = /violet {
                root /nginx/html;
                index admin.html;


                try_files $uri $uri/ /admin.html;
        }

        location / {
                root /nginx/html;
                index index.html;
                try_files $uri $uri/ /index.html;
                }

        location ~ /api/ {
                add_header Cache-Control max-age=3600;
                proxy_pass http://127.0.0.1:8888;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

# 80 强制 跳转443

> 所有80端口的请求都会被重定向到443端口（用户无感知）。

CDN设置为强制HTTPS 并 使用443回源

![image-20221006191954258](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221006191954258.png)

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
    gzip on;
    gzip_static on;
	gzip_min_length 1k;
    gzip_comp_level 7;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary on;

	
	server {
    	listen *:80;
		listen [::]:80;
    	server_name beyondhorizon.top;

    	rewrite ^(.*)$ https://www.${server_name}$1 permanent;
	}

    server {
		listen *:443 ssl http2; 
    	listen [::]:443  ssl http2; 
        server_name www.beyondhorizon.top;
		ssl on;
        ssl_certificate beyondhorizon.top_bundle.crt;
        ssl_certificate_key beyondhorizon.top.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1.2 TLSv1.3;
		ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:ECDHE-RSA-DES-CBC3-SHA:ECDHE-ECDSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';

        ssl_prefer_server_ciphers on;

	    ssl_session_cache   shared:SSL:10m;
    	ssl_session_tickets on;
    	ssl_stapling        on;
    	ssl_stapling_verify on;
		add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        client_max_body_size 10m;
        include /etc/nginx/default.d/*.conf;
		keepalive_timeout   70;

        location / {
                root /nginx/html;
                index index.html;
                try_files $uri $uri/ /index.html;
		}

		location = /admin {
                root /nginx/html;
                index admin.html;
                try_files $uri $uri/ /admin.html;
        }

        location ~ /api/ {
                add_header Cache-Control max-age=3600;
                proxy_pass http://127.0.0.1:8888;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```


