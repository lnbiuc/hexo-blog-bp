---
title: 使用Netlify持续部署Vue项目
date: 2023/3/21 23:19:00
updated: 2023/3/23 23:15:53
categories:
- DevOps
tags:
- CI/CD
---

## 项目准备

在项目根目录下创建public文件夹

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/69221679410485828.png)

public文件夹下创建`_redirects`文件，文件内容：

```
/*    /index.html   200
/:path*    /index.html   200
```

###  解释

由于Vue是SPA单页面应用，在Netlify部署时，无法解析vue-router参数，这会使在子路由页面刷新时返回404错误页。

为了解决这一问题，需要添加自定义重定向文件`_redirects`到public目录下。public目录下文件在vite构建之后会在dist根目录下，Netlify会读取该文件。

文件含义：匹配所有请求参数，均返回index.html页面和状态码200

配置后，在子路由刷新页面也可以正常返回，不会出现404错误

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/47601679410770068.png)

## 代码上传到github

根目录应该有`package.json`文件，如果你的`package.json`文件不在根目录，则需要单独配置打包路径

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/43171679410883529.png)

## Netlify

1. 进入Netlify

点击Add new site -> Import an existing project

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/51161679411020136.png)

2. 授权登录github账号，选择代码所在仓库

3. 项目构建配置

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/38611679411159252.png)

如果项目根目录没有`yarn.lock`或者`pnpm.lock`文件，默认会使用`npm install`安装依赖。

如果需要添加`npm install`的参数，如：`npm install --force`，需要添加环境变量。

添加key[`NPM_FLAGS`](https://docs.netlify.com/configure-builds/manage-dependencies/#npm)，value`--force`

4. 点击Deploy site即可完成配置

### 添加域名

此时网站应该已经可以访问，使用Netlify提供的域名 

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/90051679411502600.png)

点击Domain setting来设置自己的域名

添加域名后，需要到自己的DNS服务器添加对应的解析值

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/92161679411707298.png)

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/42661679411785496.png)

记录类型为CNAME，值为Netlify所提供的，能够正常访问的链接

## 持续集成

下一次github对应分支代码有改动时，Netlify会自动构建新的网站

![image](https://user-pic-1308549476.cos.ap-nanjing.myqcloud.com/pic/19791679411919095.png)