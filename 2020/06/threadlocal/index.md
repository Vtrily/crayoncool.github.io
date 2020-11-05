# ThreadLocal


ThreadLocal类的实现

<!--more-->

### 为什么ThreadLocalMap定义在了ThreadLocal中，而不定义在Thread中？

将ThreadLocalMap直接定义在ThreadLocal中看起来更符合逻辑，但是ThreadLocalMap并不需要Thread对象来操作，所以定义在Thread类内会增加一些不必要的开销。定义在ThreadLocal类中的原因是ThreadLocal类负责ThreadLocalMap的创建，仅当线程中设置第一个ThreadLocal时，才为当前线程创建ThreadLocalMap，之后所以其他ThreadLocal变量将使用一个ThreadLocalMap.

总的来说就是，ThreadLocalMap不是必需品，定义在Thread类中增加了成本，定义在ThreadLocal中按需创建。

ThreadLocalMap中定义了Entry[]，而Entry继承了WeakReference。Entry数组初始化容量是16，Entry是以ThreadLocal为key，Object为value的对象。

### 如何解决Hash碰撞？

使用了线性检测方法中的定址寻址法，为了减少Hash的碰撞，ThreadLocal的hashCode的生成每次会增加0x61c88647









