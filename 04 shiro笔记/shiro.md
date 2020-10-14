# 1、什么shiro

​		 shiro是一款主流的Java安全框架,不依赖任何容器，可以运行在Java SE和JavaEE项目中，它的主要作用是对访问系统的用户进行身份认证、授权、会话管理、加密等操作。

​	 shiro就是用来解决安全管理的系统化框架。



# 2、shiro的核心组件

用户 、角色、权限

![](D:\ADaDemo\04 shiro笔记\权限、角色、用户关系图.png)



1. UsernamePasswordToken，shiro用来封装用户登录信息，使用用户的登录信息来创建令牌token。
2. SecurityManager，shiro的核心部分，负责安全认证和授权
3. Suject，shiro的一个抽象概念，包含了用户信息
4. ==Realm，开发者自定义模块，根据项目需求验证授权逻辑，全部写在Realm中。（主要操作代码）==
5. AuthenticationInfo ，用户的角色信息集合，认证时候使用。
6. AuthorzationInfo，角色的权限信息集合，授权的时候使用。
7. DefaultWebSecurityManager，安全管理器，开发者自定义的Realm，需要注入到DefaultWebSecurityMannger进行管理才能生效
8. ShiroFilterFactoryBean，过滤器工厂，shiro的基于运行机制是开发者制定的规则，shiro去执行，具体的执行代码就是shiroFilterFactoryBean创建一个一个Filter对象来完成。

==给角色赋予权限，给用户赋予角色，当确认了用户的角色完成了认证，再去从角色的权限完成授权==



# 3、Token

​	1、Token的引入：Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生。

​	==2、Token的定义：Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。==

​	3、使用Token的目的：Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮。



​	shiro的运行流程图：用户先获得token，token经过subject验证用户信息，来到SecurityManager负责安全认证和授权，AuthenticationInfo 和 AuthorzationInfo 提供用户、角色、权限的集合，最后来到Realm开发者自定义模块，根据项目需求验证授权逻辑。



![](D:\ADaDemo\04 shiro笔记\shiro工作流程.png)



# 4、Springboot整合Shiro

## 1、shiro的登录认证

导入依赖：

```java
<!--        Mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>

<!--mybatis plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.3.1.tmp</version>
        </dependency>

<!--        thymeleaf and shiro-->
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>
```

导入数据库表，通常应该是三张表，==用户表、角色表、权限表==



## 2、新建自定义的Realm模块

 先写认证

```java
public class AccountRealm extends AuthorizingRealm {

    @Autowired
    IAccountService accountService;

    /**
     * 授权   --》 权限授权给对应角色
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取当前用户登录的信息
        Subject subject = SecurityUtils.getSubject();
        Account account = (Account) subject.getPrincipal();//得到用户信息，就是把doGetAuthenticationInfo方法的用户信息拿来

        //设置角色
        Set<String> roles = new HashSet<>();
        roles.add(account.getRole()); //获取实体类中角色字段内容，是否存在这个角色
        //SimpleAuthorizationInfo 是 AuthorizationInfo 子类， AuthorizationInfo是角色的权限信息集合
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo(roles);

        //设置权限
        info.addStringPermission(account.getPerms()); //添加实体类中权限字段内容

        return info;
    }



    /**
     * 认证   --> 用户 得到角色认证
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //获取token
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;

        QueryWrapper queryWrapper = new QueryWrapper();
        QueryWrapper wrapper = new QueryWrapper();
        Account account = (Account) wrapper.eq("username", token.getUsername());
        //token认证
        if(account != null){
            return new SimpleAuthenticationInfo(account,account.getPassword(),getName());
        }
       
        return null;
    }
}

```



## 3、编写ShiroConfig类

给shiro帮我们去安全验证：编写认证和授权规则：

认证过滤器：

==anon：无需认证==

==authc：必须认证==

==authcBasic：需要通过HTTPbasic认证==

==user：不一定认证，只需曾经被shiro记录即可，相当于记住我==



授权过滤器： 先认证，再授权

perms ： 必须拥有某个权限才能访问

role ： 必须拥有某个角色才能访问

rest：请求必须基于Restful风格， post 、 put、get、delete

ssl：必须是安全的url 请求，协议https

