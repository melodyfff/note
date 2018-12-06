---
title: 【Java】 SpringBoot + Servlet 3.0 注解使用笔记
tags: [Java]
date: 2018-12-06
---

> [spring-boot 2.11](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/)   
> [JSR-000315 Java Servlet3.0规范](https://jcp.org/aboutJava/communityprocess/final/jsr315/index.html)


## Servlet3.0与Servlet2.5的区别
  
- 异步处理支持
- 新增的注解支持
- 可拔插性支持  



## @WebServlet、@WebFilter、@WebListener

> @WebServlet, @WebFilter, and @WebListener annotated classes can be automatically registered with an embedded servlet container by annotating a @Configuration class with @ServletComponentScan and specifying the package(s) containing the components that you want to register. By default, @ServletComponentScan scans from the package of the annotated class.

#### ServletConfig
```java
@Configuration
@ServletComponentScan
public class ServletConfig {}
```


#### @WebServlet
```java
@WebServlet(name = "hello", 
urlPatterns = {"/hello"}, 
loadOnStartup = 1, 
description = "this is a test")
public class TestServlet extends HttpServlet {}
```

#### @WebFilter
```java
/**
 * Filter的执行顺序为类名匹配 A-Z
 * AFilter 比 BFilter先执行
 */
@WebFilter(filterName = "testFilter"
,urlPatterns = "/*")
public class TestFilter implements Filter {}
```

#### @WebListener
```java
/**
 * Request:
 * ServletRequestListener : 监听Request的创建和销毁
 * ServletRequestAttributeListener : 监听Request的属性,新增属性、移除属性和属性值被替换时
 *
 * Session:
 * HttpSessionListener : 监听HttpSession的创建跟销毁
 * HttpSessionAttributeListener : 监听session中属性,新增属性、移除属性和属性值被替换时
 *
 * ServletContext:
 * ServletContextListener : 监听ServletContext的创建和销毁
 * ServletContextAttributeListener : 监听ServletContext中属性的新增、移除和属性值的替换
 */
@WebListener
public class TestRequestListener implements ServletRequestListener {}
```