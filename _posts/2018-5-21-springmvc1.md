---
layout: post
title: Springmvc入门学习一
---
目录：
- [架构：](#架构：)
- [代码案例](#代码案例)

SpringMVC基于模型-视图-控制器模式实现，能够构建松耦合的Web应用程序。

## 架构：

![image](https://ruanwenjun.github.io/images/springmvc/springmvcjiagou.png)

**DispatcherServlet**：前端控制器，是SpringMVC的核心组件，它的任务是查询一个或多个处理器映射器，来确定将请求发送给哪个控制器。

**控制器**：获取请求的数据，然后将业务交给服务对象处理，待处理完毕后，将数据模型打包，并且标示出
用于渲染输出视图名。接下来将请求 、模型、逻辑视图名发送回DispatcherServlet。

**视图解析器**：DispatcherServlet使用视图解析器来将逻辑视图名匹配为一个特定的视图实现，可能是也可能不是JSP。


## 代码案例

(1) 配置前端控制器

前端控制器可以通过JAVA代码来实现，通过编写一个类继承AbstractAnnotationConfigDispatcherServletInitiaLizer来实现，也可以通过在web.xml来配置，这里选择在web.xml配置。
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
         http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">

    <servlet>
        <servlet-name>mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-dao.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>

```
(2) 配置Springmvc的核心组件

同样可以通过写一个java类来实现，但是这里选择使用配置xml文件来实现
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc.xsd
                        http://www.springframework.org/schema/context  
                        http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven/>

    <context:component-scan base-package="com.ruanwenjun.mvc.controller"/>

    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/view/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <mvc:resources location="/css/" mapping="/css/**"/>
</beans>
```

(3) 书写Spring配置文件

该步骤是不需要的，但是在本案例中因为需要扫描包，所以在这里配置了一个，也可以就在Springmvc.xml中配置扫描包
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ruanwenjun.mvc.dao"/>

</beans>
```
(4) 书写Controller控制器
```java
/**
 * @Author RUANWENJUN
 * @Creat 2018-05-19 15:10
 */
@Controller
public class HomeController {
    
    @Autowired
    private SpittleDao spittleDao;

    @RequestMapping("/")
    public String home(Model model) {
        List<Spittle> spittle = spittleDao.findSpittles(5);
        model.addAttribute(spittle);
        return "home";
    }

    @RequestMapping("/spittle/{spittleId}")
    public String spittle(@PathVariable long spittleId, Model model) {
        Spittle spittle = spittleDao.findSpittleById(spittleId);
        model.addAttribute("spittle", spittle);
        return "spittle";
    }

    @RequestMapping(value = "/spittle/register", method = RequestMethod.GET)
    public String showRegisterForm() {
        return "register";
    }

    @RequestMapping(value = "/spittle/register", method = RequestMethod.POST)
    public String register(Spitter spitter) {
        return "redirect:/spittle/" + spitter.getUsername();
    }

    @RequestMapping(value = "/spittle/{username}", method = RequestMethod.GET)
    public String showSpitterProfile(@PathVariable String username, Model model) {
        model.addAttribute("username", username);
        return "profile";
    }
}
```

(5) 书写Dao

 这一步同样也是不需要的，但是在本案例简单的书写，实现业务处理。这本来是应该对应数据库操作的，但是本案例没有进行数据库的操作
 
 ```java
 /**
 * @Author RUANWENJUN
 * @Creat 2018-05-19 15:22
 */
public interface SpittleDao {
    /**
     * 获得指定数目的Spittle
     * @param count
     * @return
     */
    List<Spittle> findSpittles(int count);

    /**
     * 根据Id查找
     * @param spittleId
     * @return
     */
    Spittle findSpittleById(long spittleId);
}
```
 
 ```java
 /**
 * @Author RUANWENJUN
 * @Creat 2018-05-19 15:26
 */

@Repository
public class SpittleDaoImpl implements SpittleDao {

    @Override
    public List<Spittle> findSpittles(int count) {
        List<Spittle> spittrList = new ArrayList<Spittle>();
        for (long i = 0; i < count; i++) {
            spittrList.add(new Spittle(i,"Spittle " + i, new Date()));
        }
        return spittrList;
    }

    @Override
    public Spittle findSpittleById(long spittleId) {

        return new Spittle(spittleId, "Spittle " + spittleId, new Date());
    }
}
```
(6) 用到的实体

```java
/**
 * @Author RUANWENJUN
 * @Creat 2018-05-20 21:01
 */

public class Spitter {

    private String firstName;
    private String lastName;
    private String username;
    private String password;

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

```java
/**
 * @Author RUANWENJUN
 * @Creat 2018-05-19 15:24
 */

public class Spittle {

    private final Long id;
    private final String message;
    private final Date date;
    private Double latitude;
    private Double longitude;

    public Spittle(String message, Date date) {
        this.id = null;
        this.message = message;
        this.date = date;
        this.latitude = null;
        this.longitude = null;
    }

    public Spittle(Long id, String message, Date date) {
        this.id = id;
        this.message = message;
        this.date = date;
    }

    public Long getId() {
        return id;
    }

    public String getMessage() {
        return message;
    }

    public Date getDate() {
        return date;
    }

    public Double getLatitude() {
        return latitude;
    }

    public void setLatitude(Double latitude) {
        this.latitude = latitude;
    }

    public Double getLongitude() {
        return longitude;
    }

    public void setLongitude(Double longitude) {
        this.longitude = longitude;
    }
}
```

(7) 书写JSP

书写简单的JSP页面用来显示（省）

---
需要注意的有参数接收、表单接收、资源文件映射、视图转发重定向等







