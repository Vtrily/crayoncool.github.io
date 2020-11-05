# ZooKeeper深入分析之zk是如何保障数据一致性的


ZooKeeper深入分析之zk是如何保障数据一致性的

<!--more-->

数据一致性保障

## Zab协议

- Zookeeper Atomic Broadcast

- Leader负责处理写入请求

- 两阶段提交



提交的时候需要在保证数据的一致，一致性可以分为下面三种情况
强一致性，顺序一致性，最终一致性

