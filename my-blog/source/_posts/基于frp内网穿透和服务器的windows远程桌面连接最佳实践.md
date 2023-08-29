---
title: 基于frp内网穿透和服务器的windows远程桌面连接最佳实践
date: 2023/5/22 20:12:48
updated: 2023/6/16 22:15:43
categories:
- Others
tags:
- Linux
---

> 本文介绍一种基于windows远程桌面，使用任意设备远程连接到windows电脑的方式
>
> 至少需要1台服务器（本文以centos7.9操作系统的云服务器为例）
>
> 需要1台windows电脑（本文以内核版本为19045.2965的windows10家庭版为例）
>
> 任意远程桌面软件（本文以mac操作系统下的Minecraft Remote Destop为例）

## 服务端配置

### 安装

centos7.9操作系统下载以下文件

[frp Github Release](https://github.com/fatedier/frp/releases)


![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/13141684757099377.png/default)

- 将文件上传到服务器用户目录，以root为例

使用`tar -zxvf frp_0.48.0_linux_amd64.tar.gz`解压

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/6881684757119856.png/default)

- 进入`frp_0.48.0_linux_amd64`目录修改配置文件

其中以ftpc开头的文件均为客户端需要，服务端可以直接删除
![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/58351684757133847.png/default)

### 修改配置

修改`frps.ini`

```ini
[common]
bind_port = 7000
dashboard_port = 7500
token = Dd112211
dashboard_user = violet
dashboard_pwd = Dd112211
```

**bind_port**：客户端与服务器通信的端口，默认为7000

**dashboard_port：frp**管理面板端口

**token**：客户端与服务器连接的验证token

**dashboard_user**：管理面板用户名

**dashboard_pwd**：管理面板密码

### 启动

执行以下命令，此种方法需要服务器连接窗口一直开启，不推荐使用

```shell
./frps -c ./frps.ini
```

### 自启动

使用system工具实现开机自启和后台运行

> 以下内容来自[frp官方文档](https://gofrp.org/docs/setup/systemd/)

- 安装system工具

```shell
yum install systemd
```

- 修改启动配置文件

```
vi /etc/systemd/system/frps.service
```

- 文件写入内容

```ser
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /root/frp_0.48.0_linux_amd64/frps -c /root/frp_0.48.0_linux_amd64/frps.ini

[Install]
WantedBy = multi-user.target
```

- 启动命令

```shell
# 启动frp
systemctl start frps
# 停止frp
systemctl stop frps
# 重启frp
systemctl restart frps
# 查看frp状态
systemctl status frps
```

- 设置开机自启

```shell
systemctl enable frps
```

- 服务端开启端口7000，7500，7001（后面会用到）

```shell
sudo firewall-cmd --permanent --zone=public --add-port=7000/tcp
sudo firewall-cmd --permanent --zone=public --add-port=7500/tcp
sudo firewall-cmd --permanent --zone=public --add-port=7001/tcp

# 重启防火墙
sudo firewall-cmd --reload
```

至此，服务端配置完毕

## 客户端

### 安装

[frp Github Release](https://github.com/fatedier/frp/releases)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/59441684757152013.png/default)

下载后解压到`D:\Program Files\frp_0.48.0_windows_amd64`

以frps开头的文件均为服务端所需要，可以直接删除

### 配置

修改frpc.ini

```ini
[common]
server_addr = <此处替换为你的服务器ip>
server_port = 7000
token = Dd112211

[rdp]
type = tcp
local_ip = 127.0.0.1           
local_port = 3389
remote_port = 7001  
```

server_addr：服务区地址

server_port：客户端同服务区交互端口

token：连接到服务端所需token

type：连接类型：udp或tcp

local_ip：本地访问ip

local_port：本地访问端口，windows远程桌面默认访问端口为3389

remote_port：远程访问端口

> 此处配置文件的意思时，使用server_addr:remote_port连接到local_ip:local_port，相当与访问<此处替换为你的服务器ip>:7001 == 访问127.0.0.1:3389

### 启动

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/33931684757171733.png/default)

进入powershell，使用以下命令启动

```shell
./frpc.exe -c ./frpc.ini
```

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/11441684757181534.png/default)

