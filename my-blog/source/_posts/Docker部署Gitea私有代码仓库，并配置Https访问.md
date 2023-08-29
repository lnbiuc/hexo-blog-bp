---
title: Docker部署Gitea私有代码仓库，并配置Https访问
date: 2023/2/2 17:25:14
updated: 2023/3/23 22:52:38
categories:
- Project
tags:
- Docker
- Git
---

# Gitea

官网：[Gitea](https://docs.gitea.io/zh-cn/)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/67251675329870537.png)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/80711675329877440.png)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/60481675329882677.png)

# 准备工作

## 域名

### 域名解析

首先需要一个能正常解析的域名，例如我所使用二级域名`git.vio.vin`

添加A记录解析，记录值为你服务器的IP

如果需要使用CDN，记录值为CNAME

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/41591675329894168.png)

### Https

申请ssl证书，如果你使用的是免费的Https证书，免费的一般为单域名证书，所以每个子域名都需要单独的ssl证书，需要重新申请（例如vio.vin的证书不可用于git.vio.vin）。

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/52321675329900045.png)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/71361675329904898.png)

## Docker

[Docker安装教程](https://vio.vin/article/nbvZaq2C)

## Nginx

Gitea官网并没有提供带Nginx的docker-compose安装方式。所以选择手动以docker方式安装nginx

[Docker安装Nginx教程]()(https://vio.vin/article/NTqcHv17)

## Docker环境

> 官网给的docker-compose.yml我安装后gitea无法访问到MySQL，我的解决方案是自己手动安装，docker-net，MySQL，Gitea

创建docker-net网卡，具体参数解释[Docker部署引用程序](https://vio.vin/article/NTqcHv17)

```bash
docker network create --subnet 192.168.0.0/16 --gatway 192.168.0.1 docker-net
```

安装MySQL

```bash
docker pull mysql:8.0.25
```

```bash
docker run -d -p 3306:3306 --name d-mysql --net docker-net --restart always -e MYSQL_ROOT_PASSWORD=mysqlpwd -e MYSQL_USER=violet -e MYSQL_PASSWORD=Dd112211 mysql:8.0.25
```

# 安装Gitea

随便找一个目录，新建文件

```bash
touch docker-compose.yml
```

文件内容

```yaml
version: "3"
networks:
  gitea:
    external: false
services:
  server:
    image: gitea/gitea:latest
    container_name: d-gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=d-mysql:3306
      - DB_NAME=gitea
      - DB_USER=root
      - DB_PASSWD=123456
    restart: always
    volumes:
      - /data/gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
```

安装完成后访问3000端口就能看到配置向导了，记得打开服务器和服务器安全组3000端口

## Gitea配置

第一次进入会有网页配置向导，之后想要重新配置需要到安装目录下的/conf/app.ini配置

```ini
APP_NAME = Violet's Git Repository
RUN_MODE = prod
RUN_USER = git

[repository]
ROOT = /data/git/repositories

[repository.local]
LOCAL_COPY_PATH = /data/gitea/tmp/local-repo

[repository.upload]
TEMP_PATH = /data/gitea/uploads

[server]
APP_DATA_PATH    = /data/gitea
DOMAIN           = git.vio.vin
SSH_DOMAIN       = ssl.git.vio.vin
HTTP_PORT        = 3000
ROOT_URL         = https://git.vio.vin/
DISABLE_SSH      = false
SSH_PORT         = 222
SSH_LISTEN_PORT  = 22
LFS_START_SERVER = true
LFS_JWT_SECRET   = yG6vlxASyRseSrdsdfsdfaddWEeJECaeg-CAmwiOGc
OFFLINE_MODE     = false

[database]
PATH     = /data/gitea/gitea.db
DB_TYPE  = mysql
HOST     = d-mysql:3306
NAME     = gitea
USER     = root
PASSWD   = 123456
LOG_SQL  = false
SCHEMA   = 
SSL_MODE = disable
CHARSET  = utf8mb4

[indexer]
ISSUE_INDEXER_PATH = /data/gitea/indexers/issues.bleve

[session]
PROVIDER_CONFIG = /data/gitea/sessions
PROVIDER        = file

[picture]
AVATAR_UPLOAD_PATH            = /data/gitea/avatars
REPOSITORY_AVATAR_UPLOAD_PATH = /data/gitea/repo-avatars
ENABLE_FEDERATED_AVATAR       = false

[attachment]
PATH = /data/gitea/attachments

[log]
MODE      = console
LEVEL     = info
ROUTER    = console
ROOT_PATH = /data/gitea/log

[security]
INSTALL_LOCK                  = true
SECRET_KEY                    = 
REVERSE_PROXY_LIMIT           = 1
REVERSE_PROXY_TRUSTED_PROXIES = *
INTERNAL_TOKEN                = eyJhbGcifdggdfAFNiIsInR5cCI6IkpXVCJ9.eyJuSDAfaszM1MDcxMDd9.g9T6Ix573FNwLg9Rzi20DXueBCqeBRnAHAAAAFSFM
PASSWORD_HASH_ALGO            = pbk452

[service]
DISABLE_REGISTRATION              = false
REQUIRE_SIGNIN_VIEW               = false
REGISTER_EMAIL_CONFIRM            = false
ENABLE_NOTIFY_MAIL                = false
ALLOW_ONLY_EXTERNAL_REGISTRATION  = false
ENABLE_CAPTCHA                    = false
DEFAULT_KEEP_EMAIL_PRIVATE        = false
DEFAULT_ALLOW_CREATE_ORGANIZATION = true
DEFAULT_ENABLE_TIMETRACKING       = true
NO_REPLY_ADDRESS                  = noreply.localhost

[lfs]
PATH = /data/git/lfs

[mailer]
ENABLED = false

[openid]
ENABLE_OPENID_SIGNIN = true
ENABLE_OPENID_SIGNUP = true

[repository.pull-request]
DEFAULT_MERGE_STYLE = merge

[repository.signing]
DEFAULT_TRUST_MODEL = committer
```



# Https设置

查看Gitea容器ip

```bash
docker network inspect docker-net
```

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/53321675329840167.png)

修改nginx.conf

将80端口转发到443再转发到容器IPvAddress:3000

如果需要使用ssh，需要先到gitea中配置ssh key，之后访问服务器222端口就是ssh访问

```nginx
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;
# include enable-cors.conf

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
    	server_name vio.vin git.vio.vin;

    	rewrite ^(.*)$ https://${server_name}$1 permanent;
	}
	
	server {
		listen *:443 ssl http2;
    	listen [::]:443  ssl http2;
        server_name git.vio.vin;
		ssl on;
        ssl_certificate git.vio.vin_bundle.crt;
        ssl_certificate_key git.vio.vin.key;
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
        keepalive_timeout   70;
        include /etc/nginx/default.d/*.conf;

        location / {
			proxy_pass http://192.168.0.5:3000;
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto $scheme;
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