```java
@Configuration
public class ShiroConfig {

//    //第三步 将安全管理器放置到ShiroFilterFactoryBean过滤器
//    @Bean
//    public ShiroFilterFactoryBean shiroFilterFactoryBean(
//            @Qualifier("defaultWebSecurityManager") DefaultWebSecurityManager manager){
//        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
//        //权限设置
//        Map<String,String> map = new Hashtable<>();
//        map.put("/main","authc");               //登录main网页需要 必须认证 只有认证了才考虑权限和角色
//        map.put("/manage","perms[manage]");     //登录manage网页需要 manage角色权限
//        map.put("/adminstrator","roles[administrator]");  //登录adminstator网页需要管理员角色
//        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
//        //设置默认的登录界面,不然默认登录login.jsp
//        shiroFilterFactoryBean.setLoginUrl("/login");
//
//        //设置未授权页面
//        shiroFilterFactoryBean.setUnauthorizedUrl("/unauth");
//        return shiroFilterFactoryBean;
//    }
//
//
//    //第二步 将Reaml放入到 DefaultWebSecurityManager管理
//    @Bean
//    public DefaultWebSecurityManager defaultWebSecurityManager(@Qualifier("accountRealm") AccountRealm accountRealm){
//        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
//        manager.setRealm(accountRealm);
//        return manager;
//    }
//
//    //第一步 先获得自定以得Reaml，注入bean
//    @Bean
//    public AccountRealm accountRealm(){
//        return new AccountRealm();
//    }
//
//    @Bean
//    public ShiroDialect shiroDialect(){
//        return new ShiroDialect();
//    }


    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager){
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        factoryBean.setSecurityManager(securityManager);
        //权限设置
        Map<String,String> map = new Hashtable<>();
        map.put("/main","authc");
        map.put("/manage","perms[manage]");
        map.put("/administrator","roles[administrator]");
        factoryBean.setFilterChainDefinitionMap(map);
        //设置登录页面
        factoryBean.setLoginUrl("/login");
        //设置未授权页面
        factoryBean.setUnauthorizedUrl("/unauth");
        return factoryBean;
    }


    @Bean
    public DefaultWebSecurityManager securityManager(@Qualifier("accoutRealm") AccountRealm accoutRealm){
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(accoutRealm);
        return manager;
    }

    @Bean
    public AccountRealm accoutRealm(){
        return new AccountRealm();
    }

    @Bean
    public ShiroDialect shiroDialect(){
        return new ShiroDialect();
    }
}

```



## 4、创建3个页面

main.html、 manage.html. administator.html

访问权限如下:
1、必须登录才能访问main.html
2、当前用户必须拥有manage授权才能访问manage.html
3、当前用户必须拥有administator角色才能访问administator.html

Controller：

```java
/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author Vinda
 * @since 2020-09-15
 */
@Controller
public class AccountController {
    @GetMapping("/{url}")
    public String redirect(@PathVariable("url") String url){
        return url;
    }

    @PostMapping("/login")
    public String login(String username, String password, Model model){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(username,password);
        try {
            subject.login(token);
            Account account = (Account) subject.getPrincipal();
            subject.getSession().setAttribute("account",account);
            return "index";
        } catch (UnknownAccountException e) {
            e.printStackTrace();
            model.addAttribute("msg","用户名错误！");
            return "login";
        } catch (IncorrectCredentialsException e){
            model.addAttribute("msg","密码错误！");
            e.printStackTrace();
            return "login";
        }
    }

    @GetMapping("/unauth")
    @ResponseBody
    public String unauth(){
        return "未授权，无法访问！";
    }

    @GetMapping("/logout")
    public String logout(){
        Subject subject = SecurityUtils.getSubject();
        subject.logout();
        return "login";
    }
}
```





index.html： xmlns:shiro="http://www.thymeleaf.org/thymeleaf-extras-shiro">

<link rel="shortcut icon" href="#"/> 关闭报错

```html
<!DOCTYPE html>

<html lang="en" xmlns:th="http://www.thymeleaf.org" 
      xmlns:shiro="http://www.thymeleaf.org/thymeleaf-extras-shiro">

<head>
    <meta charset="UTF-8">
  	<title>Title</title>
  	<link rel="shortcut icon" href="#"/>
</head>

<body>

  <h1>index</h1>

    <div th:if="${session.account != null}">
        
​    <span th:text="${session.account.username}+'欢迎回来！'"></span><a href="/logout">退出</a>

  </div>

    <a href="/main">main</a> <br/>

    <div shiro:hasPermission="manage">

        <a href="manage">manage</a> <br/>

  </div>

    <div shiro:hasRole="administrator">

        <a href="/administrator">administrator</a>

  </div>

</body>

</html>
```





application.yml文件：

```java
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: weida
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=UTC
  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath*:/mapper/**Mapper.xml
```

