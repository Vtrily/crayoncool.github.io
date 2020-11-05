# CAS



CAS总结

<!--more-->

CAS即compare and swap,比较并交换

CAS思想：通过三个值来控制多线程执行的安全性。内存中的值V，预期的原始值A，希望修改的新值B，只有当V=A时返回true,V!=A时返回false

CAS中的底层实现是通过调用Unsafe类中的方法进行实现的。Unsafe类是jdk提供的直接操作底层的方法，Unsafe类中都是public方法

jdk中的CAS应用时AtomicInteger

AtomicInteger中的变量值是使用volatile修饰的，保证了数据内存可见性

AtomicInteger中getAndIncrement方法调用了Unsafe中的getAndAddInt方法

```
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    return v;
}
```

CAS存在的问题是ABA问题，解决ABA问题的方法是增加版本控制，jdk中的解决类是AtomicStampedReference

i++操作是原子性的吗？
不是，通过查看编译之后的字节码可以发现i++，是分为三步进行操作的
1.先拿到i
2.i+1
3.对i进行赋值