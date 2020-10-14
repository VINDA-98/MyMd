# **** 03JVM 

[TOC]



## 1、Java 是如何实现跨平台的？

​		**跨平台的是 Java 程序，而不是 JVM。JVM 是用 C/C++ 开发的，是编译后的机器码，不能跨平台，不同平台下需要安装不同版本的 JVM**

​		java源码编译后会生成.class文件，成为字节码文件。Java虚拟机（JVM）就是负责将字节码文件翻译成特定平台下得机器码然后运行，也就是说，只要在不同平台下得机器码然后运行。==只要在不同平台上安装对应的JVM，就可以运行字节码文件，也就是我们编写编译后的java文件。==



## 2、**什么是JVM？**

​	JVM：Java Virtual Machine java虚拟机，通过==模拟一个计算机来达到一个计算机所具备的计算功能==。JVM能够跨计算机体系结构来执行java字节码，主要是jvm屏蔽了各个计算机平台相关的软件或者硬件之间的差异。使得与平台相关的耦合统一由JVM提供者来实现

​	

​	JVM**由哪些部分组成**？

​	1、类加载器，在JVM启动或者类运行的时候将需要的Class文件加载到JVM中

​	2、执行引擎，执行引擎的任务是负责执行class文件中包含的字节码指令，，相当于实际计算机的cpu

​	3、内存区，将内存划分成诺干个区以模拟实际机器上的存储、记录和调度功能模块，实际机器上的各种功能的寄存器或者pc指针的记录器等。

​	4、本地方法调用，调用C或者C++实现的本次方法的代码返回结果

![image-20201001201942975](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201001201942975.png)



### **3、类加载器（class Loader）的了解** 

​		（1）类加载器负责把java类加载到java虚拟机中，java虚拟机使用java类的方式如下：java源程序（.java）经过java编译后转成.class文件

​		（2）类加载器负责读取字节码文件（.class），并转成java.lang.class类的一个实例对象，每个这样的实例用来表示一个Java类，通过此实例的newInstance（）方法就可以创建出该类的一个对象，实际情况还要考虑比如java字节码文件可能是动态生成的，也可能是网络下载的.



**java虚拟机如何判定两个java类是相同的？**

​	（1）先看类名是否完全相同，包的路径是否一致

​	（2）再看类的加载器是否一样，例如com.Student被编译后生成Student.class文件，如果这时候用ClassLoaderA和ClassLoaderB分别加载这个class文件，那么得到的java.lang.Class的实例对象就不是同一个，对于java虚拟机来说，当我们想用不同类的对象进行相互赋值，是会抛出运行时异常ClassCastException。

运行过程：

​	![image-20201001203816047](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201001203816047.png)

​	第一个阶段是找到 .class 文件并把这个文件包含的字节码加载到内存中

​	第二阶段又可以分为三个步骤，分别是字节码验证、Class 类数据结构分析及相应的内存分配和最后的符号表的链接

​	第三个阶段是类中静态属性和初始化赋值，以及静态块的执行等



## 4、双亲委派机制

​		其实就是一种类加载器的层次关系，双亲委派模型，当一个类加载器加载类的时候，会把这个类请求委派给父类加载器，每一层都是如此，一直递归到顶层 bootstrap ClassLoader.，当父加载器无法完成这个请求时，子类才会尝试去加载，这里的双亲其实就指的是父类，没有mother。

![image-20201001234143935](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201001234143935.png)



源码解析:

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            // 第一步，检查
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }


```

- 首先从底向上的检查类是否已经加载过，就是这行代码：`Class<?> c = findLoadedClass(name);`
- 如果都没有加载过的话，那么就自顶向下的尝试加载该类。

**为什么这样设计呢？**

解析：这是对于使用这种模型来组织累加器的好处

答：主要是为了安全性，避免用户自己编写的类动态替换 Java 的一些核心类，比如 String，同时也避免了重复加载，因为 JVM 中区分不同类，不仅仅是根据类名，相同的 class 文件被不同的 ClassLoader 加载就是不同的两个类，如果相互转型的话会抛java.lang.ClassCaseException.



## 5、JVM 内存管理

![image-20201002004111469](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002004111469.png)

1. 方法区（线程共享）：==各个线程共享的一个区域==，用于存储虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
   - 运行时常量池：是方法区的一部分，用于存放编译器生成的各种字面量和符号引用。
2. 堆内存（线程共享）：==所有线程共享的一块区域==，垃圾收集器管理的主要区域。目前主要的垃圾回收算法都是分代收集算法，所以 Java 堆中还可以细分为：==新生代和老年代==；再细致一点的有 Eden 空间、From Survivor 空间、To Survivor 空间等，默认情况下新生代按照8:1:1的比例来分配。根据 Java 虚拟机规范的规定，Java 堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘一样。
3. 程序计数器（线程不共享）： Java 线程私有，类似于操作系统里的 PC 计数器，它可以看做是当前线程所执行的字节码的行号指示器。如果线程正在执行的是一个 Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是 Native 方法，这个计数器值则为空（Undefined）。此内存区域是唯一一个在 Java 虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。
4. 虚拟机栈（栈内存，线程不共享）：Java线程私有，虚拟机展描述的是Java方法执行的内存模型：每个方法在执行的时候，都会创建一个栈帧用于==存储局部变量、操作数、动态链接、方法出口等信息====；每个方法调用==都意味着一个栈帧在虚拟机栈中入栈到出栈的过程；
5. 本地方法栈（线程不共享）：和Java虚拟机栈的作用类似，区别是该区域==为 JVM 提供使用 native 方法==的服务    //native ：本地的意思



Java 的内存模型：

![image-20201002005341000](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002005341000.png)



​		Java 内存模型规定了所有的变量都存储在主内存（Main Memory）中。每条线程还有自己的工作内存（Working Memory），线程的工作内存中保存了被该线程使用到的变量的主内存副本拷贝，线程对变量的所有操作（读取、赋值等）都必须在主内存中进行，而不能直接读写主内存中的变量。不同的线程之间也无法直接访问对方工作内存中的变量，线程间的变量值的传递均需要通过主内存来完成，线程、主内存、工作内存三者的关系如上图。



**两个线程之间是如何通信的呢？**

​		答：在**共享内存**的并发模型里，线程之间共享程序的公共状态，线程之间通过写-读内存中的公共状态来隐式进行通信，典型的共享内存通信方式就是通过共享对象进行通信。==核心：通过共享对象==

![image-20201002084212940](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002084212940.png)



其他知识：

**类似-Xms、-Xmn这些参数的含义**

堆内存分配：

1. JVM初始分配的内存由-Xms指定，默认是物理内存的1/64
2. JVM最大分配的内存由-Xmx指定，默认是物理内存的1/4
3. 默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制；空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制。
4. 因此服务器一般设置-Xms、-Xmx相等以避免在每次GC 后调整堆的大小。对象的堆内存由称为垃圾回收器的自动内存管理系统回收。

非堆内存分配：

1. JVM使用-XX:PermSize设置非堆内存初始值，默认是物理内存的1/64；
2. 由XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4。
3. -Xmn2G：设置年轻代大小为2G。
4. -XX:SurvivorRatio，设置年轻代中Eden区与Survivor区的比值。

### **（1）内存泄漏和内存溢出**

1. 内存溢出指的是内存不够用了。
2. 内存泄漏是指对象可达，但是没用了。即本该被GC回收的对象并没有被回收
3. 内存泄露是导致内存溢出的原因之一；内存泄露积累起来将导致内存溢出。

内存泄漏的原因分析：

1. 长生命周期的对象引用短生命周期的对象

2. 没有将无用对象置为null

   

### （2）java创建一个对象的过程

​	![image-20201002091251954](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002091251954.png)

​		**1、检测类是否被加载：**当虚拟机执行到new关键字的时候，回去方法区中的常量池找这个类的引用，因为==方法区存储虚拟机已经加载的类的信息==，如果能找到符号引用，说明此类已经被加载到方法去，如果没找到，那么就会使用类加载器执行类的加载过程，类加载完成后继续执行。

​		**2、为对象分配内存**：类加载完毕后，虚拟机开始为对象分配内存，大小基本确定，只需要在对上分配所需要的内存。

​	 **3、为分配的内存空间初始化零值：**

对象的内存分配完成后，还需要将对象的内存空间都初始化为零值，这样能保证对象即使没有赋初值，也可以直接使用。

​		**4.对对象进行其他设置：**

​		分配完内存空间，初始化零值之后，虚拟机还需要对对象进行其他必要的设置，设置的地方都在对象头中，包括这个对象所属的类，类的元数据信息，对象的hashcode，GC分代年龄等信息。

​	**5.执行 init 方法：**

​		执行完上面的步骤之后，在虚拟机里这个对象就算创建成功了，但是对于Java程序来说还需要执行init方法才算真正的创建完成，因为这个时候对象只是被初始化零值了，还没有真正的去根据程序中的代码分配初始值，调用了init方法之后，这个对象才真正能使用。

![image-20201002092652428](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002092652428.png)

## 6、GC相关

### （1）判断内存对象是否已经死去

​		1、引用计数：每个对象由一个引用计数属性，新增一个引用时技术加1，引用释放时计数减1，计数为0时可以回收。此方法简单，无法解决对象相互循环引用的问题。

​		2、可达性分析：从GCRoots开始向下搜索，搜索所走过的路称为引用链，当一个对象不可达的时候，说明此对象是不可用的。



### （2）垃圾回收算法

​	引用计数、标记-清除、复制算法、标记-整理、分代收集算法



### 	（3）GC什么时候开始？

​	答：GC经常发生的区域是堆区，堆区还可以细分为新生代、老年代，新生代还分为一个Eden区和两个Survivor区。

### 	（4）垃圾收集器？

​		如果说收集算法是内存回收的方法论，==垃圾收集器就是内存回收的具体实现==



## 7、其他 JVM 相关面试题整理

### （1）64 位 JVM 中，int 的长度是多数？

​	答：Java 中，int 类型变量的长度是一个固定值，与平台无关，都是 32 位或者 4 个字节。意思就是说，在 32 位 和 64 位 的Java 虚拟机中，int 类型的长度是相同的。int 32位或者4个字节。

### 	（2）怎么获取 Java 程序使用的内存？堆使用的百分比？

答：可以通过 java.lang.Runtime 类中与内存相关方法来获取剩余的内存，总内存及最大堆内存。通过这些方法你也可以获取到堆使用的百分比及堆内存的剩余空间。

​	Runtime.freeMemory() 方法返回剩余空间的字节数，		

​	Runtime.totalMemory() 方法总内存的字节数，

​	Runtime.maxMemory() 返回最大内存的字节数。

### （3）Java 中堆和栈有什么区别？

​	答：JVM堆和栈都属于不同的内存区域，使用目的也不同，栈常用于保存方法帧和局部变量，而对象总是在堆上分配的，==栈通常比堆小，也不会在多个线程之间共享，而堆被整个JVM的所有线程共享==

![image-20201002102209303](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002102209303.png)



## 8、版本新特性

#### 枚举

解析：关键字 enum

答：问题：对象的某个属性的值不能是任意的，必须为固定的一组取值其中的某一个；

```java
public class EnumTest {

