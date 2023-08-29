---
title: Java🍵I/O
date: 2022/9/17 23:52:59
updated: 2023/3/23 23:02:40
categories:
- Java
tags:
- Java
---

> [原文地址](https://javaguide.cn/java/io/io-basis.html)

# 概念

## 什么是IO流

IO即输入输出。数据输入到计算机内存的过程是输入，数据输出到外部存储的过程数输出。

IO流在Java中包含输入流和输出流，根据数据的处理方式不同，又可分为字节流和字符流。

- `InputStream`/`Reader`: 所有的输入流的父类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`: 所有输出流的父类，前者是字节输出流，后者是字符输出流。

## 字节流

### InputStream

`InputStream`用于从文件中读取（字节）信息到内存中。

- `read()`：返回输入流中下一个字节数据。返回值（0 ～ 255）。如果未读取到返回 -1 ，表示文件读取完成。
- `read(byet b[])`从输入流中读取一些字节存放到数组 `b` 中，可读取长度为`b.length`。如果未读取到返回 -1 ，表示文件读取完成。
- `read(byte b[], int off, int len)`：`off`偏移量，`len`要读取最大字节数。
- `skip(long n)`：忽略输入流中第n个字符。返回忽略的字节数。
- `available()`：返回输入流中可以读取到的字节数。
- `close()`：关闭输入流并释放资源。

`FileInputStream`用于从文件中读取数据，是`InputStream`的实现类

```java
public class Main
{
    public static void main(String[] args)
    {
        try
        {
            InputStream inputStream = new FileInputStream("path");
            int available = inputStream.available();
            System.out.println("available: " + available);
            byte[] b = new byte[available];
            inputStream.read(b);
            for (byte b1 : b)
            {
                System.out.print((char) b1);
            }
        } catch (IOException e)
        {
            throw new RuntimeException(e);
        }
    }
}
```

### DataInputStream

用于限制读取数据类型

```java
DataInputStream dataInputStream = new DataInputStream(new FileInputStream("path"));
dataInputStream.readBoolean();
dataInputStream.readInt();
dataInputStream.readUTF();
```

### ObjectInputStream

用于从输入流中读取对象并反序列化，（对象需要实现`Serializable`）

```java
ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.data"));
Object object = nput.readObject();
```

---

### OutputStream

用于将数据写入文件

- `write(int b)` ：写入字节b。
- `write(byte b[ ])` : 将数组`b` 写入。
- `write(byte[] b, int off, int len)` : `off`偏移量，`len`要读取最大字节数。
- `flush()` ：刷新此输出流并强制写出所有缓冲的输出字节。
- `close()` ：关闭输出流释放系统资源。

```java
String content = "hello";
        try
        {
            // 会覆盖原有内容
            FileOutputStream stream = new FileOutputStream(path);
            stream.write(content.getBytes());
            stream.close();
        } catch (IOException e)
        {
            throw new RuntimeException(e);
        }
```

### BufferedOutputStream

字节缓冲输出流

### DataOutputStream

用于写入指定类型数据

### ObjectOutputStream

将对象写入到输出流(`ObjectOutputStream`，序列化)

### 字节缓冲流

频繁IO操作消耗性能，缓冲流将需要操作数据先加载到缓冲区，一次性写入或读取多个字符，提高效率，节省资源。

```java
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("path"));
```

### 字节缓冲输入流(BufferedInputStream)

现将数据读取到缓冲区，在从缓冲区读取，减少IO次数。

缓冲区的大小默认为 **8192** 字节，也可自定义

###  字节缓冲输出流(BufferedOutputStream)

同样通过缓冲区

缓冲区的大小默认为 **8192** 字节，也可自定义

## 字符流

### 字节流和字符流

- 字节流以字节为单位读写数据。传输效率高，但不规定编码格式容易乱码。
- 字符流以字符为单位读写数据。需要JVM将字节转换成字符，损失一部分性能，但是能自由选择编码格式。

字符流默认的编码格式是`unicode`，常见的编码格式还有

- `utf8`：英文占 **1** 字节，中文占 **3** 字节
- `gbk`：英文占 **1** 字节，中文占 **2** 字节
- `unicdoe`：中英文都占 **2** 字节

### Reader

- `read()` : 从输入流读取一个字符。
- `read(char[] cbuf)` : 从输入流中读取字符，并将其存储到字符数组 `cbuf`中。
- `read(char[] cbuf, int off, int len)` ：`off`偏移量，`len`要读取最大字节数。
- `skip(long n)` ：忽略输入流中的 n 个字符 ,返回实际忽略的字符数。
- `close()` 

### InputStreamReader

`java.io.Reader`的子类

### FileReader

`InputStreamReader`的子类，实际用到的类

```java
try
        {
            FileReader fileReader = new FileReader(path);
            int content;
            while ((content = fileReader.read()) != -1)
            {
                System.out.print((char)content);
            }
        } catch (IOException e)
        {
            throw new RuntimeException(e);
        }
