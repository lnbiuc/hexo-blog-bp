---
title: SpringMVC
date: 2022/10/5 21:29:56
updated: 2023/3/23 22:56:52
categories:
- BackEnd
tags:
- Spring
---

# 概念

MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码。

# 核心组件

- DispatcherServlet（中央处理器）：：负责接收请求、处理、响应。
- HandlerMapping（处理器映射器）：：根据uri查找处理请求的Handler。
- HandlerAdapter（处理器适配器）：：根据HandlerMapping指定的Handler，执行Handler。
- Handler（请求处理器）
- ViewResolver（视图解析器）：根据Handler返回的视图，解析Jsp并渲染成HTML，有DispatcherServlet返回客户端。

# 工作原理

![image-20221005221852139](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221005221852139.png)

# 执行流程

1. 客户端发送请求，DispatcherServlet拦截前端请求。
2. DispatcherServlet将请求地址交给HandlerMapping，HandlerMapping找到对应的Handler。
3. DispatcherServlet调用HanlderAdapter执行Handler，将结果ModelAndView返回DispatcherServlet。
4. DispatcherServlet将ModelAndView交给VierResolver。
5. VierResolver根据View返回对应View。
6. DispatcherServlet根据Modle和View渲染页面，并返回给客户端。