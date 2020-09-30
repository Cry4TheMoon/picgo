

# JVM探究

- 谈谈你对JVM的理解？

  - 百度百科
    - JVM是Java Virtual Machine（Java虚拟机）的缩写，JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。
    - 引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。

  - 自我理解
    - jvm将字节码文件编译成操作系统识别的指令来完成java代码和操作系统及硬件设备的交互，主要功能为加载.class字节码文件，管理和分配内存及垃圾回收

- java各个版本虚拟机和之前的变化更新

  - J2SE：默认虚拟机改为HotSpot VM，之前为Classic VM
  - 
  - 

- 什么是OOM，什么事栈溢出StackOverFlowErro？怎么分析？

- JVM常用的调优参数有哪些

- 内存快照如何抓取，怎么分析Dump文件？

- 谈谈JVM中类加载器的认识

- 什么是逃逸分析？

## JVM的位置

![image-20200928192354040](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928192354040.png)

## JVM的体系结构

![image-20200928135015483](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928135015483.png)

![image-20200928135046236](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928135046236.png)

调优调的是方法区和堆区，主要是堆区

## 类加载器

**作用**：加载Class文件

![image-20200928135117121](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928135117121.png)

1. 虚拟机自带的加载器

2. 启动类（根）加载器BootstrapClassLoader

3. 扩展类加载器ExtensionClassLoader

4. 应用程序加载器ApplicationClassLoader

   

## 双亲委派机制(安全)

![image-20200928140043241](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928140043241.png)

1. 类加载器收到类加载的请求
2. 将这个请求向上委托给父类加载器去完成，一直向上委托，直到启动类加载器
3. 启动类加载器检查是否能够加载这个类，能加载就结束，使用当前的加载器，否则，抛出异常，通知子加载器进行加载
4. 重复步骤3、

## 沙箱安全机制

### 什么是沙箱？

Java安全模型的核心就是Java沙箱（sandbox），什么是沙箱？沙箱是一个限制程序运行的环境。沙箱机制就是将 Java 代码限定在虚拟机(JVM)特定的运行范围中，并且**严格限制代码对本地系统资源访问**，通过这样的措施来保证对代码的有效隔离，防止对本地系统造成破坏。沙箱主要限制系统资源访问，那系统资源包括什么？——CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。

 所有的Java程序运行都可以指定沙箱，可以定制安全策略。

### 组成沙箱的基本组件：

- 字节码校验器（bytecode verifier）：确保Java类文件遵循Java语言规范。这样可以帮助Java程序实现内存保护。但并不是所有的类文件都会经过字节码校验，比如核心类。
- 类装载器（class loader）：其中类装载器在3个方面对Java沙箱起作用

1. 它防止恶意代码去干涉善意的代码；
2. 它守护了被信任的类库边界；
3. 它将代码归入保护域，确定了代码可以进行哪些操作。

 虚拟机为不同的类加载器载入的类提供不同的命名空间，命名空间由一系列唯一的名称组成，每一个被装载的类将有一个名字，这个命名空间是由Java虚拟机为每一个类装载器维护的，它们互相之间甚至不可见。

 类装载器采用的机制是双亲委派模式。1. 从最内层JVM自带类加载器开始加载，外层恶意同名类得不到加载从而无法使用；

由于严格通过包来区分了访问域，外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效。

- 存取控制器（access controller）：存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定，可以由用户指定。
- 安全管理器（security manager）：是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器优先级高。
- 安全软件包（security package）：java.security下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性，包括：
  1. 安全提供者
  2. 消息摘要
  3. 数字签名
  4. 加密
  5. 鉴别

## Native

凡事带了Native关键字的，说明方法java的作用范围达不到了，会去调用底层C语言的库

会进入本地方法栈

调用本地方法本地接口（JNI）

JNI作用：扩展Java的使用，融合不同的编程语言为Java所用，最初C,C++

Java诞生的时候C、C++横行，想要立足，必须要有调用C、C++的程序

它在内存区域中专门开辟了一块标记区域：Native Method Stack，登记native方法

在最终调用的时候，加载本地方法库中的方法通过JNI



可以实现Java程序驱动打印机，管理系统，掌握即可，在企业级应用中较为少见！

```
Class Student{
	private native void start();
}
```

**调用其他语言方法接口**：Socket..WebService..http

## PC寄存器（程序寄存器Program Counter Register）

每个线程都有一个程序计数器，是线程私有的，就是一个指针，指向方法中的方法字节码（用来存储指向一条指令的地址，也即将要执行的指令代码），在执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不计

## 方法区（Method Area）

方法区是被所有线程共享的，所有字段和方法字节码，以及一些特殊方法，如构造函数，接口代码也在此定义，简单说，所有定义的方法的信息都保存着爱该区域，此区域属于共享区间

**==静态变量、常量、类信息（构造方法、接口定义）、运行时的常量池存放在方法区中，但是实例变量存在堆内存中==**

主要存放：static,final, Class模板,常量池

## 栈

特点：FIFO(先进后出、后进先出)

栈内存，主管程序的运行，生命周期和线程同步

线程结束，栈内存也就释放，==对于栈来说，不存在垃圾回收问题==

一旦线程结束，栈就Over

栈主要存储：8大基本数据类型+对象引用+实例方法

栈的运行原理：栈帧

```
public class Test{
	public static void main(String[] args){
		new Test().test();
	}
	public static void test(){
		System.out.println("stack");
	}
}
```

![image-20200928152813078](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928152813078.png)

main()进栈，程序开始

main()调用test()，test()进栈

test()执行结束，出栈

main()执行结束，出栈，程序结束

![image-20200928152813078](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928152813078.png)



```
public class Test{
	public static void main(String[] args){
		new Test().test();
	}
	public static void test(){
		a();
	}
	public static void a(){
		test();
	}
}
```



![image-20200928153125787](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928153125787.png)

进入死循环，就会报StackOverflowError

栈+堆+方法区的交互关系

![image-20200928154805927](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928154805927.png)

## 三种JVM

- Sun公司 `HotSpot`

  ```
  java -version
  ```

  ![image-20200928155053252](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928155053252.png)

- BEA公司`JRockit`

- IBM公司`J9VM`

## 堆(Heap)

一个JVM只有一个堆内存，堆内存的大小是可以修改的

类加载器读取了类文件后，一般会把什么东西放在堆中？

​	类，方法，常量，变量，所有引用类型的真实对象

堆内存中还要细分三个区域：

- 新生区
- 养老区
- 永久区

![image-20200928161538434](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928161538434.png)

**GC垃圾回收，主要在伊甸园区和养老区**

**假设内存满了，OOM（OutOfMemoryError:Java heap sapce），堆内存不够**

在JDK8以后，永久存储区改了名字（元空间）

![image-20200928161515595](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928161515595.png)

### 新生区

- 类：诞生和成长的地方，甚至死亡
- 伊甸园：绝大多数对象都是在伊甸园区new出来的，很大的对象是在老年区new出来的
- 幸存者区(0,1)：

### 老年代

### 永久代

A third generation closely related to the tenured generation is the permanent generation which holds data needed by the virtual machine to describe objects that do not have an equivalence at the Java language level. For example objects describing classes and methods are stored in the permanent generation.

与持久代密切相关的第三代是永久代，它保存虚拟机所需的数据，以描述在Java语言级别不具有等价性的对象。例如，描述类和方法的对象存储在永久生成中。

这个区域常驻内存的，用来存放JDK自身懈怠的Class对象。interface元数据，存储的是Java运行时的一些环境或类信息，**这个区域不存在垃圾回收**，关闭VM虚拟机就会释放这个区域的内存

jdk1.6之前：永久代，常量池在方法区

jdk1.7        ：永久代慢慢退化，`去永久代`，常量池在堆中

jdk1.8之后：无永久代，常量池在元空间

 ![image-20200928164406020](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928164406020.png)

==元空间逻辑上存在，物理上不存在==

**OOM 处理办法**

- 能够看到代码第几行出错：内存快照分析工具，MAT，Jprofiler
- Dubg，一行行分析代码

**MAT，Jprofiler作用**

- 分析Dump内存文件，快速定位内存泄漏
- 获得堆中的数据
- 获得大的对象
- ...

```
public class Demo {
    byte[] arr = new byte[1*1024*1024];
    public static void main(String[] args){
        List<Demo> lists = new ArrayList<Demo>();
        int count = 0;
        try {
            while (true){
                lists.add(new Demo());
                count = count+1;
            }
        }catch (Exception e){

        }
    }
}

```

![image-20200928182549727](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928182549727.png)

![image-20200928182614882](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928182614882.png)

## 垃圾回收机制(GC)

作用范围：堆、方法区

分类：轻GC，重GC

GC算法：标记清除法、标记压缩(标记整理)、复制算法、引用计数器，怎么用？

轻GC和重GC分别在什么时候发生

### 定位垃圾

#### 引用计数法

给每一个对象赋予一个计数器，当GC执行时发现某个对象被引次数为0时，则回收该对象

![image-20200928183849302](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928183849302.png)

- 缺点：不能解决环形垃圾

#### 根查找算法

![image-20200929090845542](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929090845542.png)

### 清除垃圾

#### 复制算法

![image-20200928185010541](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928185010541.png)

调整参数设定进入老年代的时间

```
-XX: -XX:MaxTenuringThreshold=15
```

![image-20200928185808734](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928185808734.png)

![image-20200928185732606](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928185732606.png)

- 优点：没有内存的碎片

- 缺点：浪费了内存空间：多了一半的空间永远是空to。假设对象100%存活(极端情况)

- 复制算法最佳使用场景：对象存活度较低的情况下使用~比较适合新生区GC

#### 标记清除法

![image-20200928190434734](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928190434734.png)

![image-20200928190448559](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928190448559.png)

- 缺点：两次扫描严重浪费时间，会产生内存碎片

- 优点：不需要额外的空间

#### 标记压缩

![image-20200928190801683](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928190801683.png)

优点：没有内存碎片

缺点：增加了内存的移动成本

#### 标记清除压缩

先标记清除多次，再压缩

![image-20200928191041437](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200928191041437.png)

### 总结

内存效率：复制算法>标记清除算法>标记压缩算法

内存整齐度：复制算法=标记压缩算法>标记清除算法

内存利用率：标记压缩算法=标记清除算法>复制算法



### 难道没有最优算法吗？

答案：没有，没有虽好的算法，只有最合适的算法--->GC：==分代收集算法==

年轻代：复制算法

- 存活率低

老年代：标记清除+标记压缩混合实现

- 区域大，存活率高

### 垃圾回收器

![image-20200929094317385](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929094317385.png)

`CMS默认在新生代清除6次进入老年代，其他默认15次`

`轻GC：新生代空间耗尽时触发`

`重GC：在老年代无法继续分配空间时触发，新生代老年代同时回收`

`对象如何进入老年代:`

![image-20200929100536370](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929100536370.png)



**Serial**

- a stop-the-world,copying collector which uses a single GC thread

![image-20200929101421605](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929101421605.png)

缺点：串行，单线程，速度慢

**Parallel Scavenge**

![image-20200929101636806](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929101636806.png)

**ParNew**

- a stop-the-world,copying collector which uses multiple GC threads
- it differs from "Parallel Scavenge" in that it has enhancements that make it usable with CMS
- For example , "ParNew" does th synchronization needed so that it can run during the concurrent phases of CMS

![image-20200929101636806](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929101636806.png)

**CMS**

- concurrent mark sweep
- a mostly concurrent, low-pause collector
- 4 phases
  1. initial mark
  2. concurrent mark
  3. remark                                                                  
  4. concurrent sweep

![image-20200929103717596](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200929103717596.png)

## 什么是调优？

1. 根据需求进行JVM规划和预调优
2. 优化运行JVM运行环境（慢、卡顿）
3. 解决JVM运行过程中出现的各种问题（OOM）

`内存泄漏:内存一直被占用`

`内存溢出：内存不够`

```
java -Xms200m -XmX200m -XX:+PringtGC com.ysu.jvm.test
```

调优软件：

==arthas==可以使用终端来检测问题

==jprofiler==适合带图形界面，测试时使用

==jmap==可以看出哪个对象占用内存最多

```
jmap -histo 1196 | head -20
```

## JMM（Java Memory Model）

