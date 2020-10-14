# SpringMVC笔记整合

## 1、SpringMVC概念

（1）框架所属类型：JavaEE体系结构包括四层：应用层、web层、业务层、持久层。SpringMVC是Web层的框架，属于spring框架中的一部分，我们能更好的处理业务层与web层的业务需求。

（2）作用：首先SpringMVC是基于Java，实现了Web MVC设计模式，请求驱动类型的轻量级web框架，将web层进行职责解耦，SpringMVC就是要简化我们日常的web开发

​	MVC中的V就是view，使用springmvc标签库，c就是controller，M指的是Model或者ModelAndView

​	前端控制器 ：DispatcherServlet  ----- RequestMapper请求映射到url地址

​	Hander处理器 controller

​	HandlerAdapter

​	ModelAndView 包含数据以及视图

​	viewResoler 视图转换器



## 2、spring MVC使用前提

​	1、导包：

​	spring-web.jar	

​	spring-webmvc  

​	springioc基础包 

​	springaop相关包

![image-20201001110518885](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201001110518885.png)

​	2、springMVC的配置文件

​		1、扫描controller层所在的包

​		3、注解驱动 ----- 识别注解 json的注解，requestmapping注解等

​		4、视图转换器 ----- 在controller中返回的字符串会自动加上前后缀  url地址，页面地址

​	3、web.xml配置dispatcherServlet

​		1、初始化参数 ：找到springMVC的配置文件

​		2、在Tomcat服务启动后启动

​		3、拦截所有的controller的请求   <url-pattern> *</url-pattern>

​	4、编写controller类

​		1、给控制层添加@controller注解，相当于将该类注册到bean  <bean id ="" class ="" >

​		2、给方法或者类添加@RequestMapping（“url地址”）；  



## 3、springmvc传值





## 4、spingmvc页面跳转

## 5、springMVC使用json与ajax使用

## 6、springmvc的文件上传

## 7、restful风格

## 8、注解使用

## 9、核心类DispatcherServlet

