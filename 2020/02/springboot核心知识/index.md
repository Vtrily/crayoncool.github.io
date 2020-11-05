# SpringBoot核心知识


SpringBoot启动原理

<!--more-->


## SpringBoot启动原理

源码基于SpringBoot2.2.0.RELEASE

首先我们看SpringBoot的启动类代码

```
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

```

从代码中我们可以看到，SpringBoot启动类中有着@SpringBootApplication注解和SpringApplication.run()方法

我们先来看@SpringBootApplication注解的作用

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
@ConfigurationPropertiesScan
public @interface SpringBootApplication {

	
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

	
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;

}
```
@Target：这个注解能被用到什么元素上，元素可以多个。ElementType.TYPE能被用到类，接口，声明枚举上

@Retention：这个注解是说这个注解能被保留多久，一共有三种
1. SOURCE,只保留在源码上
2. CLASS,注解通过编译器记录在类文件上。默认行为
3. RUNTIME,注解通过编译器记录在类文件上，运行时可通过反射获取到

@Inherited：通过Inherited元注解声明的自定义注解，在类上使用时，可以被子类继承

以上三个注解往往在自定义注解的时候使用较多

@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan


