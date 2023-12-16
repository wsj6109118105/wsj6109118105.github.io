---
title: JVM
date: 2021-07-06 22:45:56
tags: java
---
# jdk组成
1.JDK是Java开发工具包，JDK中包含JRE（在JDK的安装目录下有一个名为jre的目录，里面有两个文件夹bin和lib，在这里可以认为bin里的就是jvm，lib中则是jvm工作所需要的类库，而jvm和 lib合起来就称为jre）和一堆Java工具（javac/java/jdb等）和Java基础的类库（即Java API 包括rt.jar）。  
2.JRE(Java Runtime Environment):是运行基于Java语言编写的程序所不可缺少的运行环境。JRE中包含了Java virtual machine（JVM），runtime class libraries和Java application launcher，这些是运行Java程序的必要组件。  
3.JVM(Java virtual machine):java虚拟机，所有的程序被编译为.class的类文件，类文件在java虚拟机上运行。
# JVM的位置
JVM位于操作系统之上。
# JVM体系结构
1.类加载器(classloader)：用于装载.class文件.  
2.执行引擎：用于执行字节码文件或本地方法。  
3.运行时数据区：包括方法区、堆、Java 栈、PC 寄存器、本地方法栈。
![avatar](https://note.youdao.com/yws/api/personal/file/WEB5b6e8ca9716506beb24ffb1e6c051d6e?method=download&shareKey=fb46a4b00408e4c4ecae5896e27b4156)
## 类加载器
1.类的加载和卸载  
JVM将指定的class文件读取到内存里，并运行该class文件里的Java程序的过程，就称之为类的加载；反之，将某个class文件的运行时数据从JVM中移除的过程，就称之为类的卸载。  
2.类的生命周期  
Java类从被虚拟机加载开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）7个阶段；其中验证、准备和解析又统称为连接（Linking）阶段。  
![avatar](https://note.youdao.com/yws/api/personal/file/WEB83ae79633994737cc0f3dea5a8505876?method=download&shareKey=9dbd097ec4b26972c440b03c9de5c3d0)
3.初始化  
类在什么时候初始化？  
创建类的实例，new对象  
访问某个类或接口的静态变量，或者对该静态变量赋值  
调用类的静态方法  
反射（Class.forName(" ")）  
初始化一个类的子类（首先会初始化子类的父类）  
JVM启动时标明的启动类，即文件名和类名相同的那个类  
**注意**：对于一个final类型的静态变量，如果该变量的值在编译时就可以确定下来，那么这个变量相当于“宏变量”。Java编译器会在编译时直接把这个变量出现的地方替换成它的值，因此即使程序使用该静态变量，也不会导致该类的初始化。反之，如果final类型的静态Field的值不能在编译时确定下来，则必须等到运行时才可以确定该变量的值，如果通过该类来访问它的静态变量，则会导致该类被初始化。  
**初始化顺序**  
对static修饰的变量或语句块进行赋值，如果同时包含多个静态变量或代码块则按从上到下的顺序，如果初始化一个类时，其父类还未初始化，则先初始化父类。
4.类加载器
- 应用程序类加载器(Application classloader)
- 扩展类加载器(Extention classloader)
- 启动类加载器(Bootstrap classloader)
![avatar](https://note.youdao.com/yws/api/personal/file/WEB8b345ca740c878d43470cc04a50d9c63?method=download&shareKey=955547d92cbe1ebdcaa7c48fd0e21243)  
**启动类加载器**：这个加载器使用c/c++实现，嵌在JVM内部，用来加载Java核心类库，负责加载扩展类加载器和应用类加载器，并为其指定父类。  
**扩展类加载器**：由Java语言编写的，由sun.misc.Launcher$ExtClassLoader实现，派生于ClassLoader类。父类加载器为null,上层加载器为启动类加载器，负责加载JRE的扩展目录。从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK系统安装目录的jre/lib/ext子目录（扩展目录）下加载类库。如果用户创建的jar放在此目录下，也会自动由扩展类加载器加载。  
**应用程序加载器**：Java语言编写的，由sun.misc.Launcher$AppClassLoader实现，父类加载器为ExtClassLoader。派生与Classloader类，上层加载器为扩展类加载器，加载我们自定义的类。  
- 注意：Classloader为一个抽象类，其后所有类加载器都继自Classloader  
- 类加载机制  
1.全盘负责：当一个类加载器负责加载某个Class时，该Class所依赖和引用其他Class也将由该类加载器负责载入，除非显示使用另一个类加载器来载入。  
2.双亲委派：先让父类加载器试图加载该Class，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类。  
3.缓存机制：保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓冲区中。这就是为什么修改了Class后，必须重新启动JVM，程序所做的修改才会生效的原因。  
- 双亲委派工作原理  
当一个类加载器接收到一个类加载的请求，他不会去加载这个类，而是把任务交给自己的父类，如果父类还存在父类，则继续向上委托，依次类推，直到顶层启动类加载器。如果父类加载器可以完成，则成功返回，否则由子类来加载，如果都失败，则抛出ClassNotFoundException。  
![avatar](https://note.youdao.com/yws/api/personal/file/WEB4758b3f88a157cb5177ac8aeeb67aac3?method=download&shareKey=8e29e223dd2500fb6a75200df8655ae4)
- 双亲委派的优点  
1.安全，可避免用户自己编写的类动态替换Java的核心类，如java.lang.String。，java核心api中定义类型不会被随意替换  
2.避免全限定命名的类重复加载  

5.沙箱安全机制  
作用：防止恶意代码污染java源代码。
## Native关键字
- Native:凡是带有Native关键字的说明java作用范围达不到不到，回去调用底层c语言的库
- 进入本地方法栈：调用本地方法接口JNI，JNI是为了扩展java的使用，融合其他语言为java所用,最初是为了融合c和c++;
- 在内存区域中专门开辟了一块标记区域：Native Method Stack，登记native方法
- 在最终执行的时候，在执行引擎中通过JNI加载本地方法库中的方法
## 方法区
- 方法区是被所有线程共享，所有字段和方法字节码，以及一些特殊方法，如构造函数，接口代码也在此定义，简单的说，所有定义的方法的信息都保存在方法区，此区域属于共享区间；
- 静态变量、常量、类信息（构造方法、接口定义）、运行时的常量池存在方法区中；但，实例变量存在堆内存中，和方法区无关
- static、final、Class模板、常量池
![avatar](https://note.youdao.com/yws/api/personal/file/WEB9ab525129d2a33929833593131c6a2f3?method=download&shareKey=3577e6221f2cef83abe4511e0f3140bd)
## 栈
- 栈内存，主管程序的运行，生命周期和线程同步，线程结束，占内存随之释放，也就是说不存在垃圾回收问题，一旦线程结束，栈就over。  
- **栈内会存储：8大基本类型+对象的引用+实例的方法。**  
- 栈的原理：栈帧
![avatar](https://note.youdao.com/yws/api/personal/file/WEB371a8c096f226abc18a1c266d8868a3f?method=download&shareKey=cdc1875494eb7f3ae1da476e1dad6a75)
- 栈满了则会包错：StackOverFlowError
## 堆(heap)
- 一个jvm只有一个堆内存，堆的大小是可以调节的
- 类加载器读取了文件之后，会将类，方法，常量，变量，保存我们所有引用的真实对象存储在堆中。  

堆中还要细分为三个区域：
- 新生区(伊甸园区)
- 养老区
- 永久区
![avatar](https://note.youdao.com/yws/api/personal/file/WEB86714cedf4f5338fc06f953964ef51be?method=download&shareKey=c3d0d5c5a1b305481f098f0fc1b3ff22)
GC垃圾回收主要在新生区，和养老区  
**堆内存满了会报错：OOM(OutOfMemoryError)**  
在jdk8之后永久区改名为元空间
**新生区**：类诞生，成长，甚至死亡，其中所有的对象都是在伊甸园区new出来的  
**永久区**：是常驻内存的，用来存储jdk自带的class对象，Interface元数据存储的是java运行时的环境或类信息，这个区域不存在垃圾回收，关闭虚拟机就会释放这个区域
- jdk1.6之前，常量池在方法区
- jdk1.7去永久代，常量池在堆中
- jdk1.8之后，无永久代，常量池在元空间  

堆内存调优：
- Xms 是指设定程序启动时占用内存大小。
- Xmx 是指设定程序运行期间最大可占用的内存大小。
- Xss 是指设定每个线程的堆栈大小。
- -XX:+PrintGC  输出形式：[GC 118250K->113543K(130112K), 0.0094143 secs]  [Full GC 121376K->10414K(130112K), 0.0650971 secs]
- -XX:+PrintGCDetails 输出形式：[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs]                 [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]
## GC
轻GC，重GC
- 引用计数法
- 复制算法
- 标记压缩清除算法
