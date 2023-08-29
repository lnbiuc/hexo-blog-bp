---
title: Docker
date: 2022/12/24 01:13:16
updated: 2023/3/23 22:53:11
categories:
- DevOps
tags:
- Docker
---

# docker命令

## 安装docker

```bash
# 删除已有内容
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 安装工具、配置源                  
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装docker
sudo yum install -y docker-ce docker-ce-cli containerd.io

# 镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 下载image
```bash

# 下载应用
docker pull nginx

# 下载指定版本
docker pull redis:1.2

# 删除images
docker rmi redis
docker rmi imageID

```
## 启动image
```bash
# 查看docker进程
docker ps

# 查看docker images
docker images

# 启动容器
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

docker run [设置项] nginx

# 设置name
--name=docker-nginx

# 后台运行
-d

# 开机启动
--restart=always

# 删除运行的image
docker rm nginx
docker rm -f nginx
docker stop nginx

# 更新容器设置，使其开机启动
docker update docker-nginx --restart=always
```

## 端口映射
```bash
# 本机端口:容器端口
docker run --name=docker-nginx --restart=always -d -p 80:80 nginx
```

## 修改容器内配置文件
```bash
docker exec -it docker-nginx /bin/bash
```

## 提交image
```bash
docker commit -a lnbiuc -m "change index.html" docker-nginx docekr-nginx-lnbiuc:v0.1
```

## 保存image成压缩文件
```bash
docker save -o docekr-nginx-file-tar.tar docekr-nginx-lnbiuc

# 使用压缩包
docker load -i docekr-nginx-file-tar.tar
```

## 推送到docker hub
```bash
docker tag docekr-nginx-lnbiuc:last lnbiuc/lnbiuc-nginx

docker login

docker push lnbiuc/lnbiuc-nginx

docker pull lnbiuc/lnbiuc-nginx
```

## 数据挂载
```bash
# ro：容器内只读
docker run --name docker-nginx --restart always -d -p 80:80 -v /data/html:/usr/share/nginx/html:ro nginx

# 挂载多个文件
docker run -d -p 80:80 -v /data/html:/usr/share/nginx/html:ro -v/data/conf/nginx.conf:/etc/nginx/nginx.conf --name docker-nginx --restart always
```

## 查看容器运行日志
```bash
docker logs docker-nginx
```

## 复制容器内文件到容器外
```bash
docker cp docker-nginx:/etc/nginx/nginx.conf /data/conf/nginx.conf

# 容器外复制到容器内
docker cp /etc/nginx/nginx.conf docker-nginx:/data/conf/nginx.conf
```

# 部署springboot

## 启动redis
```bash
docker run -d -p 6379:6379 --name docker-redis --restart always -v /data/redis/redis.conf:/etc/redis/redis.conf -v /data/redis/data:/data redis redis-server /etc/redis/redis.conf
```

## 打包springboot应用

```dockerfile
FROM openjdk:8
LABEL maintainer=lnbiuc
COPY target/*.jar /app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

```bash
# 构建
docker build -t simple-count:0.1  .

# 启动
docker run -d -p 8080:8080 simple-count:0.1
```

## 上传

```bash
docker tag simple-count:0.3 lnbiuc/springboot-simple-count

docker push lnbiuc/springboot-simple-count
```



