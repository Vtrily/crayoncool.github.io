# zookeeper源码解析

# dubbo之扩展点机制

## dubbo的扩展点机制主要有什么？

- 扩展点自动包装

- 扩展点自动装配

- 扩展点自适应

- 扩展点自动激活

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


