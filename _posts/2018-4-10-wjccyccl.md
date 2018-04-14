---
layout: post
title: springmvc文件上传、异常处理器!
---
* [图片长传](#图片长传)
* [异常处理器](#异常处理器)

# 图片上传
1. 导入额外的包
- commons-fileupload-1.2.2.jar
- commons-io-2.4.jar
2. 在springmvc.xml中配置文件解析器

```java
<!--文件长传的解析器  -->
    <!--这里的ID好像不能更改-->
	<bean id="multipartResolver" 
	class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="maxUploadSize" value="500000"></property>
	</bean>
```
3. 表格使用post提交方式，并修改enctype属性

```java
<form id="itemForm"	
    action="${pageContext.request.contextPath }/updateitem.action" method="post
		enctype="multipart/form-data">
```
4. 书写测试代码

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

# 异常处理器
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

