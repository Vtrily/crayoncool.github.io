# Synchronized基础


HashMap和HashTable的安全性有什么区别？

<!--more-->

HashMap是线程不安全的，HashTable是线程安全的

HashTable的线程安全是因为它的put方法使用synchronized进行修饰的。


### 为什么会有线程安全问题？

出现线程安全问题一般是因为主内存和工作内存数据不一致和重排序导致的，

### Synchronized的作用

对于普通同步方法，锁是当前实例对象

对于静态同步方法，锁是当前类的Class对象

对于同步方法块，锁是Synchronized括号里配置的对象

代码块同步是用monitorenter和monitorexit指令实现的。

为了减少获得锁和释放锁带来的性能消耗，引入了偏向锁和轻量级锁。
Java SE1.6中，锁有4种状态，无锁状态、偏向锁、轻量级锁、重量级锁状态


使用-XX:-UseSpinning参数关闭自旋锁优化；-XX:PreBlockSpin参数修改默认的自旋次数

JDK1.6默认自旋次数是10次

JDK1.6之后引入了自适应自旋锁，自适应自旋锁是根据上一次自旋的时间和次数决定的

锁消除：

```
public void add(String str1){
    StringBuffer sb = new StringBuffer("hello");
    sb.append(str1);
}
```

StringBuffer 是线程安全的，因为内部的关键方法被sunchronized修饰，但是上述代码中，不存在竞争共享资源的现象，所有JVM会自动消除StringBuffer中的锁