       public static void main(String[] args) {
            WeekDay day = WeekDay.FRI;
            System.out.println(day);
             //结果：FRI
            System.out.println(day.name());
             //结果：FRI
            System.out.println(day.ordinal());
             //结果：5
            System.out.println(WeekDay. valueOf("SUN"));
             //结果：SUN
            System.out.println(WeekDay. values().length);
             //结果：7
      }
      
      public enum WeekDay{
             SUN,MON ,TUE,WED, THI,FRI ,SAT;
      }
}
```

**总结：** 枚举是一种特殊的类，其中的每个元素都是该类的一个实例对象，例如可以调用WeekDay.SUN.getClass().getName 和 WeekDay.class.getName()。

**注意：** 最后一个枚举元素后面可以加分号，也可以不加分号。

```java
public class EnumTest {

       public static void main(String[] args) {
            WeekDay day = WeekDay.FRI;
      }
      
      public enum WeekDay{
             SUN(1),MON (),TUE, WED,THI ,FRI,SAT;
            
             private WeekDay(){
                  System. out.println("first" );
            }
            
             private WeekDay(int value){
                  System. out.println("second" );
            }
             //结果：
             //second
             //first
             //first
             //first
             //first
             //first
             //first
      }
}
```



#### 自动拆装箱

答：在 Java 中数据类型分为两种：基本数据类型、引用数据类型(对象)

自动装箱：把基本类型变成包装器类型，本质是调用包装器类型的valueOf（）方法

**注意**：基本数据类型的数组与包装器类型数组不能互换

在 java程序中所有的数据都需要当做对象来处理，针对8种基本数据类型提供了包装类，如下：

------

int → Integer
byte → Byte
short → Short
long → Long
char → Character
double → Double
float → Float
boolean → Boolean



在 jdk 1.5 以前基本数据类型和包装类之间需要相互转换：

基本---引用 `Integer x = new Integer(x);`
引用---基本 `int num = x.intValue();`

1）`Integer x = 1; x = x + 1;` 经历了什么过程？装箱→拆箱→装箱
2）为了优化，虚拟机为包装类提供了缓冲池，**Integer池**的大小为 -128~127 一个字节的大小。**String池**：Java 为了优化字符串操作也提供了一个缓冲池；

![image-20201002103000553](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201002103000553.png)



#### 静态导入

答：**静态导入：**导入了类中的所有静态成员，简化静态成员的书写。
import语句可以导入一个类或某个包中的所有类
import static语句导入一个类中的某个静态方法或所有静态方法



#### 泛型 Generics

​		答：引用泛型之后，允许指定集合里元素的类型，免去了强制类型转换，并且能在编译时刻进行类型检查的好处。Parameterized Type作为参数和返回值，Generic是vararg、annotation、enumeration、collection的基石。

泛型可以带来如下的好处总结如下：

1. 类型安全：抛弃List、Map，使用List、Map给它们添加元素或者使用Iterator遍历时，编译期就可以给你检查出类型错误

2. 方法参数和返回值加上了Type: 抛弃List、Map，使用List、Map

3. 不需要类型转换：List list = new ArrayList();

4. 类型通配符“?”： 假设一个打印List中元素的方法printList,我们希望任何类型T的List都可以被打印

   

#### 注解（Annotations）

答：

​		注解(Annotation)是一种应用于类、方法、参数、变量、构造器及包声明中的特殊修饰符，它是一种由JSR-175标准选择用来描述元数据的一种工具。Java从Java5开始引入了注解。在注解出现之前，程序的元数据只是通过java注释和javadoc，但是注解提供的功能要远远超过这些。注解不仅包含了元数据，它还可以作用于程序运行过程中、注解解释器可以通过注解决定程序的执行顺序。



#### 新增Formatter格式化器(Formatter)

`Formatter` 类是Java5中新增的 `printf-style` 格式化字符串的解释器，它提供对布局和对齐的支持，提供了对数字，字符串和日期/时间数据的常用格式以及特定于语言环境的输出。常见的 Java 类型，如 `byte`，`java.math.BigDecimal` 和 `java.util.Calendar` 都支持。 通过 `java.util.Formattable` 接口提供了针对任意用户类型的有限格式定制。



#### Diamond Operator

JDK7

类型判断是一个人特殊的烦恼，入下面的代码：

```
Map<String,List<String>> anagrams = new HashMap<String,List<String>>();
```

通过类型推断后变成：

```
Map<String,List<String>> anagrams = new HashMap<>();
```



#### JDK8 接口默认方法和静态方法

​		Java 8用默认方法与静态方法这两个新概念来扩展接口的声明。与传统的接口又有些不一样，它允许在已有的接口中添加新方法，而同时又保持了与旧版本代码的兼容性。

#### 	Lambda 表达式

`Lambda` 表达式（也称为闭包）是整个Java 8发行版中最受期待的在Java语言层面上的改变，Lambda允许把函数作为一个方法的参数（即：**[行为参数化](https://www.jianshu.com/p/eb7721f32a5d)**，函数作为参数传递进方法中）。

一个 `Lambda` 可以由用逗号分隔的参数列表、`–>` 符号与函数体三部分表示。

首先看看在老版本的Java中是如何排列字符串的：



```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");  
Collections.sort(names, new Comparator<String>() {

    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }

});
```

lambda表达式：

```java
Collections.sort(names, (String a, String b) -> {  
    return b.compareTo(a);
});
```

但是实际上还可以写得更短：



```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));  
```

对于函数体只有一行代码的，你可以去掉大括号{}以及return关键字，但是你还可以写得更短点：



```java
Collections.sort(names, (a, b) -> b.compareTo(a));  
```

Java编译器可以自动推导出参数类型，所以你可以不用再写一次类型。

#### 函数式接口

`Lambda` 表达式是如何在 Java 的类型系统中表示的呢？每一个Lambda表达式都对应一个类型，通常是接口类型。而**函数式接口**是指仅仅只包含一个抽象方法的接口，每一个该类型的Lambda表达式都会被匹配到这个抽象方法。因为**默认方法**不算抽象方法，所以你也可以给你的函数式接口添加默认方法。

我们可以将Lambda表达式当作任意只包含一个抽象方法的接口类型，确保你的接口一定达到这个要求，你只需要给你的接口添加 `@FunctionalInterface` 注解，编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错的。

示例如下：



```
@FunctionalInterface
interface Converter<F, T> {  
    T convert(F from);
}

