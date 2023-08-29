---
title: Centos8环境搭建及项目部署
date: 2022/9/14 13:37:03
updated: 2023/3/23 22:54:25
categories:
- DevOps
tags:
- Linux
---

### 查看磁盘使用情况(查看每个根路径的分区大小)

```shell
df -h
```

### 查看磁盘剩余空间

```shell
df -hl
```

## 环境安装

### Mysql

#### 直接使用命令一键安装，默认最新稳定版，这里安装的是8.0.26

```shell
sudo dnf install @mysql
```

#### 设置开机启动

```shell
sudo systemctl enable --now mysqld
```

#### 查看Mysql运行状态

```shell
sudo systemctl status mysqld
```

#### 为Mysql设置密码

```shell
sudo mysql_secure_installation
```

进入密码配置：

- VALIDATE PASSWORD component（验证密码组件）： 输入y 
- 选择密码验证策略等级，输入0
- 输入新密码两次
- 确认是否继续使用提供的密码？输入y 
- 移除匿名用户？ 输入y 
- 不允许root远程登陆？ 输入n
- 移除test数据库？ 输入y 
- 重新载入权限表？ 输入y 

#### 设置root用户可以远程登录

```shell
mysql -uroot -p
```

- 再次输入密码
- 依次执行以下语句

```sql
use mysql;
update user set host='%' where user='root';
flush privileges;
```

- 输入`exit`退出mysql

#### 开启mysql 3306端口

检查防火墙是否开启

```shell
sudo systemctl status firewalld
```

启动防火墙

```shell
sudo service iptables start #启动防火墙
```

开启3306端口

```shell
sudo firewall-cmd --add-port=3306/tcp --permanent
```

重启防火墙

```shell
sudo firewall-cmd --reload
```

#### 重启Mysql服务

```shell
sudo systemctl restart mysqld
```

### Redis

#### 使用命令安装

添加yum源

```shell
sudo yum install epel-release
```

更新源

```shell
sudo yum update
```

安装redis数据库

```shell
sudo yum -y install redis
```

启动redis

```shell
sudo systemctl start redis
```

#### 设置redis密码

**需要使用vi编辑器**

```shell
sudo vi /etc/redis.conf
```

使用:/requirepass 找到这一行

`#requirepass foobared`

删除前面的 `#` 将foobared改成你需要的密码

```shell
requirepass 密码
```

使用`:wq!`退出并保存

重启redis

```shell
sudo systemctl restart redis
```

#### 设置开机启动redis

```shell
sudo systemctl enable redis.service
```

#### 其他命令

```shell
sudo systemctl start redis.service #启动redis服务器
sudo systemctl stop redis.service #停止redis服务器
sudo systemctl restart redis.service #重新启动redis服务器
sudo systemctl status redis.service #获取redis服务器的运行状态
sudo systemctl disable redis.service #开机禁用redis服务器
```

#### 进入redis

```
redis-cli
```

验证密码

```shell
auth <password>
```

### Nginx

#### 使用命令安装

```shell
sudo yum -y install nginx
```

#### 启动

```shell
sudo service nginx start
```

设置开机启动

```shell
sudo systemctl enable nginx
```

#### 修改配置文件

```shell
sudo vi /etc/nginx/nginx.conf
```

检查配置文件有没有语法错误

```shell
sudo nginx -t
```

重启

```shell
sudo service nginx restart
```

重载配置文件

```shell
sudo service nginx reload 
```

#### 其他命令

```shell
sudo systemctl enable nginx # 设置开机启动 
sudo service nginx start # 启动 nginx 服务
sudo service nginx stop # 停止 nginx 服务
sudo service nginx restart # 重启 nginx 服务
sudo service nginx reload # 重新加载配置
```

### JRE JDK

#### 安装JRE

查询jdk安装包

```shell
sudo yum search java|grep jdk
```

安装jdk1.8

```shell
sudo yum install java-1.8.0-openjdk
```

输入`java`查看是否安装成功

#### 安装JDK

```shell
sudo yum install java-1.8.0-openjdk-devel.x86_64
```

#### 配置环境变量

```shell
sudo vi /etc/profile
```

将下列代码粘贴到最后，并保存

```shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

使用`javac`测试是否配置成功

## 部署项目

### Springboot

#### 将文件上传到服务器

返回服务器根目录

```shell
cd /
```

创建目录存放jar文件

```shell
mkdir springboot
```

使用`scp`指令上传jar文件

```shell
scp <文件名>.jar root@<服务器IP>:/springboot
```

#### 运行Jar文件

查看8888端口是否被占用

```shell
netstat -nlp |grep :8888
```

后台运行jar文件

```shell
sudo nohup java -jar <文件名>.jar &
```

返回进程PID说明运行成功

#### 	其他

杀死进程

```shell
sudo kill -9 <进程PID>
```

### VUE

#### 将前端文件压缩，scp不能直接上传文件夹

```shell
tar zcvf <压缩后文件名>.tar.gz <需要被压缩的文件夹>
```

#### 上传文件

```shell
scp <文件名>.tar.gz root@<服务器IP>:/usr/local/nginx/html
```

#### 解压

```shell
tar zxvf <文件名>.tar.gz
```

## springcloud相关

### zookkeeper
#### 安装
下载文件
```shell
sudo wget https://archive.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
```

解压
```shell
tar -zxvf zookeeper-3.4.9.tar.gz 
```

移动目录
```shell
mv zookeeper-3.4.9 /usr/local
```

创建数据目录
```shell
cd /usr/local/zookeeper
mkdir data
```

复制配置文件
```shell
cd /usr/local/zookeeper/conf
cp zoo_sample.cfg  zoo.cfg
```

在配置文件中修改数据目录

将dataDir修改为刚才创建的目录
```shell
dataDir=/usr/local/zookeeper/data
```

修改数据目录使用权限
```shell
chmod 777 /usr/local/zookeeper/data
```

#### 启动

进入bin目录
```shell
cd /usr/local/zookeeper/bin
```

启动
```shell
./zkServer.sh start
```

停止、查看状态
```shell
./zkServer.sh stop
./zkServer.sh status
```

防火墙开启2181端口
```shell
firewall-cmd --zone=public --add-port=2181/tcp --permanent
systemctl restart firewalld.service
systemctl status firewall
```