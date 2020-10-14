# Linux 学习笔记（10.8）

## 1、学习路线

![image-20201008212733377](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201008212733377.png)





#创建一个名为file的文件，touch是一个命令

**touch file** 

#进入一个目录，cd是一个命令

**cd /etc/**

#查看当前所在目录

**pwd**



在命令行中获取帮助

**man man**

![image-20201008214601670](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201008214601670.png)



图形命令字符：

sudo apt-get update 

sudo apt-get install sysvbanner  安装相关插件

banner vinda  打印图形

printerbanner -w 50 A 更换字体





## 2、用户、权限管理

### 当前用户名

who am i

who mom likes 



### 创建用户

#添加名字为Vinda的用户

sudo adduser vinda

useradd 只创建用户，不创建用户密码和工作目录

#添加密码

sudo passwd shiyanlou

#切换用户

su -l vinda

#退出当前用户

ctrl + D



#查看用户组

groups vindagroups

#查看/etc/group

cat /etc/group | sort

#获得过滤条件后的信息

cat /etc/group | grep -E "vinda"

#给用户组获取root权限

groups vindagroups

sudo usermod -G vinda

groups vindagroups

#删除用户

sudo deluser vinda --remove-home



## 3、文件权限

#列出较长格式的文件

ls -l

![image-20201008215948372](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201008215948372.png)

#修改文件权限

![image-20201008220138748](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201008220138748.png)

​	==每个文件有三组固定的权限，分别对应拥有者，所属用户组，其他用户==

​	文件的读写执行对应字母 `rwx`，以二进制表示就是 `111`，用十进制表示就是 `7`，对进制转换不熟悉的同学可以看看 [进制转换](https://baike.baidu.com/item/进制转换/3117222)。例如我们刚刚新建的文件 iphone11 的权限是 `rw-rw-rw-`，换成对应的十进制表示就是 666，这就表示这个文件的拥有者，所属用户组和其他用户具有读写权限，不具有执行权限。



#修改命令

chmod 600 filename

ls -alh iphone11

#加减赋值操作

chmod go-rw iphone11

`g`、`o` 还有 `u` 分别表示 group（用户组）、others（其他用户） 和 user（用户），`+` 和 `-` 分别表示增加和去掉相应的权限。



## 4、文件基本操作

### fhs文件图

![](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/fhs.png)

基本切换

cd ..  

cd /

cd ~  #/home/用户名

绝对路径 cd /usr/local/bin 

相对路径 cd ../../usr/local/bin



### 创建目录

mkdir  dirname

#父级目录

mkdir -p father/son/grandson



### 复制文件

#将某文件复制到目标目录下

cp filename father/son/grandson

#将某目录复制到目标目录下

cp -r   dirname father/son/grandson  

![image-20201009001324052](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201009001324052.png)



### 删除文件

#删除一个文件

rm test

#强制删除一个目录

rm -f test

#删除目录

rm -r family

#强制删除权限不够的目录

rm -rf family



### 移动文件

mkdir Documents 

touch file1

#移动文件到Document

mv file1 Documents

#重命名

mv file1 file2



#批量重命名

`rename` 命令并不是内置命令，若提示无该命令可以使用 `sudo apt-get install rename` 命令自行安装。

cd /home/vinda/

touch file{1..5}.txt

```java
# 批量将这 5 个后缀为 .txt 的文本文件重命名为以 .c 为后缀的文件:
rename 's/\.txt/\.c/'  *.txt
# 批量将这 5 个文件，文件名和后缀改为大写:
rename 'y/a-z/A-Z/' *.c
```



### 查看文件

cd /home/shiyanlou 

#获得etc 下password 文件到当前目录下

cp /etc/passwd passwd 

cat passwd

显示行号

cat -n passwd

#nl命令，更加专业的打印

**nl -b a passwd**

-b : 指定添加行号的方式，主要有两种：   

 -b a:表示无论是否为空行，同样列出行号("cat -n"就是这种方式)   

 -b t:只列出非空行的编号并列出（默认为这种方式）

 -n : 设置行号的样式，主要有三种：    -n ln:在行号字段最左端显示   

 -n rn:在行号字段最右边显示，且不加 0   

 -n rz:在行号字段最右边显示，且加 0 -w : 行号字段占用的位数(默认为 6 位)



### 阅读文件

more 、less

#阅读password

more passwd

打开后默认只显示一屏内容，终端底部显示当前阅读的进度。可以使用 `Enter` 键向下滚动一行，使用 `Space` 键向下滚动一屏，按下 `h` 显示帮助，`q` 退出。



只看前面几行

tail /etc/passwd

只看一行

tail -n 1 /etc/passwd



### 查看文件类型

file /bin/ls

![image-20201009003041602](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201009003041602.png)





## 5、环境变量于文件查找

​		要解释环境变量，得先明白变量是什么，准确的说应该是 Shell 变量，所谓变量就是计算机中用于记录一个值（不一定是数值，也可以是字符或字符串）的符号，而这些符号将用于不同的运算处理中。通常变量与值是一对一的关系，可以通过表达式读取它的值并赋值给其它变量，也可以直接指定数值赋值给任意变量。为了便于运算和处理，大部分的编程语言会区分变量的类型，用于分别记录数值、字符或者字符串等等数据类型。Shell 中的变量也基本如此，有不同类型（但不用专门指定类型名），可以参与运算，有作用域限定。

### 	创建变量

​	 declare tmp; #创建

​	\# 正确的赋值 tmp=vinda # 错误的赋值 带空格 tmp = vinda

   echo $tmp #打印变量

| 命 令    | 说 明                                                        |
| -------- | ------------------------------------------------------------ |
| `set`    | 显示当前 Shell 所有变量，包括其内建环境变量（与 Shell 外观等相关），用户自定义变量及导出的环境变量。 |
| `env`    | 显示与当前用户相关的环境变量，还可以让命令在指定环境中运行。 |
| `export` | 显示从 Shell 中导出成环境变量的变量，也能通过它将自定义变量导出为环境变量。 |



### 命令的查找路径与顺序

查看 `PATH` 环境变量的内容：

```bash
echo $PATH
```

默认情况下你会看到如下输出：

```bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

创建一个 Shell 脚本文件，你可以使用 gedit，vim，sublime 等工具编辑。如果你是直接复制的话，建议使用 gedit 或者 sublime，否则可能导致代码缩进混乱。

```bash
cd /home/shiyanlou
touch hello_shell.sh
gedit hello_shell.sh
```

**注意不要省掉第一行，这不是注释，有用户反映有语法错误，就是因为没有了第一行。**

```bash
#!/bin/bash

for ((i=0; i<10; i++));do
    echo "hello shell"
done

exit 0
```

为文件添加可执行权限，否则执行会报错没有权限：

```bash
chmod 755 hello_shell.sh
```

执行脚本：

```bash
cd /home/shiyanlou
./hello_shell.sh
```

创建一个 C 语言 `hello world` 程序：

```bash
cd /home/shiyanlou
gedit hello_world.c
```

输入如下内容，同样不能省略第一行。

```c
#include <stdio.h>

int main(void)
{
    printf("hello world!\n");
    return 0;
}
```

保存后使用 gcc 生成可执行文件：

```bash
gcc -o hello_world hello_world.c
```

**gcc 生成二进制文件默认具有可执行权限，不需要修改。**

在 `/home/shiyanlou` 家目录创建一个 `mybin` 目录，并将上述 `hello_shell.sh` 和 `hello_world` 文件移动到其中：

```bash
cd /home/shiyanlou
mkdir mybin
mv hello_shell.sh hello_world mybin/
```

现在你可以在 `mybin` 目录中分别运行你刚刚创建的两个程序：

```bash
cd mybin
./hello_shell.sh
./hello_world
```

![此处输入图片的描述](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/document-uid735639labid60timestamp1532339433567.png)

​		回到上一级目录，也就是 `shiyanlou` 家目录，当再想运行那两个程序时，会发现提示命令找不到，除非加上命令的完整路径，但那样很不方便，如何做到像使用系统命令一样执行自己创建的脚本文件或者程序呢？那就要将命令所在路径添加到 `PATH` 环境变量了。





### 添加自定义路径到“path“环境变量

```bash
PATH=$PATH:/home/shiyanlou/mybin
```

**注意这里一定要使用绝对路径**

### 安装JDk

#rpm安装jdk到/usr/local/myapp/jdk

rpm -ivh --prefix=/usr/local/myapp/jdk jdk-8u

cd jdk

#获得当前目录，复制下来

pwd



#设置环境变量 

vim /etc/profile

#文件中添加

```java
export JAVA_HOME=/usr/local/myapp/jdk/jdk1.8.0_261-adm64
export PATH=$path:$JAVA_HOME/bin
```

#重新读取

source /user/etc/profile 

#查看java版本

java -version



### 安装Tomcat

#安装加包名，到当前目录下

tar -xzvf apache-tomcat...

#打开端口

firewall -cmd --zone=public --add-port=8080/tcp --permanent

systemctl status firewalld 查看防火墙状态

firewall-cmd --reload



