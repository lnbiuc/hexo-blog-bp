---
title: Docker部署应用程序
date: 2023/1/12 23:02:06
updated: 2023/3/23 22:52:57
categories:
- DevOps
tags:
- Linux
- Docker
---


> 使用docker在centos8下部署一个springboot+vue应用程序，使用的容器有springboot、mysql、redis、nginx。包含springboot镜像构建Dockerfile、bridge容器互联、目录映射等内容。

# 基础环境（必须）

- 开启centos防火墙端口转发

```bash
firewall-cmd --add-masquerade --permanent
```

- 重启防火墙

```bash
firewall-cmd --reload
```

- 创建目录

  根目录下创建/data文件夹，用于统一保存挂载到容器内的文件

  - /data/redis/data：[目录]保存redis持久化数据
- /data/redis/redis.conf：[文件]redis配置文件
  - /data/nginx/html：[目录]nginx html文件目录
- /data/nginx/log：[目录]日志
  - /data/nginx/nginx/nginx.conf：[文件]nginx配置文件


# 网络

- 创建docker子网

```
docker network create --subnet 192.168.0.0/16 --gatway 192.168.0.1 docker-net
```

> 192.168.0.0/16：子网范围
>
> 192.168.0.1：网关地址
>
> docker-net：子网名称
>

- 创建后查看

```bash
docker network inspect docker-net
```

