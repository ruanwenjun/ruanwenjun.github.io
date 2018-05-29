---
layout: post
title: Spring----Bean的装配
tags: [Spring]
---
目录：
* [隐式的Bean发现机制和自动装配](#隐式的bean发现机制和自动装配)
* [基于Java的显示装配](#基于java的显示装配)
* [使用XML进行显示装配](#使用xml进行显示装配)

Spring的一个重要特性就是控制反转（IOC)。一个应用肯定是由许多相互依赖的对象组成，对象要想获得他所依赖的对象，那么需要new一个对象，例如 一个学校需要一个老师，那么需要在学校里面new一个老师，而这种通过new 对象的方式来获取依赖对象，对象之间的耦合度就很高。而IOC就是让Spring容器来管理对象，将对象的的依赖关系交给容器来实现，从而对象如何得到他的依赖对象的责任被反转了，从而达到解耦的目的。

Spring中实现IOC的技术是DI，通过注入的方式将依赖对象注入。

Spring装配Bean的三种方式

- 隐式的Bean发现机制和自动装配(首选)
- 基于Java的显示装配
- 使用XML进行显示装配

## 隐式的Bean发现机制和自动装配

通过组件扫描和自动装配实现。
只需要在类上面加上注解然后开启组件扫面即可，ApplicationContext和BeanFactory会创建实例对象

```java
@Configuration
@ComponentScan
public class Config {

}

@Component
public class Gril {

    @Value("TangYan")
    private String name;
    @Value("10")
    private int age;

    @Override
    public String toString() {
        return "Gril{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}


@Component
public class Boy {

    @Autowired
    private Gril gril;

    @Override
    public String toString() {
        return "Boy{" +
                "gril=" + gril +

                '}';
    }
}

public class App {
    public static void main(String[] args) throws IOException {
        ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        Boy boy = context.getBean(Boy.class);
        System.out.println(boy);
    }
}

打印：Boy{gril=Gril{name='TangYan', age=10}}
```
## 基于Java的显示装配

有的时候，如果需要将第三库中的组件加入到Spring容器中，使用第一种方法实现不了，你总不能到第三方库的源码上面加注解吧，那么时候可以使用Java代码或者XML的方式进行Bean的装配。

```java
@Configuration
public class Config {

    @Bean
    public Gril gril(){
        return new Gril("TangYan",10);
    }
    @Bean
    public Boy boy(){
        return new Boy(gril());
    }
}

public class Gril {

    private String name;
    private int age;

    public Gril(String name,int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Gril{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

public class Boy {

    private Gril gril;

    public Boy(Gril gril) {
        this.gril = gril;
    }

    @Override
    public String toString() {
        return "Boy{" +
                "gril=" + gril +

                '}';
    }
}

public class App {

    public static void main(String[] args) throws IOException {
        ApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        Boy boy = context.getBean(Boy.class);
        System.out.println(boy);

    }
}

打印：Boy{gril=Gril{name='TangYan', age=10}}
```
## 使用XML进行显示装配

```java

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="boy" class="com.spring.beandemo.Boy">
        <property name="gril" ref="gril"/>
    </bean>
    <bean id="gril" class="com.spring.beandemo.Gril">
        <property name="name" value="Tangyan"/>
        <property name="age" value="10"/>
    </bean>
</beans>

public class Boy {

    private Gril gril;

    public void setGril(Gril gril) {
        this.gril = gril;
    }

    @Override
    public String toString() {
        return "Boy{" +
                "gril=" + gril +

                '}';
    }
}

public class Gril {

    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Gril{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

public class App {

    public static void main(String[] args) throws IOException {
        ApplicationContext context = new FileSystemXmlApplicationContext("classpath:applicationContext.xml");
        Boy boy = context.getBean(Boy.class);
        System.out.println(boy);

    }
}
```

---

通常以上方式可以混合使用，另外，Bean的配置还涉及到条件化的Bean装配，Bean作用域设置，自动装配的歧义等。但都可以通过注解或者XML配置
