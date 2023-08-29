---
title: Redis🧑🏼‍💻进阶-数据结构
date: 2022/10/9 18:55:25
updated: 2023/3/23 23:01:21
categories:
- Middleware
tags:
- Redis
---

# Redis中的数据结构

- 5种基础数据结构：string（字符串）、list（列表）、set（集合）、hash（散列）、zset（有序集合）
- 3种特殊数据结构：HyperLogLogs（基数统计）、Bitmap（位存储）、Geospatial（地理位置）

# 对应关系

> [图片来自](https://www.pdai.tech/md/db/nosql-redis/db-redis-data-type-enc.html)

![image-20221016185227739](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221016185227739.png)

# 动态字符串SDS

Redis并没有使用C中的字符串，而是自己定义了一种数据结构，即简单动态字符串（Simple Dynamic String SDS）。Redis中String的底层数据结构使用的就是SDS。

## SDS内存结构

- `len`保存SDS保存字符串的长度
- `buf[]`数组保存字符串的每个元素
- `alloc`分别以unint8、unint16、unint32、unint64表示整个SDS
- `flags`始终为一字符，以低三位标示着头部的类型

## SDS的优势

- **O(1)复杂度获取字符串长度**：在Redis中如果要获取字符串长度只需读取len属性即可，在C中如果要获取字符串长度需要遍历整个字符串，是O(n)复杂度。使用`strlen <K>`获取字符串长度。
- **防止缓冲区溢出**：在C中，使用`strcat`中拼接两字符串，如果没有分配足够的内存空间，会造成缓冲区溢出。SDS会先根据len先推算拼接后的长度是否满足内存空间要求。不满足会先扩展，再拼接，从而防止缓冲区溢出。
- **减少修改字符串内存重新分配次数**：SDS使用`len`和`alloc`实现**空间预分配**和**惰性空间释放**。
    - **空间预分配**：对字符串空间扩展时，扩展大于需求，从而减少字符串长度增加所需的内存重新分配次数。当字符串小于1M时，redis会分配字符串大小一倍的空间，当字符串大小大于1M是，额外多分配1M空间。
    - **惰性空间释放**：字符串长度减少时，不立即归还内存，而是使用`alloc`属性将这些空间记录下来，等待后续使用。如果觉得这样会浪费空间，可以自行设置定时释放未使用空间。
- **二进制安全**：C中使用空字符来表示一个字符串的结束。但是如果Redis中保存的二进制文件（如图片）中可能包含空字符，C无法正确读取。SDS使用`len`属性表示的长度来判断字符串是否结束。

# 压缩列表ZipList

ziplist是一种为提高存储效率设计的特殊编码的双向链表。可用于存储字符串或整数。

## ziplist内存结构

- `zlbytes`存储整个ziplist所占用内存字节数
- `zltail`存储ziplist中最后一个entry的偏移量，用于快速定位最后一个entry，方便快速pop操作
- `zllen`存储ziplist中entry的数量。
- `zlend`是终止字节

## Entry结构

<prevlen> <encoding> <entry-data>

- `prevlen`前一个entry的大小
- `encoding`当前entry的类型和长度
- `entry-data`entry数据

PS：如果entry-data中保存的是整形数据，encoding会和entrydata合为一个，此时entry结构为<prevlen> <encoding>

## ziplist优点

节省内存

- 普通List保存是，每个元素占用空间大小一致，均为当前List中最大元素所占的空间大小，造成了空间浪费。
- ziplist通过特殊的数据结构，尽量是每个元素占用与自身大小相同的空间，减少空间浪费。
- 传统List在内存中地址连续，ziplist所占大小不一致，所以地址不连续，需要通过`prelen`字段保存上一个元素的length来解决遍历问题。

## ziplist缺点

- 不预留内存空间，list中元素被移除之后list无用空间立即归还，这导致每一次修改list都需要重新分配空间。

# 快表QuickList

以ziplist作为结点的双端链表，如何理解QuickList：`List<List<ziplist>> quickList`

## quicklist在内存中结构

- `quicklistNode`链表中的节点，在quicklist中，这个节点是ziplist
- `quickListLZF`ziplist经过LZ4算法压缩后，可以包装成一个`quickListLZF`结构
- `quicklistBookmark`位于quicklist尾部
- `quicklist`
    - head、tail指向头尾指针
    - len表示链表中的节点
    - count表示quicklist中ziplist中entry数目
    - fill控制每个quicklist中ziplist最大所占空间
    - compress控制是否要对每个ziplist以LZ4算法进行压缩以节省空间
- `quicklistIter`迭代器
- `quicklistEntry`对ziplist中entry封装

# 哈希表-Dict-HashTable

哈希表于大多数语言中的HashTable无异，差别主要存在于解决哈希冲突、扩容。

## 哈希冲突

使用链地址法，[在Java数据结构中已经学习过了](https://www.beyondhorizon.top/article/xclfxsXC)

## 扩容

扩容步骤：

1. 根据原哈希表所占空间大小创建一个新哈希表，大小为原哈希表一倍，（收缩也是减小为原来一倍大小）
2. 使用哈希算法重新计算索引值
3. 新表创建并赋值完毕之后删除原表

扩容发生条件

1. redis没有执行`bgsave`或`bgrewriteaof`命令，负载因子大于等于1
2. redis没有执行`bgsave`或`bgrewriteaof`命令，负载因子大于等于5

负载因子 = 哈希表中节点数量 / 哈希表大小

# 整数集 IntSet

intset是redis中集合类型底层实现之一，当一个集合只包含整数数值元素，并且元素数量较少，redis使用intset作为集合键的底层实现

## 内存结构

- `encoding`编码方式
- `length`存储整数个数
- `contents`指向实际存储数值的连续内存区域

## 扩容

intset底层为数组，当新插入的元素大小大于原数组中最大元素的大小是，就需要对整个数组进行扩容，每个元素所占空间大小修改为最大元素的大小

会扩容但不会缩容！

# 跳表ZSkipList

跳跃表的目的类似于红黑树，为了性能。跳跃表可以在对数时间内完成插入、查找、删除。但是跳跃表所占空间较大，属于空间换时间的数据结构。

## 什么是跳跃表

传统链表如果需要查询元素，需要从头遍历整个链表知道找到查询元素，时间复杂度O(n)。跳跃表通过增加多级索引的方式减少遍历长度，从而加快查找速度。跳跃表实现了平衡树、哈希表类似的效果。

为什么不使用平衡树，而是跳跃表

> [链接:link:](https://www.jianshu.com/p/8ac45fd01548)

[图片来自](https://www.pdai.tech/md/db/nosql-redis/db-redis-x-redis-ds.html#%E6%95%B4%E6%95%B0%E9%9B%86---intset)

![image-20221009184634434](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20221009184634434.png)