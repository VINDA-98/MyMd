# SSM框架整合（IDEA版）

​	**学完Spring+springMVC+Mybatis，接下来就整合一下这三个框架，**

​	**这个基础框架搭建好了，以后就直接可以那做启动模板用了，动手动手！**

<br>





[TOC]

<br>

<hr />



## 1、新建项目

![IDEA中新建Maven项目](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002110120175.png)

<br>

起个好听的名字哈

![image-20201002110216801](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002110216801.png)



<br>

确认好信息

![image-20201002110306475](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002110306475.png)



<br>

在src下右键添加新的文件，因为对maven项目的目录结构要求很严，IDEA会提醒用户在改文件夹下最好创建什么项目

![image-20201002163111704](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002163111704.png)

<br>

自动导入后就完成项目新建啦

![image-20201002110428772](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002110428772.png)

<br>



## 2、项目路径分配

controller ： 控制层代码

service      :    业务层代码

dao    		： 持久层代码

entity  	   :   实体类代码

db.proerties 	：数据库配置信息

log4j.proerties :  日志配置信息

spring-mvc.xml：springmvc相关配置信息

spring-mybatis.xml：spring整合mybaits配置文件

mapper文件：存放实体类对应的mapper映射文件

views ： 存放jsp视图文件



**注意分级**

![image-20201002154645659](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002154645659.png)

<br>

## 3、导入POM.XML依赖信息

```xml
 <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- spring版本号 -->
    <spring.version>4.3.6.RELEASE</spring.version>
    <!-- mybatis版本号 -->
    <mybatis.version>3.5.3</mybatis.version>
  </properties>

  <dependencies>
    <!-- mybatis/spring整合包 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>2.0.3</version>
    </dependency>

    <!-- 单元测试 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>

    <!--  文件上传依赖-->
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.6</version>
    </dependency>

    <dependency>
      <artifactId>commons-fileupload</artifactId>
      <groupId>commons-fileupload</groupId>
      <version>1.3.3</version>
    </dependency>
    <!-- springMVC相关end -->


    <!--  阿里数据库连接池-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.19</version>
    </dependency>

    <!--    jstl-->
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>


    <!-- 实现slf4j接口并整合 -->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.2</version>
    </dependency>

    <!-- JSON -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.8.7</version>
    </dependency>


    <!-- 数据库 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.41</version>
      <scope>runtime</scope>
    </dependency>

    <!-- MyBatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>

    <!-- Spring -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.12</version>
      <scope>provided</scope>
    </dependency>

  </dependencies>
```





## 4、各种XML文件配置

### （1）web.xml

