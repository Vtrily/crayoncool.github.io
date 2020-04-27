# 深入剖析Web技术演进


Web技术演进

<!--more-->

# TCP连接

全双工

三次握手

四次挥手

# IO模型

- 阻塞IO模型(BlockingIO)
- 非阻塞IO模型(Non-BlockingIO)
- IO复用模型(I/O multiplexing)
- 信号驱动的IO模型(SIGIO)
- 异步IO模型(Asynchronous IO)

IO分为两个阶段：1. 数据准备阶段 2.内核空间复制回用户进程缓冲区阶段

## 阻塞IO模型（BlockingIO）

![阻塞IO模型](/blog-img/1587891803983.jpg)

### 特点

在IO执行阶段的两个阶段（等待数据和拷贝数据）都被阻塞

## 非阻塞IO模型(Non-BlockingIO)

![非阻塞IO模型](/blog-img/1587896234262.jpg)

### 特点

在非阻塞式IO中，用户进程不断主动询问内核数据是否准备好

进程轮询调用，消耗CPU资源

## IO复用模型

![IO复用模型](/blog-img/1587900039436.jpg)

### 特点



## 信号驱动的IO模型(SIGIO)

![信号驱动的IO模型](/blog-img/1587900362430.jpg)

### 特点



## 异步IO模型(Asynchronous IO)

![异步IO模型](/blog-img/1587900421245.jpg)

### 特点


# tomcat流程


# 