Converter<String, Integer> converter = (from) -> Integer.valueOf(from);  
Integer converted = converter.convert("123");  
System.out.println(converted); // 123  
```

> **注意：** 如果 `@FunctionalInterface` 如果没有指定，上面的代码也是对的。



#### Steam

Java8添加的 `Stream API(java.util.stream)` 把真正的函数式编程风格引入到Java中。这是目前为止对Java类库最好的补充，因为 `Stream API` 可以极大提供Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。使用 Steam 写出来的代码真的能让人兴奋，这里链出之前的一篇文章：[Java 8——函数式数据处理（流）](https://www.jianshu.com/p/6fab3047c7e7)



流可以是无限的、有状态的，可以是顺序的，也可以是并行的。在使用流的时候，你首先需要从一些来源中获取一个流，执行一个或者多个中间操作，然后执行一个最终操作。中间操作包括`filter`、`map`、`flatMap`、`peel`、`distinct`、`sorted`、`limit` 和 `substream`。终止操作包括 `forEach`、`toArray`、`reduce`、`collect`、`min`、`max`、`count`、`anyMatch`、`allMatch`、`noneMatch`、`findFirst` 和 `findAny`。 `java.util.stream.Collectors` 是一个非常有用的实用类。该类实现了很多归约操作，例如将流转换成集合和聚合元素



**一些重要方法说明：**

- `stream`: 返回数据流，集合作为其源
- `parallelStream`: 返回并行数据流， 集合作为其源
- `filter`: 方法用于过滤出满足条件的元素
- `map`: 方法用于映射每个元素对应的结果
- `forEach`: 方法遍历该流中的每个元素
- `limit`: 方法用于减少流的大小
- `sorted`: 方法用来对流中的元素进行排序
- `anyMatch`: 是否存在任意一个元素满足条件（返回布尔值）
- `allMatch`: 是否所有元素都满足条件（返回布尔值）
- `noneMatch`: 是否所有元素都不满足条件（返回布尔值）
- `collect`: 方法是终端操作，这是通常出现在管道传输操作结束标记流的结束

**一些使用示例：**

**2.1 Filter 过滤：**

```java
stringCollection  
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);
```



**2.2 Sort 排序：**

```java
stringCollection  
    .stream()
    .sorted()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);
