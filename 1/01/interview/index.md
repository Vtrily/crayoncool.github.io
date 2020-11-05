# 

# Java
1. 泛型实现、多态实现
2. equal、hashcode、==区别
3. try、catch、finally执行顺序，finally执行return语句会发生什么
直接返回
4. 引用类型有哪几类，各种的用途，threadLocal是什么类型
强引用，软引用，弱引用，虚引用，threadlocal是弱引用
5. SPI实现
Service Provider Interface，在项目resource目录下的META-INF文件夹下创建一个类路径命名的文件，文件内是这个类的扩展类的实现类的全路径

Dubbo SPI的实现是在java SPI的基础上做了AOP和IOC的扩展
主要是扩展类的自动装配，自动包装，自适应@Adaptive，自激活@Active

6. hashmap大小为什么是2的n次方

7. 多线程单例实现


8. happen before
9. CAS、Lock、AQS、synchronized
10. nio底层实现
11. 类加载器、类加载过程、什么时候会触发类
12. 对象创建过程、对象结构
13. JVM内存结构、JVM内存模型、volatile
14. JVM垃圾收集过程
15. 调优分析（堆转存快照分析、调优命令、工具）

# Spring
1. IOC初始化流程、循环依赖解决
2. SpringMVC处理流程
3. AOP原理

# 操作系统
1. 进程和线程区别、线程同步机制、进程间通信
一个进程中有多个线程
2. IO多路复用
3. awk、sed命令

# 网络
1. TCP三次握手四次挥手
2. TCP、UDP区别和使用场景
一个可靠一个不可靠
3. HTTP2.0相比HTTP1.1的优点
4. HTTPS建立连接过程
5. 浏览器发出请求到接受响应的详细过程

# Mysql
1. 数据库范式
2. 事务、锁、MVCC
3. 索引最左匹配，b+tree搜索过程
4. 分库分表
5. 查询优化方法

# mongo
1. 关系、非关系型数据库区别，各自优缺点
2. mongo优势，适用场景
3. 分片实现
 
# 缓存
1. redis分布式锁设计
2. redis分布式限流设计
3. redis高可用实现
4. 缓存和DB数据一致性保障
5. 缓存雪崩、穿透

# 消息
1. 消息队列解决什么问题（解耦、削峰、异步）
2. 高可用实现
3. 如何保证不被重复消费
4. 如何保证可靠性
5. 如何保证顺序性

# 算法
1. 用两个线程顺序打印0到10的奇偶数

```java
public class PrintNum implements Runnable {

    private Boolean runNow;

    private Object o;

    private Integer num;

    public PrintNum(Boolean runNow, Object o, Integer num) {
        this.runNow = runNow;
        this.o = o;
        this.num = num;
    }
    @Override
    public void run() {
        synchronized(o) {
            while (num <= 100) {
                if (runNow) {
                    runNow = false;
                } else {
                    try {
                        o.wait();
                    } catch (InterruptedException e) {

                    }
                }
                System.out.println(num);
                num += 2;
                o.notify();
            }
        }
    }

    public static void main(String[] args) {
        Object o = new Object();
        Thread t1 = new Thread(new PrintNum(false, o, 2));
        Thread t2 = new Thread(new PrintNum(true, o, 1));
        t1.start();
        t2.start();

        
    }
}
```
2. LRU算法实现
3. 先进先出环形队列
4. 堆排序
5. 第2（k）大数
6. 合并两个有序数组、合并k个有序数组
7. 删除单向链表的倒数第k个节点
快慢节点
8. 二叉树遍历（递归、while）
9. 给两个整数数组 A 和 B ，返回两个数组中公共的、长度最长的子数组的长度
10. 给定一个正整数 n，将其拆分为至少两个正整数的和，并使这些整数的乘积最大化，返回最大乘积
11. 计算从 0 到 n (含 n) 中数字 2 出现的次数（22计两次）
 
# 分布式
1. CAP理论

数据一致性，服务可用性，分区容错性只能满足其二

为什么只能满足其二

只要有网络，必然会有网络中断的问题，需要保证网络恢复过来服务可用

CP,AP

数据库主从部署问题，ZK



2. 一致性hash
3. 分布式事务
4. 微服务组成部分（网关、服务发现、服务注册、限流、熔断降级）
5. docker、k8s

# 系统设计
1. 短链服务设计
2. 秒杀设计
3. 高并发下统计最大值、最小值、平均值
4. 延时队列
双层时间轮