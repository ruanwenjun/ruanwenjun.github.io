---
layout: post
title: 动态代理和cglib代理
tags: [Spring]
---
> Spring使用动态代理和动态字节码生成技术(Cglib代理)来实现AOP

目录：
* [动态代理](#动态代理)
* [cglib代理](#cglib代理)

## 动态代理

代理模式是指在访问者与被访问者之间加入一层代理，从而可以减少被访问者的负担，同时也可以对访问进行一些拦截、过滤等操作。

代理模式是基于接口实现的，即代理与被代理对象必须实现同一接口。代理模式又分为静态代理和动态代理。

静态代理实现过程比较简单，就是代理类和被代理类实现同一接口，代理内部保存有被带理对象的具体实例，外部通过访问代理对象访问被带理对象。

动态代理是指为指定接口在运行期间动态的生成代理对象。动态代理的实现主要由Proxy类和InvocationHandler接口负责
```java
/**
 * Request接口
 *
 * @Author RUANWENJUN
 * @Creat 2018-06-05 19:55
 */

public interface MyRequest {
    /**
     * 发出请求
     */
    void request();
}


/**
 * 具体请求
 * @Author RUANWENJUN
 * @Creat 2018-06-05 19:58
 */

public class MyRequestImpl implements MyRequest {
    @Override
    public void request() {
        System.out.println("I want to send a request !!!!");
    }
}

/**
 * @Author RUANWENJUN
 * @Creat 2018-06-05 19:56
 */

public class MyRequestInvocationHandler implements InvocationHandler {
    private Object o ;
    public MyRequestInvocationHandler(Object o){
        this.o = o;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if(method.getName().equals("request")){
            System.out.println("Hello, I'm Proxy. Time is " + new Date());
        }
        return method.invoke(o,args);
    }
}

**
 * @Author RUANWENJUN
 * @Creat 2018-06-05 19:59
 */

public class ProxyDemo {
    public static void main(String[] args) {
        MyRequest myRequest = (MyRequest) Proxy.newProxyInstance(
                ProxyDemo.class.getClassLoader(), new Class[]{MyRequest.class},
                new MyRequestInvocationHandler(new MyRequestImpl()));

        myRequest.request();
    }
}

输出：
Hello, I'm Proxy. Time is Tue Jun 05 20:37:59 CST 2018
I want to send a request !!!!
```
动态代理只能对实现了相应接口的类进行代理。如果一个类没有实现任何接口，那么就不能对其进行代理。


## cglib代理

cglib可以为没有实现接口的类进行代理,底层似乎是通过ASM动态修改字节码技术实现的。要使用cglib代理需要导入第三方包
- cglib-3.2.6.jar
- asm-all-6.0_BETA.jar

```java
/**
 * @Author RUANWENJUN
 * @Creat 2018-06-05 20:09
 */

public class Response {
    public void response(){
        System.out.println("Hello, I want to send a response. Time is" + new Date());
    }
}

/**
 * @Author RUANWENJUN
 * @Creat 2018-06-05 20:11
 */

public class ResponseCallback implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        if(method.getName().equals("response")){
            System.out.println("Hello, I am CGLIB");
        }

        return methodProxy.invokeSuper(o,args);
    }
}

/**
 * @Author RUANWENJUN
 * @Creat 2018-06-05 20:19
 */

public class CglibDemo {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();

        enhancer.setSuperclass(Response.class);
        enhancer.setCallback(new ResponseCallback());

        Response proxy = (Response) enhancer.create();
        proxy.response();
    }
}

输出：
Hello, I am CGLIB
Hello, I want to send a response. Time isTue Jun 05 20:42:49 CST 2018
```

由于cglib是动态创建子类的方式实现代理，所以缺点是不能对final或private方法进行代理。

SpringAOP默认使用动态代理实现，当一个类没有实现接口的时候，会使用cglib代理。Cglib代理所创建的对象性能比jdk动态代理性能高，但是创建时花费的时间也要长。
