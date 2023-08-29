---
title: Rabbit MQ
date: 2022/7/25 23:43:28
updated: 2023/3/23 23:00:21
categories:
- Middleware
tags:
- MQ
---

# MQ
## MQ相关概念
### 什么是MQ
MQ(message queue),消息队列，遵循队列先入先出的原则。是一种跨进程的通信机制，用于上下游消息传递。
### MQ的作用
#### 流量消峰
将流量储存于一个队列中，按队列顺序一次处理，避免瞬间流量过大导致超出系统负载。
#### 应用解耦
如果系统发生故障，队列任务不丢弃，等待系统恢复正常后再依次处理，多个任务队列互不干扰，实现解耦。
#### 异步处理
两个或多个服务监听同一个消息队列，一个服务的任务完成时，向队列添加消息，其它服务通过监听这个消息总线（队列），可以得知任务完成，实现异步。
### 关键词
- **生产者**
发送（产生）消息的程序。
- **交换机**
接收生产者的消息，并将消息推送到特定的队列中。
- **队列**
消息缓冲区
- **消费者**
等待接收消息的程序

# RabbitMQ

## 工作原理

![image-20220725234534749](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220725234534749.png)

## 安装

1. 连接服务器

```sh
ssh root@0.0.0.0
```

2. 官网下载

[RabbitMQ](https://www.rabbitmq.com/download.html)

3. 上传文件到服务器

```sh
scp rabbitmq-server-3.10.6-1.el8.noarch.rpm root@0.0.0.0:/usr/local/software
```

4. 安装erlang环境

- 安装环境

```sh
sudo yum install -y gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git
```
- 创建目录
```sh
cd /home/local
mkdir dowload
cd dowload
```
- 下载文件
```sh
wget http://erlang.org/download/otp_src_25.1.tar.gz
```
- 解压
```sh
tar -zvxf otp_src_20.2.tar.gz
```
- 安装
```
cd otp_src_20.2.tar.gz
./otp_build autoconf
./configure && make && sudo make install
```
5. 安装rabbitMq

```sh
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
```

6. 启动

```sh
/sbin/service rabbitmq-server start 
```

7. 查看状态

```sh
/sbin/service rabbitmq-server status
```

8. 开启web服务器（需停止rebbitmq之后安装，再启动）

```sh
rabbitmq-plugins enable rabbitmq_management
```

9. 服务器防火墙开启一个端口

先查看是否已经开启

```sh
firewall-cmd --query-port=15672/tcp
```

开启15672端口

```sh
firewall-cmd --add-port=15672/tcp
```

这样就可以在浏览器里访问了

![image-20220726003609377](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220726003609377.png)

10. 创建账户

```sh
rabbitmqctl add_user violet Dd112211
```

设置用户角色

```sh
rabbitmqctl set_user_tags violet administrator
```

可以登陆了

![image-20220726003828392](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220726003828392.png)

![image-20220727212834057](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220727212834057.png)

此时可以看到没有权限

11. 设置用户权限

```sh
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
rabbitmqctl set_permissions -p "/" violet ".*" ".*" ".*"
```

用户 violet 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限 当前用户和角色

```sh
rabbitmqctl list_users
```

重启

```sh
rabbitmqctl stop_app
rabbitmqctl start_app
```

## 发送消息

1. 创建项目导入依赖

```xml
<dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.14.2</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.11.0</version>
        </dependency>
    </dependencies>
```

2. Java

```java
package top.beyondhorion.t1;


import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * ClassName: Produce
 * date: 2022/7/27 21:39
 *
 * @author ayanamirei
 */


public class Producer
{
    /*
    队列名称
     */
    public static final String QUEUE_NAME = "queue_1";
    /*
    队列地址
     */
    public static final String HOST = "127.0.0.1";
    
    public static final String USERNAME = "violet";
    
    public static final String PASSWORD = "Dd112211";
    public static void main(String[] args) throws IOException, TimeoutException
    {
        //1. 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2. 设置工厂参数
        factory.setHost(HOST);
        factory.setPort(5672);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        //3. 创建连接
        Connection connection = factory.newConnection();
        //4. 获取信道
        Channel channel = connection.createChannel();
        //5. 生成队列
        /*
        参数：
        String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        queue: 队列名
        durable: 是否持久化，如果为true，该队列在服务器重启之后仍然存在
        exclusive: 该队列是否共享，如果为true，该队列仅限此连接使用
        autoDelete: 是否自动删除，如果为true，队列中没有消息时自动删除队列
        arguments: 其他参数（构造函数）
         */
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        //6. 设置要发送的消息
        String msg = "hello word!";
        //7. 发送消息
        /*
        参数：
        exchange: 需要发送的交换机
        routingKey: 路由Key（队列名）
        props: 属性（other properties for the message - routing headers etc）
        body: 消息体
         */
        channel.basicPublish("",QUEUE_NAME,null, msg.getBytes());
        //8. 发送前记得开启服务器5672端口（管理面板使用15672端口，发送消息使用5672端口）
        System.out.println("send finished");
    }
}

```

3. 发送成功

![image-20220727221824959](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220727221824959.png)

![image-20220727221935242](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220727221935242.png)

## 接收消息

```java
package top.beyondhorion.t1;


import com.rabbitmq.client.*;

import java.util.Arrays;

/**
 * ClassName: Consumer
 * date: 2022/7/27 22:34
 *
 * @author ayanamirei
 */


public class Consumer
{
    /*
    队列名称
     */
    public static final String QUEUE_NAME = "queue_1";
    /*
    队列地址
     */
    public static final String HOST = "127.0.0.1";
    
    public static final String USERNAME = "violet";
    
    public static final String PASSWORD = "Dd112211";
    
    public static void main(String[] args) throws Exception
    {
        //1. 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2. 设置工厂参数
        factory.setHost(HOST);
        factory.setPort(5672);
        factory.setUsername(USERNAME);
        factory.setPassword(PASSWORD);
        //3. 创建连接
        Connection connection = factory.newConnection();
        //4. 获取信道
        Channel channel = connection.createChannel();
        //5. 接收消息
        //接收的回调
        DeliverCallback deliverCallback = ( consumerTag, message) -> {
            //接收成功就输出接收到的消息
            System.out.println(new String(message.getBody()));
        };
        //取消的回调
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("ERROR");
        };
        /*
        参数：
        queue: 队列名
        autoAck: 如果为真，接收到消息会确认
        deliverCallback: 接收成功回调
        cancelCallback: 消费者取消时的回调
         */
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }
}

```

## 工作队列（Work Queue）

当有大量消息和多个等待接收的消费者时，多个消费之会轮询接收到消息

## 消息应答

当一个消费者在处理消息时出错，导致消息没有被处理完，此时消息会丢失。

为了保证发送消息过程中消息不丢失，RabbitMQ使用了消息应答机制。消费者在接收到消息并且处理该消息之后，告诉 rabbitmq处理完成，rabbitmq才会将消息删除。

### 自动应答

消息发送后立即被认为已经传送成功，同时RabbitMQ会将消息删除，容易出现消息丢失的情况。

### 手动应答

- A.Channel.basicAck(用于肯定确认)  RabbitMQ 消息已经被处理完毕，可以删除

- B.Channel.basicNack(用于否定确认)  消息未成功处理完毕，重新发送

- C.Channel.basicReject(用于否定确认)  不对该消息处理，并且指令RabbitMQ删除该消息 

### 消息自动重新入队 

由于网络问题导致没有收到应答回调消息（未发送ack确认），该消息会被重新入队，并发送。

## RabbitMQ 持久化 


