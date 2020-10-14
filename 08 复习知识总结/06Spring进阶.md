# 06Spring进阶

## 1、spring循环依赖

### 	spring当中的循环依赖怎么解决？

​		spring当中是默 认单例支持循环。怎么证明是默认支持的？

​		怎么关闭循环依赖？解决循环依赖的细节？

 

**（1）研究依赖注入功能：**

​	初始化的时候完成依赖注入

​		初始化bean-----也叫springbean的生命周期

​		到底在哪个步骤完成依赖注入？

| bean     | SpringBean                                   |
| -------- | -------------------------------------------- |
| java对象 | Spring管理的Bean对象，有一系列的完整生命周期 |
|          |                                              |
|          |                                              |



**（2）spring bean的产生过程 bean 是由什么产生而来的？**

Class --- BeanDefinitionReader 

类被类加载器加载后，拿到spring管理，生成BeanDefinitionReader 对象（不一定生成），该对象提供了spring里面用到的API

![image-20201013201550367](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201013201550367.png)

![image-20201013202003587](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201013202003587.png)



## 2、源码解析-循环依赖	

​		通过实现接口 BeanFactoryPostProcessor 来获取springbeanfactory里面的内容。

```java
@Component
public class VindaBeanFactoryPostPorcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory BeanFactory) throws BeansException {
        //使用子类对象，子类对象的方法更加丰富
        GenericBeanDefinition indexService =
                (GenericBeanDefinition) BeanFactory.getBeanDefinition("indexService");
        /**
         *     public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
         *         // beanDefinitionMap.get  就是获取map中的key
         *         BeanDefinition bd = (BeanDefinition)this.beanDefinitionMap.get(beanName);
         *         if (bd == null) {
         *             if (this.logger.isTraceEnabled()) {
         *                 this.logger.trace("No bean named '" + beanName + "' found in " + this);
         *             }
         *
         *             throw new NoSuchBeanDefinitionException(beanName);
         *         } else {
         *             return bd;
         *         }
         *     }
         */

        //修改bean的mapper映射设置
        indexService.setBeanClass(StudentService.class);

    }
}
```



```java
public class Test {
    public static void main(String[] args) {
        //初始化Spring容器
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(Application.class);
        //这时候使用IndexService的初始化，加载就是StudentService的内容
        System.out.println(ac.getBean(IndexService.class));
    }
}
```

​		在这里拿到的IndexService的Bean对象其实就是一个StduentService类，这样==没有====添加注解的StduentService就替换掉IndexService里面的类内容==

​		相当于Map集合里（“IndexService”，BeanClass）->（“StudentService”，BeanClass）；



### 1、getSingleton()方法



```java
@Nullable
public Object getSingleton(String beanName) {
    return this.getSingleton(beanName, true);
}

@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
        synchronized(this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }

    return singletonObject;
}
```



最难方法: 判断是否开始实例化或者是实例化过程。

```java
protected boolean isPrototypeCurrentlyInCreation(String beanName) {
    Object curVal = this.prototypesCurrentlyInCreation.get();
    return curVal != null && (curVal.equals(beanName) || curVal instanceof Set && ((Set)curVal).contains(beanName));
}
```