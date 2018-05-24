---
layout: post
title: springmvc文件上传、异常处理器
---
* [图片长传](#图片长传)
* [异常处理器](#异常处理器)

## 图片上传

(1) StandardServletMultipartResolver(推荐使用)
1.需要在web.xml的前端控制器中配置临时文件的路径，同时还可以配置上传大小之类的

```
<servlet>
    <servlet-name>mvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <multipart-config>
        <location>E:\\mvcPicture\\tem</location>
    </multipart-config>
</servlet>
<servlet-mapping>
    <servlet-name>mvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

2.在springmvc.xml中配置解析器

```
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/>
```

同样，这里的id不能改

3.然后就可以在控制器中接收文件
```java
@RequestMapping(value = "/spittle/register", method = RequestMethod.POST)
    public String register(@RequestPart("profilePicture") MultipartFile multipartFile,
                           Spitter spitter, RedirectAttributes model) throws IOException {
        multipartFile.transferTo(new File("E:\\mvcPicture\\" + multipartFile.getOriginalFilename()));
        model.addFlashAttribute("profilePicture", "/pic/" + multipartFile.getOriginalFilename());
        return "redirect:/spittle/" + spitter.getUsername();
    }
```
注意，这里的RedirectAttributes是用来在转发的过程中传递数据的，它其实就是将数据放在会话中，使用addFlashAttribute方法，当转发后的请求结束时，会自动清除会话中的数据。

(2) 使用CommonsMultipartResolver

1.导入额外的包
- commons-fileupload-1.2.2.jar
- commons-io-2.4.jar
2.在springmvc.xml中配置文件解析器

```
<!--文件长传的解析器  -->
    <!--这里的ID好像不能更改-->
	<bean id="multipartResolver" 
	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="maxUploadSize" value="500000"></property>
	</bean>
```
3.表格使用post提交方式，并修改enctype属性

```
<form id="itemForm"	
    action="${pageContext.request.contextPath }/updateitem.action" method="post
		enctype="multipart/form-data">
```
4.书写测试代码

```java
//更新一个商品
@RequestMapping("updateitem.action")
public ModelAndView itemUpdate(Items item,MultipartFile pictureFile)
    throws IllegalStateException, IOException {
    //获得文件名
	String name = pictureFile.getOriginalFilename();
	//获得文件扩展名
	String extension = FilenameUtils.getExtension(name);
	//创建文件名称
	name = UUID.randomUUID().toString().replaceAll("-", "")+"."+extension;
	//将文件复制到电脑
	pictureFile.transferTo(new File("E:\\ecplices _workspace\\"
	    +"simplework\\ssm\\upload\\"+name));
	    
	item.setPic(name);
	is.itemUpdate(item);
	ModelAndView mav = new ModelAndView();
	mav.setViewName("success");
	return mav; 
}
```

---
测试成功

这里配置了Tomcat里面的虚拟目录
![image](https://ruanwenjun.github.io/images/QQ图片20180410161626.png)，所以选择将文件长传到该目录，然后可以通过localhost:8080/pic/文件名来获得图片

## 异常处理器
(1) 使用@ControllerAdvice注解
```java
@ControllerAdvice
public class ErrorController  {
    @ExceptionHandler(IOException.class)
    public String error(Model model){
        model.addAttribute("error","文件长传异常");
        return "error";
    }
}
```
这个注解使用之后，会自动加载该对象

(2) 配置HandlerExceptionResolver

1. 书写异常处理器类

```java
package cn.ruanwenjun.exception;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

/**
 * @author ruanwenjun E-mail:861923274@qq.com
 * @date 2018年4月10日 下午3:46:46
*/
//异常处理器
public class CustomeException implements HandlerExceptionResolver{
    //ex是捕获到的异常
	public ModelAndView resolveException(HttpServletRequest request,
	HttpServletResponse response, Object handler,
			Exception ex) {
		
		ModelAndView mav = new ModelAndView();
		mav.addObject("error", ex.getMessage());
		mav.setViewName("error");
		return mav;
	}

}

```
2. 配置实例化异常处理器，在springmvc.xml中配置

```java
<bean  id = "customeException"
class="cn.ruanwenjun.exception.CustomeException"/>
```
测试通过