```

### Writer

- `write(int c)` : 写入单个字符。
- `write(char[] cbuf)` ：写入字符数组 `cbuf`。
- `write(char[] cbuf, int off, int len)` ：`off`偏移量，`len`要读取最大字节数。
- `write(String str)` ：写入字符串 。
- `write(String str, int off, int len)` ：写入字符串，`off`偏移量，`len`要读取最大字节数。
- `append(CharSequence csq)` ：将指定的字符序列附加到指定的 `Writer` 对象并返回该 `Writer` 对象。
- `append(char c)` ：将指定的字符附加到指定的 `Writer` 对象并返回该 `Writer` 对象。
- `flush()` ：刷新此输出流并强制写出所有缓冲的输出字符。
- `close()`:关闭输出流释放相关的系统资源。

### FileWriter

```java
try
        {
            Writer writer = new FileWriter(path,true);
            writer.write("123");
        } catch (IOException e)
        {
            throw new RuntimeException(e);
        }
```

### 字符缓冲流

同字节缓冲流类似

## 字节流和字符流的区别

- 字节流读取单个字节，字符流读取单个字符。
- 字节流适合处理二进制文件（图片、视频、音频等），字符流适合处理文本信息。

# BIO（阻塞式IO）

> - 阻塞IO和非阻塞IO
>
> **程序级别**，程序请求操作系统后，如果IO资源没有准备好。阻塞IO会时程序进入等待状态、非阻塞IO不会中断执行。
>
> - 同步IO和非同步IO
>
> **操作系统级别**，操作系统收到IO请求之后，如果IO资源没有准备好，如何响应程序。同步IO不响应，直到IO资源准备好。非同步IO先返回一个标记，IO资源准备好之后，在用事件机制返回程序。

# NIO（非阻塞IO 或 New IO）

- 将数据打包成块进行传输
- 

## Channel

通道，应用程序和操作系统交互的通道，通过通道输入或输出数据，全双工通信，可用于读或写，或同时读写

- FileChannel 文件读写数据
- DatagramChannel 通过UDP读写网络中数据
- SocketChannel 通过TCP读取网络中数据
- ServerSocketChannel 监听TCP链接，每个TCP链接创建一个SocketChannel

## Buffer

缓冲区

## Selector

选择器，选择器的作用

- 事件订阅的channel管理

应用程序向selector对象注册应用程序需要的channel。以及具体的某个channe执行那些IO事件。

- 轮询代理

应用程序不直接询问操作系统IO状态，由selector代理询问。

- 由JVM具体实现

多路IO复用技术，同一个端口可以接受多个客户端IO事件，处理多种IO协议。

# AIO（异步IO）

使用订阅-通知模式，应用程序向操作系统订阅IO资源，当IO资源准备好之后由系统通知程序。

