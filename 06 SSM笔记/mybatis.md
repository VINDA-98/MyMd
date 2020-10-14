



# **mybatis**

## 1、持久层框架 jdbc简化  轻量级框架 

ORM 持久层指的是：**将业务数据存储到磁盘或数据库，使得数据具备长期存储能力，只要磁盘不损坏，在系统宕机，断电情况下，重启系统仍然可以读取数据**

**javaee重量级框架** Ejb enterprise javabean



**JDBC**（JavaDataBase Connectivity）就是 Java 数据库连接, 说的直白点就是使用 **Java 语言操作数据库**

本来我们是通过控制台或客户端操作的数据库, JDBC 是用 Java 语言来发送 SQL 语句

​	

## 2、ORM框架  object relation mapper 对象关系映射

entity 实体类 ----》 ORM  框架  《 ---- 数据库表

对实体类操作   《 ----  》  对数据表操作

insertStudent （Student student） --->完成对数据库插入数据的操作

List<Student> seletAllStudents() --> 返回查询所有学生的一个集合

## 3、mybatis的特点

**1、对原生sql的支持,hibernate hql自定义语言**

​		优点：接近JDBC，比较灵活。

​		缺点：对SQL语句依赖程度很高；并且属于半自动，数据库移植比较麻烦，比如mysql数据库编程Oracle数据库，部分的sql语句需要调整。

**2、sql语句与代码分离，存放于xml配置文件中**

​		优点：便于维护管理，不用在java代码中找这些语句；

​		缺点： JDBC方式可以用用打断点的方式调试，但是Mybatis不能，需要通过log4j日志输出日志信息帮助调试，然后在配置文件中修改。

**3、用逻辑标签控制动态SQL的拼接4**

​		优点：用标签代替编写逻辑代码；

​		缺点：拼接复杂SQL语句时，没有代码灵活，拼写比较复杂。不要使用变通的手段来应对这种复杂的语句。

**4、查询的结果集与java对象自动映射**

​		优点：保证名称相同，配置好映射关系即可自动映射或者，不配置映射关系，通过配置列名=字段名也可完成自动映射。

​		缺点：对开发人员所写的SQL依赖很强



## 4、mybatis框架搭建

导入jar包，同时配合数据库驱动包

在maven在数据库中添加依赖

mybatis组成：

​	（1）mybatis.xml组成 设置mybatis基本引用信息

​				属性、jdbc数据库信息、类的别名、映射文件

​    （2）java接口 定义了对实体的操作（增删改查）

​	（3） 映射文件 -->与java接口对应，通过namespace 命名空间映射文件与java接口关联起来，<insert>都与SQL与之对应

​				==id要与接口中的方法名保持一致==，约定优于配置

   （4）通过mybatis提供的api来操作mybatis



## 5、mybatis.xml配置文件配置过程

1、确认文档定义

2、<configuration></configuration>标签内容中添加具体配置信息

3、添加数据库配置信息 ,

```xml
<properties>

​	数据库信息

</properties>
```

4、给实体类包添加别名

```xml
 <typeAliases>

​	<typeAliase type = "实体类" alise = "别名”>

</typeAliases> 
```

5、添加扫描的mapper文件包

<mapper>包路径</mapper>



## 6、mapper.xml映射配置文件

```xml
<mapper namespace = "包含对应接口的类">
	
    <select id =""  paramaterType = ""  result = "">  sql  </select>

</mapper>
```



## 7、通过junit来完成测试

```java
@Tets
public void test01(){
	1、加载mybatis的配置文件  使用流
    InputStream is = Resources.getResourceAsStream("mapper.xmls");
    2、通过配置文件创建工厂的构建器
    SqlSessionFactory build = new SqlSessionFactoryBuilder.build();
    3、通过工厂打开sqlSeesion
    SqlSeesion sqlSeesion = build.openSession(true);  //true自动提交注解
    4、通过sqlSeesion获取接口
    StudentDao studentdao = sqlSeesion.getMapper(StudenDao.class);
    
    studentdao.insert();
    studentdao.update();  ....
    
    空指针异常：找点前面的对象就完事了

}
```



## 8、多表查询

再建一个实体类

一对一：使用主外键绑定

```java
public class Student {
private Integer id;
private String name;
private Integer age;
private Group group;
    
```

```xml
<resultMap type="order" id="orderUserResultMap">
    <id property="id" column="id" />
    <result property="userId" column="user_id" />
    <result property="number" column="number" />
    <result property="createtime" column="createtime" />
    <result property="note" column="note" />

    <!-- association ：配置一对一属性 -->
    <!-- property:order里面的User属性名 -->
        <!-- javaType:属性类型 -->
    <association property="user" javaType="user">
        <!-- id:声明主键，表示user_id是关联查询对象的唯一标识-->
        <id property="id" column="user_id" />
        <result property="username" column="username" />
        <result property="address" column="address" />
    </association>

</resultMap>
```