```

**2.3 Map 映射：**



```java
stringCollection  
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);
```



#### HashMap的底层实现有变化

Java8中，HashMap内部实现又引入了红黑树（数组+链表+红黑树），使得HashMap的总体性能相较于Java7有比较明显的提升。



#### JVM内存管理方面，由元空间代替了永久代。

区别：

1. 元空间并不在虚拟机中，而是使用本地内存
2. 默认情况下，元空间的大小仅受本地内存限制
3. 也可以通过-XX：MetaspaceSize指定元空间大小

#### Base 64

在Java 8中，Base64编码已经成为Java类库的标准。它的使用十分简单，下面让我们看一个例子：



```
import java.nio.charset.StandardCharsets;  
import java.util.Base64;

public class Base64s {

    public static void main(String[] args) {
        final String text = "Base64 finally in Java 8!";

        final String encoded = Base64.getEncoder().encodeToString(text.getBytes(StandardCharsets.UTF_8));
        System.out.println(encoded);

        final String decoded = new String(Base64.getDecoder().decode(encoded), StandardCharsets.UTF_8);
        System.out.println(decoded);
    }

}
```

程序在控制台上输出了编码后的字符与解码后的字符：



```
QmFzZTY0IGZpbmFsbHkgaW4gSmF2YSA4IQ==  
Base64 finally in Java 8!  
```

Base64类同时还提供了对URL、MIME友好的编码器与解码器（`Base64.getUrlEncoder() / Base64.getUrlDecoder()`, `Base64.getMimeEncoder() / Base64.getMimeDecoder()`）。

#### Date/Time API

Java 8 在包java.time下包含了一组全新的时间日期API。新的日期API和开源的Joda-Time库差不多，但又不完全一样，下面的例子展示了这组新API里最重要的一些部分：

**1.Clock 时钟：**

`Clock`类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代`System.currentTimeMillis()`来获取当前的微秒数。某一个特定的时间点也可以使用`Instant`类来表示，`Instant`类也可以用来创建老的`java.util.Date`对象。代码如下:



```
Clock clock = Clock.systemDefaultZone();  
long millis = clock.millis();  
Instant instant = clock.instant();  
Date legacyDate = Date.from(instant);   // legacy java.util.Date  
```

**2.Timezones 时区：**

在新API中时区使用`ZoneId`来表示。时区可以很方便的使用静态方法`of`来获取到。时区定义了到UTS时间的时间差，在`Instant`时间点对象到本地日期对象之间转换的时候是极其重要的。代码如下:



```
System.out.println(ZoneId.getAvailableZoneIds());  
// prints all available timezone ids
ZoneId zone1 = ZoneId.of("Europe/Berlin");  
ZoneId zone2 = ZoneId.of("Brazil/East");  
System.out.println(zone1.getRules());  
System.out.println(zone2.getRules());  
// ZoneRules[currentStandardOffset=+01:00]
// ZoneRules[currentStandardOffset=-03:00]
```

**3.LocalTime 本地时间：**

`LocalTime`定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。之后比较时间并以小时和分钟为单位计算两个时间的时间差。代码如下:



```
LocalTime now1 = LocalTime.now(zone1);  
LocalTime now2 = LocalTime.now(zone2);  
System.out.println(now1.isBefore(now2));  // false  
long hoursBetween = ChronoUnit.HOURS.between(now1, now2);  
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);  
System.out.println(hoursBetween);       // -3  
System.out.println(minutesBetween);     // -239  
```

`LocalTime`提供了多种工厂方法来简化对象的创建，包括解析时间字符串。代码如下:



```
LocalTime late = LocalTime.of(23, 59, 59);  
System.out.println(late);       // 23:59:59  
DateTimeFormatter germanFormatter = DateTimeFormatter  
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);
LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);  
System.out.println(leetTime);   // 13:37  
```

**4.LocalDate 本地日期：**

LocalDate表示了一个确切的日期，比如2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。下面的例子展示了如何给Date对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。代码如下:



```
LocalDate today = LocalDate.now();  
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);  
LocalDate yesterday = tomorrow.minusDays(2);  
LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);  
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();