之后使用其他设备通过<你的服务器ip>:7001，就可以远程访问到windows设备了

如果此时无法访问，请按照以下步骤检查

### 自启动

程序目录新建文件`start.bat`

```bat
@echo off
:home

./frpc.exe -c ./frpc.ini

goto home
```

打开任务计划程序

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/36741686924391076.png/default)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/68801686924484760.png)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/59551686924545528.png)

程序或脚本填start.bat文件位置，例如:`D:\Program Files\frp_0.48.0_windows_amd64\start.bat`

起始于填`start.bat`所在文件夹路径，例如：`D:\Program Files\frp_0.48.0_windows_amd64`

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/18471686924639504.png)

## 错误检查

### 服务器端口是否开启

 ```shell
 sudo firewall-cmd --list-ports
 ```

### 客户端电脑是否允许3389端口

进入防火墙高级设置，新建入站规则

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/19891684757189055.png/default)
添加允许3389端口

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/40961684757195401.png/default)
### 客户端是否开启远程桌面

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/45481684757202809.png/default)
如果以上检查无误，仍然无法连接，请到frp github查看相关issue

> 注：windows家庭版无法使用被远程桌面连接，专业版可以被远程桌面连接如果你是windows家庭版，需要进行以下配置

## windows家庭版配置

windows家庭版无法通过windows10设置开启远程桌面，需要使用补丁（此方法在windows版本19045.2965可用，高于此版本可能导致失效），如果该方法失效，请到github issue寻找最新的解决方案。

> 推荐升级到windows专业版

[Github Issue](https://github.com/stascorp/rdpwrap/issues/999)

[该方法原作者](https://www.cnblogs.com/hwajie/p/12447224.html)（此方法已在2020年失效）

2023年以后的请看以下方法，穿越回去的可以忽略

### 下载软件

[RDPWrap-v1.6.2.zip下载地址(作者Stas'M Corp)](https://github.com/stascorp/rdpwrap/releases/download/v1.6.2/RDPWrap-v1.6.2.zip)

[autoupdate-v07.09.2019.zip下载地址(作者asmtron)](https://github.com/stascorp/rdpwrap/files/3588284/autoupdate-v07.09.2019.zip)

### 安装1

将RDPWrap解压到`C:\Program Files\RDP Wrapper`

使用管理员身份运行`install.bat`

等待安装结束，点击`RDPConf.exe`

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/46711684757213020.png/default)
左上角`Wrapper state`，`Service state`，`Listener state`应该为全绿状态，此时远程桌面可以正常访问，如果有红色，请看下面方法

### 安装2

解压autoupdate到`C:\Program Files\RDP Wrapper`目录下，使用管理员身份运行`update.bat`

以管理员身份打开powershell，运行以下命令

```shell
net stop termservice

net start termservice
```

再次打开`RDPConf.exe`查看左上角状态是否为绿，如果不是继续下一步

打开`rdpwrap.ini`，如果没有自行创建，删除原文件内容，粘贴以下内容（从下方链接获取）

https://raw.githubusercontent.com/sebaxakerhtc/rdpwrap.ini/master/rdpwrap.ini

之后再执行：以管理员身份打开powershell，运行以下命令

```shell
net stop termservice

net start termservice
```

再次打开`RDPConf.exe`查看左上角状态是否为绿，如果此时状态为绿的，再次启动客户端frp即可`./frpc.exe -c ./frpc.ini`，如果还是不行，到[Github](https://github.com/stascorp/rdpwrap/issues/999)寻找最新方法



至此，你应该可以使用任意软件通过服务器远程到windows电脑了。