一对多：使用关联表

```java
public class Student {
private Integer id;
private String name;
private Integer age;
private  List<Group> group;


```



```java
 <resultMap type = "Student" id ="groudStudent">
    <!-- column是数据库字段， property是我们实体类需要的 id是主键的意思 -->
 	 <id column="id" property="id"></id>
     <result column= "name" property="name" />
     <result column= "age" property="age" />
     <result column= "gid" property="group.gid" />
     <result column= "gname" property="group.gname" />
 </resultMap> //自定义返回结果集映射
     
     
 resultMap type="user" id="userOrderResultMap">
    <id property="id" column="id" />
    <result property="username" column="username" />
    <result property="birthday" column="birthday" />
    <result property="sex" column="sex" />
    <result property="address" column="address" />

    <!-- 配置一对多的关系
        property：填写pojo类中集合类类属性的名称
        javaType：填写集合类型的名称 
    -->
    <collection property="orders" javaType="list" ofType="order">
        <!-- 配置主键，是关联Order的唯一标识 -->
        <id property="id" column="oid" />
        <result property="number" column="number" />
        <result property="createtime" column="createtime" />
        <result property="note" column="note" />
    </collection>
</resultMap>
    

```



或者一个新的类 上面有需要的内容信息



![](D:\ADaDemo\06 mybatis笔记\多表查询.png)



注意：mybaits的值的注入机制

​	当实体类没有无参的构造方法时，值的注入是通过构造方法去注入

​	==实体类有无参的构造方法，值注入会根据set方法注入我们想要的参数内容==



<resultMap>

​	实体类和数据库的表，实体类的属性和数据表的字段一一对应。属性名和数据库名不一致时候，可以通过映射来关联。

</resultMap>  



![](D:\ADaDemo\06 mybatis笔记\resultmap.png)



实体类某个属性是别的实体类的属性

![](D:\ADaDemo\06 mybatis笔记\别的实体类.png)

查询时候只需要先设置group里面接口，返回gid和gname，

然后包装resultMap ，

==一对一用：association（关联）==

==一对多用：collection （集合）==

![](D:\ADaDemo\06 mybatis笔记\注入.png)



一对多

![](D:\ADaDemo\06 mybatis笔记\一对多.png)



group中文件就只有group，studentMapper就只有student表的查询

![](D:\ADaDemo\06 mybatis笔记\一对多查询.png)



不好之处：子查询，效率受到影响，解决办法，在mybatis的配置文件中添加懒加载，或者使用关联查询

![](D:\ADaDemo\06 mybatis笔记\懒加载.png)

关联查询： sql语句复杂， 不推荐

![](D:\ADaDemo\06 mybatis笔记\关联查询.png)





## 9、动态sql

​	基于OGNL表达式，动态sql主要作用就是用来拼接sql，根据给定的条件会产生不同的sql语句。

​	if：

```xml
<select id="” resultMap=“”>
    select *from student
     where 1 = 1
       <if test=" id != null && id != '' ">
          and id = #{id}
       </if>
       <if test=" name != null && name != '' ">
          and name = #{name}
       </if>                                   
      
</select>
```

![](D:\ADaDemo\06 mybatis笔记\动态sql插入.png)



​	where

![](D:\ADaDemo\06 mybatis笔记\where标签.png)

动态查询: 新建map集合,添加对应的键值对信息,执行dao层对应方法	



set

![](D:\ADaDemo\06 mybatis笔记\set动态标签.png)

set会在指定位置添加set

自动把最后一个逗号去掉

所有条件不满足,不添加set

![](D:\ADaDemo\06 mybatis笔记\批量插入.png)



使用?的是预处理语句

? 预编译处理,防止sql注入, 供值的占位符

## 10、mybatis缓存机制

## 11、mybaits分页

​	

​	逻辑分页:rowbounds 仍然会把数据库的数据全部查询出来,前端分页,在内存截取指定范围的数据

​	==使用:在接口中的select方法中添加参数(Rowbounds rb)==

在测试代码中: 起始页码\三条

==RowBounds rb = new RowBounds(0,3);==	

物理分页: 只会把指定范围的数据查询出来放入内存中.

## 12、补充内容

​	sql片段（片段）

![](D:\ADaDemo\06 mybatis笔记\sql片段.png)

​	![](D:\ADaDemo\06 mybatis笔记\sql片段引用.png)



自增字段处理（了解）

​	多参数（了解）