System.out.println(dayOfWeek);    // FRIDAY  
```

从字符串解析一个LocalDate类型和解析LocalTime一样简单。代码如下:



```
DateTimeFormatter germanFormatter = DateTimeFormatter  
        .ofLocalizedDate(FormatStyle.MEDIUM)
        .withLocale(Locale.GERMAN);
LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);  
System.out.println(xmas);   // 2014-12-24  
```

**5.LocalDateTime 本地日期时间：**

`LocalDateTime`同时表示了时间和日期，相当于前两节内容合并到一个对象上了。`LocalDateTime`和`LocalTime`还有`LocalDate`一样，都是不可变的。`LocalDateTime`提供了一些能访问具体字段的方法。代码如下:



```
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);  
DayOfWeek dayOfWeek = sylvester.getDayOfWeek();  
System.out.println(dayOfWeek);      // WEDNESDAY  
Month month = sylvester.getMonth();  
System.out.println(month);          // DECEMBER  
long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);  
System.out.println(minuteOfDay);    // 1439  
```

只要附加上时区信息，就可以将其转换为一个时间点`Instant`对象，`Instant`时间点对象可以很容易的转换为老式的`java.util.Date`。代码如下:



```
Instant instant = sylvester  
        .atZone(ZoneId.systemDefault())
        .toInstant();
Date legacyDate = Date.from(instant);  
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014  
```

格式化`LocalDateTime`和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式。代码如下:



```
DateTimeFormatter formatter =  
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");
LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);  
String string = formatter.format(parsed);  
System.out.println(string);     // Nov 03, 2014 - 07:13  
```

和`java.text.NumberFormat`不一样的是新版的`DateTimeFormatter`是不可变的，所以它是线程安全的。