​	

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <!-- 编码过滤器 -->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <!-- 配置DispatcherServlet -->
  <servlet>
    <servlet-name>SpringMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 配置springMVC需要加载的配置文件-->
    <init-param>

      <param-name>contextConfigLocation</param-name>
      <!-- 记得给Spring配置文件、SpringMVC配置文件都放行-->
      <param-value>classpath:spring-*.xml</param-value>
    </init-param>

    <load-on-startup>1</load-on-startup>
    <async-supported>true</async-supported>
  </servlet>
  <servlet-mapping>
    <servlet-name>SpringMVC</servlet-name>
    <!-- 匹配所有请求 -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>


  <!-- druid连接池 启用 Web 监控统计功能    start-->
  <filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    <init-param>
      <param-name>exclusions</param-name>
      <param-value>*.js ,*.gif ,*.jpg ,*.png ,*.css ,*.ico ,/druid/*</param-value>
    </init-param>
  </filter>

  <filter-mapping>
    <filter-name>DruidWebStatFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <servlet>
    <servlet-name>DruidStatView</servlet-name>
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>DruidStatView</servlet-name>
    <url-pattern>/druid/*</url-pattern>
  </servlet-mapping>
  <!-- druid连接池 启用 Web 监控统计功能    end-->

</web-app>
```

<br>

### （2）spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

    <!-- 扫描web相关的bean -->
    <context:component-scan base-package="top.weidaboy.controller"/>

    <!-- 开启SpringMVC注解模式 -->
    <mvc:annotation-driven/>

    <!-- 静态资源默认servlet配置 -->
    <mvc:default-servlet-handler/>
    <mvc:resources mapping="/static/**" location="/WEB-INF/static/" />

    <!-- 配置jsp 显示ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <!--        设置访问web-inf下的静态资源文件，减轻服务器加载负担，限制用户通过url直接访问到jsp文件
                    在web-inf文件下用转发forword  不要用重定向redirect  -->
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean id="multipartResolver"
          class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!--设置字符编码防止文件名乱码 -->
        <property name="defaultEncoding" value="utf-8" />
        <!--设置上传文件的总大小，单位是字节b -->
        <property name="maxUploadSize" value="2048576" />
        <property name="resolveLazily" value="true" />
    </bean>

</beans>
```

<br>

### （3）spring-mybatis.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 扫描service包下所有使用注解的类型 -->
    <context:component-scan base-package="top.weidaboy.service"/>

    <!-- 配置数据库相关参数properties的属性：${url} -->
    <context:property-placeholder location="classpath:db.properties"/>

    <!-- 阿里 druid 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <!-- 数据库基本信息配置 -->
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <property name="filters" value="${jdbc.filters}"/>
        <!-- 最大并发连接数 -->
        <property name="maxActive" value="${jdbc.maxActive}"/>
        <!-- 初始化连接数量 -->
        <property name="initialSize" value="${jdbc.initialSize}"/>
        <!-- 配置获取连接等待超时的时间 -->
        <property name="maxWait" value="${jdbc.maxWait}"/>

        <!-- 最小空闲连接数 -->
        <property name="minIdle" value="${jdbc.minIdle}"/>

        <!-- 最大空闲连接数 -->
        <property name="maxIdle" value="${jdbc.maxIdle}"/>

        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="${jdbc.timeBetweenEvictionRunsMillis}"/>

        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="${jdbc.minEvictableIdleTimeMillis}"/>
        <property name="validationQuery" value="${jdbc.validationQuery}"/>
        <property name="testWhileIdle" value="${jdbc.testWhileIdle}"/>
        <property name="testOnBorrow" value="${jdbc.testOnBorrow}"/>
        <property name="testOnReturn" value="${jdbc.testOnReturn}"/>
        <property name="maxOpenPreparedStatements" value="${jdbc.maxOpenPreparedStatements}"/>

        <!-- 超过时间限制是否回收 -->
        <property name="removeAbandoned" value="${jdbc.removeAbandoned}"/>

        <!-- 1800 秒，也就是 30 分钟 -->
        <property name="removeAbandonedTimeout" value="${jdbc.removeAbandonedTimeout}"/>

        <!-- 关闭 abanded 连接时输出错误日志 -->
        <property name="logAbandoned" value="${jdbc.logAbandoned}"/>
    </bean>


    <!-- 配置SqlSessionFactory对象 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 扫描entity包 使用别名 -->
        <property name="typeAliasesPackage" value="top.weidaboy.entity"/>
        <!-- 扫描sql配置文件:mapper需要的xml文件 -->
        <property name="mapperLocations"  value="classpath:mapper/*.xml"/>
    </bean>

    <!-- mapper扫描器 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="top.weidaboy.dao"></property>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    &lt;!&ndash; 通知 &ndash;&gt;-->
<!--    <tx:advice id="txAdvice" transaction-manager="transactionManager">-->
<!--        <tx:attributes>-->
<!--            &lt;!&ndash; 传播行为 &ndash;&gt;-->
<!--            <tx:method name="save*" propagation="REQUIRED" />-->
<!--            <tx:method name="insert*" propagation="REQUIRED" />-->
<!--            <tx:method name="add*" propagation="REQUIRED" />-->
<!--            <tx:method name="create*" propagation="REQUIRED" />-->
<!--            <tx:method name="delete*" propagation="REQUIRED" />-->
<!--            <tx:method name="update*" propagation="REQUIRED" />-->
<!--            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>-->
<!--            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>-->
<!--            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>-->
<!--        </tx:attributes>-->
<!--    </tx:advice>-->

<!--    &lt;!&ndash; 切面 &ndash;&gt;-->
<!--    <aop:config>-->
<!--        &lt;!&ndash;切入点必须是在service层&ndash;&gt;-->
<!--        <aop:advisor advice-ref="txAdvice"-->
<!--                     pointcut="execution(* top.weidaboy.service.*.*(..))" />-->
<!--    </aop:config>-->

    <!-- 配置基于注解的声明式事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```

<br>



## 5、配置文件

### （1）db.properties

```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/study?useUnicode=true&characterEncoding=utf8&useSSL=true
jdbc.username=root
jdbc.password=weida
jdbc.filters=stat
jdbc.maxActive=50
jdbc.initialSize=1
jdbc.maxWait=60000
jdbc.minIdle=10
jdbc.maxIdle=15
jdbc.timeBetweenEvictionRunsMillis=60000
jdbc.minEvictableIdleTimeMillis=300000
jdbc.validationQuery=SELECT 'x'
jdbc.testWhileIdle=true
jdbc.testOnBorrow=false
jdbc.testOnReturn=false
jdbc.maxOpenPreparedStatements=20
jdbc.removeAbandoned=true
jdbc.removeAbandonedTimeout=1800
jdbc.logAbandoned=true
```



### （2）log4j.properties

```properties
log4j.properties
log4j.rootLogger=trace,console

log4j.appender.console= org.apache.log4j.ConsoleAppender
log4j.appender.console.Target= System.out
log4j.appender.console.layout= org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%-5p][%d{yyyy-MM-ddHH:mm:ss}]%m%n
```



## 6、其他配置

### 	（1）entity

```java
@Data
public class Student {
    private Long id;
    private String sanme;
    private String password;
}
```

### （2）dao

```java
public interface StudentDao {
    Student login(Student student);
}
```

### （3）service

```java
public interface StudentService {
    Student login(Student student);
}
```

<br>

```java
@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentDao studentDao;

    @Override
    public Student login(Student student) {
        return studentDao.login(student);
    }
}
```

### （4）controller

```java
@Controller
@RequestMapping("/student")
public class StudentController {

    @Autowired
    private StudentService studentService;

    @RequestMapping("/login")
    @GetMapping
    public String login(Student student){
        return studentService.login(student).toString();
    }

    @RequestMapping("/")
    public String index(){
        return "test";
    }
}
```

### （5）test.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>这是首页</title>
</head>
    <body>
         <h1>加油</h1>
    </body>
</html>
```

<br>

## 7、配置tomcat

### （1）添加tomcat配置

![image-20201002160059092](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002160059092.png)

<br>



### （2）添加项目信息

![image-20201002160208906](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002160208906.png)

<br>

![image-20201002160005700](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002160005700.png)

<br>

### （3）修改tomcat配置

![image-20201002160317896](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002160317896.png)

<br>

## 8、启动程序

#### 测试成功！

启动成功后，在浏览器地址栏输入  ==ip+端口号+请求url== 就可以看到结果啦

![image-20201002161015257](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002161015257.png)



<br>

## 9、小结

**本次碰到的坑：**
1、
org.springframework.beans.factory.BeanCreationException: Error creating bean with name
对应的bean没有添加注解
对应bean添加注解错误，例如将spring的@Service错选成dubbo的包
选择错误的自动注入办法。

2、
org.springframework.web.util.NestedServletException:
Request processing failed; nested exception is java.lang.NullPointerException
错误出现的原因：处理该请求的方法所需要的部分参数在请求时没有值，导致空指针错误；
解决方法一：在第一次访问该页的请求，或者说访问出错的页请求链接上加上处理该请求的方法所需的参数；
解决方法二：配置两个处理请求的方法，一个不需要参数（用于第一次访问），一个不需要参数，用于处理有参数的请求。

3、
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)问题
即在mybatis中dao接口与mapper配置文件在做映射绑定的时候出现问题
简单说，就是接口与xml要么是找不到，要么是找到了却匹配不到。

还是有很多学不好的，先简单记录吧