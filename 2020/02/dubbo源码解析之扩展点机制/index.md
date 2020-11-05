# dubbo源码解析之扩展点机制


本文主要分析dubbo的扩展点机制

<!--more-->

Dubbo总体分为业务层，RPC层，Remote层

# dubbo之扩展点机制

Java SPI

Dubbo SPI

Dubbo的SPI是在Java SPI的基础上实现了IOC和AOP的增强





## dubbo的扩展点机制主要有什么？

- 扩展点自动包装

Wrapper类，不是扩展点真正的实现，Wrapper类中持有真正的扩展点实现类，在Wrapper类中添加额外的逻辑，这就是所谓的AOP增强

- 扩展点自动装配

扩展点实现类的成员如果为其他扩展点类型，自动注入依赖，这就是IOC增强

- 扩展点自适应

有些扩展并不想在框架启动阶段被加载，而是希望在扩展方法被调用时，根据运行时参数进行加载
静态指定扩展点实现，SPI注解和配置

动态指定扩展点实现
@Adaptive注解

- 扩展点自动激活

对于集合类扩展点，例如Filter,每个Filter实现不同的功能，需要多个同时激活，可以用自动激活来简化配置
@Activate

## 问题

- 多个包装类只能生效一个？

并不是，它是一个嵌套关系

- 包装类内部使用哪个实现？

最底层的那个类

- 不加载包装类AOP会不会失效？

自动包装

- 自激活扩展类如何构建链表？

- 自激活扩展类的链表节点顺序？

## 源码分析

根据LoadBalance的加载机制分析扩展机制

获取扩展类实例
getExtension
getDefaultExtension
getAdaptiveExtension
getActivateExtension

getExtension流程

1.加载缓存
2.加载扩展类信息
3.实例化扩展类
4.注入依赖
5.查找包装类注入扩展类实例
6.返回

getAdaptiveExtension
1.加载缓存
2.加载扩展类信息
3.生成代理类代码
4.实例化
5.注入依赖
6.返回

getActivateExtension
1.加载所有扩展类
2.获取扩展类注解信息
3.创建匹配的扩展类实例
4.排序扩展类实例
5，返回

getExtensionClasses()

双重检查，因为加锁之前可能会被注入值，因为场景是高并发的


