---
layout: post
title: Spring中的Import注解
tags: [Spring]
---
目录：
- [问题描述](#问题描述)
- [案例](#案例)
- [实现过程](#实现过程)
- [总结](#总结)

# 问题描述

在使用Spring框架的时候，我们可以很方便的在容器中注入我们自己写的Bean。<br/>
例如使用@Component等注解，或者在@Configuration标注的配置类中使用@Bean注解，或者其他的工厂之类的方式。<br/>
但是如果是希望注入第三方包中的对象，那么@Component注解就无法使用了，当然还是可以使用@Bean这种方式。但是这种方式需要使用者去写代码，比较不那么优雅。有没有一种办法，能够让使用者只添加一个注解就可以实现注入三方包中的对象呢？<br/>
当然是可以的，那就是在配置类上使用@Import注解，这种方式其实就是很多@Enablexx注解的实现原理，在Springboot，Springcloud中很常见。

# 案例

例如，现在使用@Import向容器中注入一个Cat对象，可以使用如下代码：<br/>
```java
@Configuration
@Import(value = {Cat.class})
public class ImportConfig {
}

public class Cat{

}

public class ImportConfigTest {

	@Test
	public void test() {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(ImportConfig.class);
		String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
		for (String name : beanDefinitionNames) {
			System.out.println(name);
		}
	}

}
```
输出结果中发现，确实将Cat对象注入进了容器。<br/>
其实@Import注解的注入方式还有ImportSelector，ImportBeanDefinitionRegistrar。可以实现这两个接口，然后加入@Import的value中，具体可以查看@Import的注释。

# 实现过程
那么@Import注解的实现过程是怎样的呢？通过Debug发现，其实是通过解析@Configuration的时候，通过找到@Configuration标注的配置类上的@Import注解，从而进行注入。(废话）<br/>

具体的的调用栈如下：<br/>
```java
AnnotationConfigApplicationContext.refresh()
    ->invokeBeanFactoryPostProcessors(beanFactory) //调用BeanFactory后置处理器
        ->ConfigurationClassPostProcessor.processConfigBeanDefinitions()// 该后置处理器就是去解析Configuration配置类
            ->ConfigurationClassParser.collectImports()// 会收集所有的@Import中的class,放到ConfigurationClass的resource属性中
    ->finishBeanFactoryInitialization()这一步才会真正创建Cat对象
```
中间省略了很多步，只显示了大致的逻辑

# 总结

Spring是一个不错的框架，里面有很多很好用的功能，几乎所有的功能都是通过一些后置处理器实现的，同时这些后置处理器的设置，很便于后续版本添加功能，这种"设计模式"很值得借鉴，在代码中设置模版，将所有的策略组件化，通过设置、替换组件实现不同的功能。