```json
[
    {
        "Name": "docker-net",
        "Id": "1fc4884e63c593800fbbf263d10c09a8dadb52307096d57a5b2e80f84574ab90",
        "Created": "2023-01-11T19:18:32.691672677+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

- 加入网络

  - 在docker启动参数中加入`--net docker-net`

  - 容器启动之后，使用命令`docker network connect docker-net <容器ID/NAME>`


# redis

- 下载redis docker镜像

```bash
docker pull redis
```

- 创建redis配置文件

```bash
vi redis.conf
```

- redis.conf文件中写入

```conf
requirepass <你的redis密码>
```

- 启动redis

```bash
docker run -d -p 6379:6379 --name docker-redis --restart always -v /data/redis:/etc/redis -v /data/redis/data:/data --net docker-net redis redis-server /etc/redis/redis.conf
```

> -d：后台运行
>
> -p：端口映射，宿主机6379端口映射到容器6379端口
>
> --name：容器启动后的名字
>
> --restart always：容器自动重启
>
> -v：文件挂载将容器内目录挂载到宿主机，左边是宿主机地址，右边是容器内地址
>
> --net docker-net：加入docker子网

- 启动后使用`docker ps -a`查看容器是否正常启动

# MySQL

- 获取镜像

```bash
docker pull mysql:8.0.25
```

> ":"用于指定MySQL版本

- 启动mysql容器

```bash
docker run -d -p 3306:3306 --name d-mysql --net docker-net --restart always -e MYSQL_ROOT_PASSWORD=mysqlpwd -e MYSQL_USER=violet -e MYSQL_PASSWORD=Dd112211 mysql:8.0.25
```

> -e MYSQL_ROOT_PASSWORD=mysqlpwd：-e表示自定义参数，root用户密码
>
> -e MYSQL_USER=violet：容器启动时创建一个用户
>
> -e MYSQL_PASSWORD=Dd112211：该用户的密码
>
> 其他参数可以在[docker hub mysql](https://hub.docker.com/_/mysql)查看
>

- 设置用户权限

可以为容器启动时创建的用户赋予权限，或者让root用户可以远程登录

进入容器内

```bash
docker exec -it d-mysql /bin/bash
```

进入mysql-client

```bash
mysql -uroot -p
```

设置root可以远程登录

```bash
grant all privileges on *.* to 'root'@'%';
```

刷新权限

```bash
FLUSH PRIVILEGES;
```

- 退出mysql-client，退出docker容器

```bash
exit
```

- mysql数据库数据导入docker mysql

在mysql-client中执行

```bash
mysqldump -uroot -p --databases arrogant javawebsql my_blog springcloud vote_sys >/data/mysql/data-backup.sql
```

使用mysqldump来下载数据，保存在可执行的sql文件中

> /data/mysql/data-backup.sql：目录及最后生成sql文件名字
>
> --databases 和 目录之间的是需要转储的数据库名
>
> 将sql文件复制到mysql容器内

```bash
docker cp /data/mysql/data-backup.sql d-mysql:/home/
```

进入mysql容器，进入mysql-client，执行命令，将数据写入，必须在sql文件目录下进入

```bash
source data-backup.sql
```

# 构建springboot镜像

- application.yml设置

数据库地址设置为d-mysql，即mysql docker容器name

```yaml
url: jdbc:mysql://d-mysql:3306/my_blog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
```

redis地址设置为d-redis

```yaml
host: d-redis
```

- 在src目录下创建文件Dockerfile

```dockerfile
FROM openjdk:8
LABEL maintainer=lnbiuc
COPY ./*.jar /app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

- 上传文件

将Dockerfile和jar包放到同一个目录下

- 构建镜像

```bash
docker build -t api:8.8 .
```

> -t：表示tag这里，这里指镜像版本，api是镜像名字
>
> .：点非常重要，表示在当前目录下构建

- 构建完成，查看镜像

```bash
docker images
```

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/18081673535701138.png)

- 启动容器

```bash
docker run -d -p 8888:8888 -v /data/api:/log --name api --net docker-net --restart always  api:8.8.3
```

这里将log文件夹挂载出来方便查看，jar包根目录，所以log文件夹也在同级的根目录下

# Nginx

- 拉取镜像

```bash
docker pull nginx
```

- 获取springboot访问地址

```bash
docker network inspect docker-net
```

输出

```json
[
    {
        "Name": "docker-net",
        "Id": "1fc4884e63c593800fbbf263d10c09a8dadb52307096d57a5b2e80f84574ab90",
        "Created": "2023-01-11T19:18:32.691672677+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "00729e5b41d6ddcec195722869e57d1baf7bd754793cac7f986d452032b6810d": {
                "Name": "d-redis",
                "EndpointID": "2ea0446903341146ccc741aac0431ae3f3e0e6acc8972763f024136a3499a0ae",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "0682772aaec9f7d6bb435cf0d38e478c5298fa577783a226d086867d71bcf4b3": {
                "Name": "d-nginx",
                "EndpointID": "97dde4d3e0b8fff81722518b3b62094af5f0c1b1b484ef112c7db3490fb89ee7",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "27325e2b869eb28534e5a3e65de5eb8a650fa8488ded994de46cbb271af40a17": {
                "Name": "d-gitea",
                "EndpointID": "1a9ab8e968041b4df12ebaacecb74a166f23438cbba9b5345878c6fa672d982d",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "bf2d1cf717fbc3a39c06068bba9638ad6ce49d407b3a6e28c196803188597038": {
                "Name": "api",
                "EndpointID": "34a832756b3c9c051d67e096f27448a80165407579de0a25877fe4d4e9f6e5d3",
                "MacAddress": "02:42:c0:a8:00:05",
                "IPv4Address": "192.168.0.5/16",
                "IPv6Address": ""
            },
            "ebb72c81a769e9e8385cdbf910a5845ae3f53d8cd8e4d3d317046dd726b74947": {
                "Name": "d-mysql",
                "EndpointID": "c86a7becc085a4f47a35a8488b81bef771e0ae9f6eadd7e41d0db2beae3a8862",
                "MacAddress": "02:42:c0:a8:00:06",
                "IPv4Address": "192.168.0.6/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

其中name为api下的IPv4Address即使nginx需要反向代理的地址

- 准备好ssl证书和nginx.conf文件，放到/data/nginx/nginx目录下

我的nginx.conf

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
    	server_name beyondhorizon.top;

    	rewrite ^(.*)$ https://${server_name}$1 permanent;
	}

    server {
    	listen *:80;
		listen [::]:80;
    	server_name www.beyondhorizon.top;

    	rewrite ^(.*)$ https://${server_name}$1 permanent;
	}

    server {
		listen *:443 ssl http2;
    	listen [::]:443  ssl http2;
        server_name beyondhorizon.top www.beyondhorizon.top;
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
        keepalive_timeout   70;
        include /etc/nginx/default.d/*.conf;

        location / {
                root /usr/share/nginx/html;
                index index.html;
                try_files $uri $uri/ /index.html;
		}

        location ~ /api/ {
                add_header Cache-Control max-age=3600;
                proxy_pass http://192.168.0.5:8888;
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

- 上传html文件到/data/nginx/html下
- 启动nginx

```bash
docker run -d -p 80:80 -p 443:443 -v /data/nginx/html:/usr/share/nginx/html -v /data/nginx/nginx:/etc/nginx -v /data/nginx/log:/var/log/nginx --name d-nginx --restart always --net docker-net nginx
```

- 后续如果修改了nginx配置文件，使用命令重启容器即可

```bash
docker restart d-nginx
```



此时全部容器都应该已经启动了，并且网站也可以正常访问啦

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/61871673535676537.png)
