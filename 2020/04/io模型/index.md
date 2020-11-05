# IO模型


UNIX环境中的IO模型分析

<!--more-->

首先，声明一下IO发生时涉及的对象和步骤。对于一个network IO,它会涉及到两个系统对象，用户和系统。当发生一个read操作时，会有以下两个阶段：

```
1、数据准备阶段
2、将数据从内核拷贝到进程中
```

# IO模型

- 阻塞IO模型(BlockingIO)
- 非阻塞IO模型(Non-BlockingIO)
- IO复用模型(I/O multiplexing)
- 信号驱动的IO模型(SIGIO)
- 异步IO模型(Asynchronous IO)

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

