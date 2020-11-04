---
title: JAVA虚拟机基础知识
date: 2020-10-26 15:59:43
categories:
	- 必备知识
tags:
	- 编程
	- JAVA
---

# 1 JVM概括

## 1.1 JVM 介绍

`JVM`是一种用于计算设备的**规范**，它是一个**虚构**的计算机的软件实现，简单的说，`JVM`是运行`byte code`字节码程序的一个容器。 

### 1.1.1 特点

- **基于堆栈的虚拟机**：最流行的计算机体系结构，如英特尔 `X86` 架构和 `ARM` 架构上运行基于 **寄存器**。比如，安卓的 `Davilk` 虚拟机就是基于 **寄存器** 结构，而 `JVM` 是基于栈结构的。

- **符号引用** ：除了**基本类型**以外的数据 **（类和接口）** 都是通过**符号**来引用，而不是通过显式地使用**内存地址**来引用。

- **垃圾收集** ：一个类的实例是由用户程序**创建**和垃圾回收**自动销毁**。

- **网络字节顺序** ：`Java class`文件用网络字节码顺序来进行存储，保证了小端的`Intel x86`架构和大端的`RISC`系列的架构之间的无关性。

### 1.1.2 JVM 字节码

`JVM`使用Java字节码的方式，作为`Java` **用户语言** 和 **机器语言** 之间的中间语言。实现一个**通用的**、 **机器无关** 的执行平台。

## 1.2 JVM架构和组件

### 1.2.1 JVM 架构

Java虚拟机主要包括类加载器、运行时数据区、执行引擎JIN。

HotSpot JVM架构：支持强大特性和能力的基础平台，支持实现高性能和强大的可伸缩性的能力。举个例子，Hotspot 虚拟机 JIT 编译器生成动态的优化，换句话说，它们在 Java 应用执行期做出优化，为底层系统架构生成高性能的本地机器指令。另外，经过它的运行时环境和多线程垃圾回收成熟的进化和连续的设计， Hotspot 虚拟机在高可用计算系统上产出了高伸缩性。

![jvm-1](/images/jvm01.png)

JVM有两个性能指标：

- 停顿时间 - 响应延迟是指一个应用回应一个请求的速度，包括桌面UI响应事件响应事件、网站返回网页速度、数据查询返回速度
- 吞吐量 - 特定时间内一个应用工作量最大值，包括给定时间内完成的任务数量、一小时内批处理程序完成工作量|数据查询完成数量

### 1.2.2 HotSpot对象

- jvm遇到new指令

  检查方法区是否有class对象信息

  该信息是否加载好了，是否验证、准备、解析了

  如果都准备了，就直接根据class信息在堆区实例化，把引用存储到线程栈中使用

  如果没有，就执行类加载过程   

- 对新生对象内存分配

  1. 内存绝对规整：Bump the pointer 指针碰撞，仅仅把内存分界指针挪动一段与对象大小相同的距离，分配空间
     2. 内存不规整：Free List，通过空闲列表找到足够大的内存空间
             	3. 将内存空间初始化为0
           	4. JVM对对象设置
                  有哪些实例
                  元数据信息
                  对象hashcode
                  对象GC分代年龄等
     3. GC收集器的内存分配算法
           Serial、ParNew（带Compact过程的收集器）-》指针碰撞
           CMS（基于Mark-Sweep）-》Free List

- 对象内存分布

  - 对象头（对象自身的运行时数据（Mark Word））：哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳。该部分数据被设计成1个 非固定的数据结构 以便在极小的空间存储尽量多的信息（会根据对象状态复用存储空间）
  - 对象类型指针：对象指向它的类元数据的指针,class对象。虚拟机通过这个指针来确定这个对象是哪个类的实例。
  - 实例数据：对象真正有效的信息，即代码中定义的字段内容。
  - 对齐填充：占位符占位作用，对象的大小必须是8字节的整数倍。

### 1.2.3 对象头

- 布局
- GC状态（00轻量级锁、01无锁或偏向锁、10重量级锁、11gc待回收）
- 类型（0表示非自旋锁、1表示自旋锁）
- 同步状态（）
- (identity) hash code（JVM直接计算的）
- 数组长度 (*前提得是数组*)

#### 1.2.3.1 klass pointer

klass pointer的存储内容是一个指针，指向了其类元数据的信息，jvm使用该指针来确定此对象是类的哪个实例。

如果你有一个Person实例的引用，那么找到元数据就靠它了。

实例对象数据和类信息数据

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-18.png)

#### 1.2.3.2 Mark Word

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-19.png)

内具体容包括：

- unused：未使用的
- hashcode：上文提到的**identity** hash code，本文出现的hashcode都是指identity hash code
- thread: 偏向锁记录的线程标识
- epoch: 验证偏向锁有效性的时间戳
- age：分代年龄
- biased_lock 偏向锁标志
- lock 锁标志
- pointer_to_lock_record 轻量锁lock record指针
- pointer_to_heavyweight_monitor 重量锁monitor指针

`cms_free`从名字就能看出和cms收集器有关系，因为cms算法是**标记-清理**的一款收集器，所以内存碎片问题是将不可达对象维护在一个列表**free list**中，笔者推测此处应该是标记对象是否在**free list**中。

调用过hashcode方法(或者隐式调用：存到hashset里，map的key，调用了默认未经重写的toString()方法)，把“坑位”占了，偏向锁想存的线程id没地方存了，自然就直接是**轻量级锁**了。即本来是无锁状态可以变为偏向锁状态的，结果有hashcode存在，没有空间存线程ID，只能直接轻量级。

#### 1.2.3.3 实验

使用OpenJDK提供的 jol 工具：

```java
public static void main(String[] args) {        
    // 声明一枚长度为3306的数组        
    int[] intArr = new int[3306];        
    // 使用jol的ClassLayout工具分析对象布局        		System.out.println(ClassLayout.parseInstance(intArr).toPrintable()); 
}
```

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-20.png)

对象头的三个部分，分别印证了上文提到的klass pointer和Mark Word，以及数组独有的长度属性。

| 偏向锁标志 | 0        | 1      | -        | -        | -    |
| ---------- | -------- | ------ | -------- | -------- | ---- |
| 锁标志     | 01       | 01     | 00       | 10       | 11   |
| 状态       | 无锁状态 | 偏向锁 | 轻量级锁 | 重量级锁 | GC   |

## 1.3 JVM内存区域  

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-2.png)

- 程序计数器PC ：当前线程执行字节码的行号指示器。

对于线程，用于切换时对运行现场的恢复，每个线程PC都是私有的，互不影响，单独存储。如果线程正在执行的是一个普通方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是 Native 方法，这个计数器值则为空（Undefined）。此内存区域是唯一一个在 JVM 中没有规定任何 OutOfMemoryError 情况的区域。

- JVM 栈：线程私有，其生命周期和线程相同。创建栈帧，栈帧栈帧存储了局部变量表、操作数栈、常量池引用。StackOverflowError，OutOfMemoryError

- 本地方法栈：虚拟机栈为 Java 方法服务；本地方法栈为 Native 方法服务。本地方法栈也会抛。StackOverflowError 异常和 OutOfMemoryError 异常

- Java 堆：存放对象实例，几乎所有的对象实例都是在这里分配内存。Java 堆是垃圾收集的主要区域（因此也被叫做 "GC 堆"）。现代的垃圾收集器基本都是采用分代收集算法，该算法的思想是针对不同的对象采取不同的垃圾回收算法。

  新生代（Young Generation）Eden、Survivor#1、Survivor#2。老年代（Old Generation）

- 方法区：用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现。

- 运行时常量池：Class 文件中除了有类的版本、字段、方法、接口等描述信息，还有一项信息是常量池（Constant Pool Table），用于存放编译器生成的各种字面量和符号引用，这部分内容会在类加载后被放入这个区域。

- 直接内存：

并不是虚拟机运行时数据区的一部分，也不是 JVM 规范中定义的内存区域。NIO 类可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆里的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆中来回复制数据。

- 线程私有
  线程栈
  		本地方法栈
  		程序计数器
- 线程公有
  方法区
  		堆区
  		运行时常量区

## 1.4 JVM字节码执行引擎

输入字节码，过程字节码解析，输出执行结果。

### 1.4.1 动态链接

每个栈帧都包含一个指向运行时常量池[6]中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接。通过第6章的讲解，我们知道Class文件的常量池中存有大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用为参数。这些符号引用一部分会在类加载阶段或第一次使用的时候转化为直接引用，这种转化称为静态解析。另外一部分将在每一次的运行期间转化为直接引用，这部分称为动态连接。

### 1.4.2 方法调用

Class文件中一般存储的是符号引用，只有到运行时是才会解析为直接引用，但静态方法和私有方法会在连接环节中的解析是就为直接引用，因为静态方法属于类，私有方法外部无法访问，不存在其他对象引用

方法调用字节码指令：四个

- invokestatic     静态方法
- invokespecial    构造方法私有方法父类方法
- invokevirtual    所有虚方法
- invokeinterface  接口方法

### 1.4.3 分派

Java多态特性在JVM中是如何处理的？如何找到正确的方法执行？（重载和覆盖重写）

静态分派：静态指类型是是表面类型左值，根据类型找方法的对应参数是重载，所以静态分派是重载的原理，而且静态分派是多方的，可以在编译时找到多个方法然后建立直接引用。

动态分派：动态指右值实例类型，根据实例类型找对应的方法是多态，是重写的原理，动态分派在运行时建立单方的一个直接引用。

静态分派符合转型策略，如char->int->long->float->double这是基本类型，如果都没有就找引用类型Character，再没有依据父类双亲委派原则，会找Character继承的类或实现的接口等等。

```java
Father fs = new Son();
```

类型是 fs， 但实例化用的是 Son的类信息，所以堆分配了Son的空间。当调用方法时，先在堆内Son空间找，如果找到就调用（重写）；如果没有，就向上找Son的父类；如果父类们也没有，就报错 `java.lang.AbstractMethodError`

那么为什么 fs 调用的属性就是 Father的，而不是Son的呢？这就牵扯到对象的访问定位。

线程栈中指针要访问堆空间，中间有个句柄池，这个句柄池存储了实例对象的指针和对象类型指针。所以当访问fs的属性是就调用其类型指针，当访问fs的方法时就调用其实例指针。

### 1.4.4 对象的访问定位

- 句柄：
  线程栈中引用访问句柄池
  		句柄池存储对象实例数据指针和对象类型数据指针
  		修改少，移动对象只需要移动句柄，适合频繁移动对象地址
- 直接指针：
  引用直接指向对象实例数据，实例数据中包含对象类型指针
  		访问速度快，适合频繁访问对象

## 1.5 线程安全实现方法

### 1.5.1 线程安全

互斥同步：互斥→同步，只有互斥了才能保证同步，即多线程下共享数据在同一时刻只能被同一线程修改访问。互斥方法有临界区，互斥量，信号量

最基本的互斥同步手段就是synchronized关键字，synchronized关键字经过编译之后，会在同步块的前后分别形成monitorenter和monitorexit这两个字节码指令，这两个字节码都需要一个reference类型的参数来指明要锁定和解锁的对象。

如果Java程序中的synchronized明确指定了对象参数，那就是这个对象的reference；如果没有明确指定，那就根据synchronized修饰的是实例方法还是类方法，去取对应的对象实例或Class对象来作为锁对象。

在执行monitorenter指令时，首先要去尝试获取对象的锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象的锁，把锁的计数器加1，相应地，在执行monitorexit指令时会将锁计数器减1，当计数器为0时，锁就被释放了。如果获取对象锁失败了，那当前线程就要阻塞等待，直到对象锁被另外一个线程释放为止。

首先，synchronized同步块对同一条线程来说是可重入的，不会出现自己把自己锁死的问题。其次，同步块在已进入的线程执行完之前，会阻塞后面其他线程的进入。上一章讲过，Java的线程是映射到操作系统的原生线程之上的，如果要阻塞或唤醒一条线程，都需要操作系统来帮忙完成，这就需要从用户态转换到核心态中，因此状态转换需要耗费很多的处理器时间。对于代码简单的同步块（如被synchronized修饰的getter()或setter()方法），状态转换消耗的时间可能比用户代码执行的时间还要长。所以synchronized是Java语言中一个重量级（Heavyweight）的操作，有经验的程序员都会在确实必要的情况下才使用这种操作。而虚拟机本身也会进行一些优化，譬如在通知操作系统阻塞线程之前加入一段自旋等待过程，避免频繁地切入到核心态之中。

还可以使用JUC中的重入锁（ReentrantLock）实现同步，都有可重入性，重入表示线程执行时，其他线程可以进入不是完全禁止的。语法层面上就是经常写的lock|unlock方法。比Synchronized有三个优点：

- 等待可中断：等待获得锁久了可以选择放弃等待
- 可实现公平锁：公平指时间公平先等待的先用，等待最久的先用
- 锁可绑定多个条件：多个condition，Synchronized只能有一个

### 1.5.2 非阻塞同步

上面介绍的是阻塞同步，会在等待和唤醒上有性能问题。就是在线程等到锁获得对象锁时，CPU是阻塞的，线程没法干其他事，只能干等着。同时阻塞同步也属于悲观锁策略，认为什么都不做一定不安全。

非阻塞就是发出需要锁通知，然后干其他事，等有锁了再来拿。

基于冲突检测的乐观并发策略，通俗地说就是先进行操作，如果没有其他线程争用共享数据，那操作就成功了；如果共享数据有争用，产生了冲突，那就再进行其他的补偿措施（最常见的补偿措施就是不断地重试，直到试成功为止），这种乐观的并发策略的许多实现都不需要把线程挂起，因此这种同步操作被称为非阻塞同步（Non-BlockingSynchronization）。

CAS指令需要有三个操作数，分别是内存位置（在Java中可以简单理解为变量的内存地址，用V表示）、旧的预期值（用A表示）和新值（用B表示）。CAS指令执行时，当且仅当V符合旧预期值A时，处理器用新值B更新V的值，否则它就不执行更新，但是不管是否更新了V的值，都会返回V的旧值，上述的处理过程是一个原子操作。

### 1.5.3 非同步ThreadLocal

保证线程安全，不一定非要线程同步：2种方式

- 可重入代码：变量均在私有区，无堆和方法区变量，可以通过参数传递过来，不调用非可重入方法。
- 线程本地存储TLS：共享数据放在私有区域当然没有问题
- 使用volatile修饰变量：强制从缓存写入主存

### 1.5.4 锁优化

高并发锁优化：适应性自旋、锁消除、锁粗化、轻量级锁、偏向锁

- 自旋锁：一直等待获得锁，但不是阻塞等待，而是运行等待，但有等待时间限制，自适应自旋锁没有时间限制
- 锁消除：JVM发现用不到锁
- 锁粗化：原来是对一条语句加锁，现在是对整个方法加锁

偏向锁：消除在无竞争下同步原语。这个锁会偏向于第一个获得它的线程，如果在接下来的执行过程中，该锁没有被其他的线程获取，则持有偏向锁的线程将永远不需要再进行同步。

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-3.png)

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-4.png)

## 1.6 类加载器

类加载分为7个部分：加载→验证→准备→解析→初始化→使用→卸载。

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-5.png)

\#加载：做三件事：

- 通过类全限定名获得该类的二进制字节流

- 类的静态存储结构转为方法区的运行时数据结构

- 堆内生命一个对象实例的引用java.lang.Class指向方法区的class结构

  （看上一篇的示意图，内存分配的时候，堆内对象实例含有一个指向方方法区的引用）

注意第一步字节流的获取可以有多种方式，默认是.class文件，还可以从压缩包中如jar、网络流applet、运行时计算如动态代理java. lang. reflect. Proxy类、其他文件生成jsp、数据库获取。

只要把字节流给类加载器就行，无论从什么地方得到字节流。



\#验证：四类验证

上面得到字节流很随意，这步验证要保证安全，符合JVM要求才行

- 文件格式验证，保证字节流格式可以存储在方法区中，并完成存储

- 元数据验证，对字节码描述的信息进行语义分析，保证符合java语法规范，如父类接口实现抽象类不可继承等

- 字节码验证，数据流控制流|方法体分析，如方法跳转返回不会到指令外

- 符号引用验证，引用匹配性验证，保证能访问到

  

\#准备：赋默认初始值三种情况

经过安全验证后，在方法区正式为类分配内存并赋默认初始值。

- 静态类变量static，直接赋值默认值
- 非静态实例变量，保持不动
- 最终变量final，编译为ConstantValue，直接赋值设定的值

非静态实例变量不会操作，后者会在实例化时分配到堆中。

```java
class Test{
  public static void i=10;
  public void j=20;
  public static final k=30;
}
//准备后，方法区内：i=0 k=30
```

\#解析：将常量池的符号引用替换为直接引用

- 符号引用：字面值，相对于程序的自定义引用，类似C使用相对地址
- 直接引用：根据当前内存设定的指针或偏移量，相对虚拟机而言

\##类或接口解析：

在类D中，要把符号引用N转为类C的直接引用，就把N的全限定名给D去加载类C到方法区，然后直接引用。

\##字段解析：

即变量包括静态非静态的，因为父子类变量相同问题，需要建立连接确定是父类的字段，还是父类的父类的字段

\##类方法解析：

\##接口方法解析：

所谓解析就是找的意思，按照继承关系往上找，直到找到返回直接引用.

![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-6.png)

### 1.6.2 双亲委派原则

加载的第一步：通过类全限定名获得该类的二进制字节流，这件事放在JVM外部去做了，做这件事的叫类加载器。

两种不同加载器：

- BootstrapClassLoader加载器，使用CPP编写用于加载JVM核心rt.jar和java_home\lib文件，JVM的一部分

- 其他加载器，使用Java语言继承ClassLoader抽象类实现，包括ExtensionClassLoader 加载java_home|lib|ext或者被系统变量指定的目录，和 ApplicationClassLoader 加载classpath指定的类库。

  双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

使用双亲委派模型来组织类加载器之间的关系，有一个显而易见的好处就是Java类随着它的类加载器一起具备了一种带有优先级的层次关系。例如类java.lang.Object，它存放在rt.jar之中，无论哪一个类加载器要加载这个类，最终都是委派给启动类加载器进行加载，因此Object类在程序的各种类加载器环境中都是同一个类。相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己写了一个名为java.lang.Object的类，并放在程序的ClassPath中，那系统中将会出现多个不同的Object类，Java类型体系中最基础的行为也就无从保证，应用程序也将会变得一片混乱。

### 1.6.3 类加载器关系

1. `Bootstrap Classloader` 是在`Java`虚拟机启动后初始化的。
2. `Bootstrap Classloader` 负责加载 `ExtClassLoader`，并且将 `ExtClassLoader`的父加载器设置为 `Bootstrap Classloader`
3. `Bootstrap Classloader` 加载完 `ExtClassLoader` 后，就会加载 `AppClassLoader`，并且将 `AppClassLoader` 的父加载器指定为 `ExtClassLoader`。
   ![img](https://raw.githubusercontent.com/hujunchina/java-interview/master/img/jvm-13.png)

# 2 GC 介绍

## 2.1 JVM 内存划分

一、JVM运行数据区域（内存划分）

1.  程序计数器：一个寄存器空间，存储线程执行当前指令的地址。
2.  JVM栈：一块内存区域，方法一定是对方法而言，是方法执行的内存模型，方法从执行到结束表示创建栈帧入栈到出栈的过程。
3.  栈帧：包括局部变量，操作栈，动态链接，方法出口等。
4.  本地方法栈：是native方法的内存模型。
5.  JVM堆：一大块内存区域且线程共享，存储对象实例和数组，GC活动主要区域。由于GC采用分代收集算法，从内存回收角度细分，可以再分为新生代和老年代，新生代再分为Eden区Survivor#1区Survivor#2。
6.  方法区：一块内存区域且线程共享，存储类加载信息|常量|静态变量|编译后的代码，GC在此区域做常量池回收和类型卸载。
7.  运行时常量池：存放编译期生成的字面值String和符号引用，使用String的intern方法可以检测有没有存储这个字面值，有就直接返回，没有就在常量池中创建。

### 2.1.1 JVM对象实例化过程

```java
Object obj = new Object();
```

1.  在JVM堆中创建Object实例，并把结构化内存的首地址返回给obj存储到JVM栈帧中。
2.  实例只是一些数值，还需要Object类的类型定义才知道每个成员变量的内存范围。
3.  所以需要在实例中再创建一个引用指向，方法区对象类型数据（类定义）。

![img](C:\code\github\java-interview\img\jvm-7.png)

## 2.2 垃圾回收器与内存分配策略

两种判断Rechable和Unrechable对象的方法：

- 引用计数算法：引用此对象实例计数器就加一，引用失效就减一；但问题是很难解决对象之间的循环引用问题。相互依赖，关联关系，各自把对象引用作为成员变量，此时双方的计数器均为1，当两个对象置null时，计数器无法减为0，因为对象的引用未失效。

- ```java
  obj1.val = obj2;
  obj2.val = obj1;
  obj1 = null;
  obj2 = null;
  //但是val的引用未失效，计数器还是1
  //obj1.val = null;
  //obj2.val = null;
  ```

- 根搜索算法：以图论思想来检测不可达的节点，此时节点即对象实例，边或路径即引用链接。从GCroots很多个根节点出发，这些根节点可为栈中的引用对象，方法区静态变量引用对象，方法区常量引用对象，Native方法引用对象。

四种引用类型（一种优化，如果内存很大，可以保存，如果小清除）

- 强引用：普通引用
- 软引用：还有用但非必需对象，内存不足，会清除对象实例回收内存
- 弱引用：非必需对象，强度比软引用弱；无论内存如何都会回收内存
- 虚引用：最弱引用，唯一目的是在对象被清除时收到GC通知

为什么Eden和两个survivor区内存分配大小为8：1：1？

经验总结，一般Eden区回收垃圾最大可达70%~95%之多，所以剩下的可达对象实例完全可以放在survivor中。

类卸载满足条件：

- 类所有实例已被回收，没有实例了
- 加载该类的ClassLoader已经被回收
- Class对象没有被引用，无法反射访问Class任何方法

## 2.3 GC算法

标记清除法（mark-swap）：标记所有清除对象然后清除。两个问题：

- 效率 - 标记和清除都不高
- 空间 - 产生大量不连续碎片，空间碎片多无法再次分配大内存给对象实例，会再次触发垃圾回收

标记复制算法：就是把Rechable对象复制到servivor，然后Eden区全清除，一开始设计的是5：5，后来根据经验改进为8：1：1。

标记整理法（mark-Compact）：

用于老年代GC清理，先标记存活对象，然后把这些对象整理到一端，集中起来，最后把边界以外的unrechable对象清理。

所以总体选择分代回收方式，新生代用标记复制算法，老年代用标记整理算法。

![img](C:\code\github\java-interview\img\jvm-8.png)

Serial收集器：单线程版的收集器，会让所有工作线程停止运行。

![img](C:\code\github\java-interview\img\jvm-9.png)

Parallel New收集器：多线程并行的收集器，可以和CMS合作

![img](C:\code\github\java-interview\img\jvm-10.png)

Parallel Scavenge觅食收集器：复制算法并行多线程新生代收集器，特别处是可以精准的控制吞吐量。

![img](C:\code\github\java-interview\img\jvm-11.png)

Concurrent Mark Sweep CMS收集器：多线程并发老年代收集器，特别处是获取最短回收停顿时间，使用标记清除法。

![img](C:\code\github\java-interview\img\jvm-12.png)

Garbage first G1收集器：使用标记整理老年代垃圾，没有空间碎片；精确控制停顿，不牺牲吞吐量的前提降低回收停顿。

G1对新生代minor gc采用复制算法，从Eden复制到Survivor，每次复制增加一次年龄，当到达预设值后移到老年代

G1对老年代mixed gc采用标记整理算法，当mixed gc无法使用就使用fullgc，注意fullgc效率很低，再注意mixed gc会同时清理新生代和老年代，这和minor gc不冲突，因为我们设置了一个垃圾占比值，先用minor gc处理，当达到占比值时使用mixed gc处理。

mixed gc前需要并发标记：

1.   Initial marking phase: 标记GCRoots(会STW)；
2.   Root region scanning phase：标记存活Region；
3.   Concurrent marking phase：标记存活的对象；
4.   Remark phase：重新标记(会STW)；
5.   Cleanup phase：回收内存。

## 2.4 补充

为什么垃圾回收要采用分代模式？ 减少Full GC的次数（FullGC就是Major GC）

### 2.4.1 垃圾回收10种模式

1. 早期 Serial + Serial Old：单线程清除垃圾，停顿时间长，淘汰了（建个虚拟机分配单核CPU，默认Serial！）

2. 改进 Parallel Scavenge + Parallel Old：JDK 1.8 默认的组合PS+PO，简称UseParallel；多线程并发回收垃圾，无法清除浮动垃圾；复制算法+标记整理，一次回收几个GB。
3. 调优方向ParNew + CMS：把默认组合调参成这种组合，如果调不了最优就用G1，并发垃圾回收不搞STW，但有浮动垃圾；并发，停顿时间短为目标；标记时还是STW， 复制算法+标记压缩，一次回收几十个GB
   - 四个阶段
     初始标记：STW很短，只找到ROOT对象，不相信索图
     		并发标记：不STW，与工作线程并发执行，索图标记对象
     		重新标记：STW短，修正标错或漏标的，必须重头在重新扫码一遍
     		并发清理：清除垃圾，会产生浮动垃圾，碎片化严重，最后调用serial old来清理，触发FULLGC
     PN和PS差不多，为了配合CMS而产生了PN
4. 新思维G1：基于分代的GC无论如何调都无法最优，内存空间越来越大，所以G1产生以局部清理为主，而不是全部的空间区域，不用分代
   思想：G1把内存分为很多小的区域称为Region，这样按区域回收就不会STW
   并发标记使用三色标记算法
5. 颜色指针 ZGC：G1缺点，结构占内存多；前18位不用，中间分为四段，后面42为做内存
   四个颜色指针
   		1.marked 0  已标记
   		2.marked 1  已标记
        		3. remapped  正在移动的对象，产生一个读屏障，当对象移动完成再标记
                    	4. fianalizable 销毁对象
                   支持内存大小：2的42次方是4TB内存，最大可支持16TB，44次方，因为现代计算机最大总线有48条，不是64条，考虑经济因素，所以48-4（个指针）=44次方，CPU根据指令总线还是数据总线区分内存传过来的是数据还是指令。

### 2.4.2 如何标记的并防止错标漏标？

##### 三色标记法

​	白色：未被标记对象
​	灰色：自身被标记，成员变量未被标记
​	黑色：自身和成员变量均已标记完成

![image-20200522182716413](C:\code\github\java-interview\img\jvm-14.png)	漏标，A黑色标记完了，但又重新指向D，GC以为D是白色的没有标记，就清除了，漏标 
CMS处理方式： Incremental  Update 增量更新；把A再次变成灰色（写屏障）这样重新标记时会再扫码A。

![image-20200522182740734](C:\code\github\java-interview\img\jvm-16.png)并发标记，产生漏标：ABA问题，线程1把A标称黑色，线程2发现A有连接又把A标位灰色，但线程1是在2后执行的，最后A是黑色的。所以重新标记必须重新扫描一遍。
G1处理方式：SATB，snapshot at the beginning  初始快照；对断开的引用对象快照，保存到堆栈区，当再次扫码时，去检测堆栈是否为空；后续GC重点优化的地方，时间长待优化。

## 2.5 JVM命令

### 2.5.1 命令类型

-命令：直接用，正规命令
-X命令：非正规命令，不太稳定
-XX命令：下个版本可能淘汰
JVM调整命令

-命令：直接用，正规命令
-X命令：非正规命令，不太稳定
-XX命令：下个版本可能淘汰
JVM调整命令：1. -XX:+UseTuning

### 2.5.2 常用查看命令

-XX:+PrintCommandLineFlags：打印命令行参数，可以查看堆大小，使用了什么GC

```
-XX:InitialHeapSize=131714624 
-XX:MaxHeapSize=2107433984 
-XX:+PrintCommandLineFlags 
-XX:+UseCompressedClassPointers 
-XX:+UseCompressedOops 
-XX:-UseLargePagesIndividualAllocation 
-XX:+UseParallelGC 
```

-XX:+PrintFlagsInitial，默认值，初始化值
-XX:+PrintFlagsFinal，设置值（最终生效的值）
-XX:+PrintGCDetails，打印GC详细信息
-XX:+PrintGC，查看使用了什么GC；更新：-Xlog:gc instead

### 2.5.3 Parallel常用参数

-XX:ServiorRatio：存活区占比
-XXPreTenureSizeThreshold：设置大对象大小
-XX:MaxTenuringThreshold：调整年龄
-XX:+ParallelGCThreads：并行收集器线程数

```

```

# 3 调优案例

## 3.1 FullGC情况

### 3.1.1 一个案例

```java
package cn.edu.zju.a06jvm;

import com.alibaba.algorithm.zuochengyun.Question;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

@Question(
        question = "模拟FullGC问题，并调优",
        condition = "从数据库中读取信用数据，套用模型，并把结果进行记录和传输",
        solution = "java -Xms:30MB -Xmx:30MB -XX:+PrintGC file"
)
public class A05FullGCProblem {
    private static class CardInfo{
        BigDecimal price = new BigDecimal(0.0);
        String name = "张三";
        int age = 30;
        Date birthDate = new Date();

        public void m(){}
    }

    private static ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(
            50,
            new ThreadPoolExecutor.DiscardOldestPolicy()
    );

    private static void modelFit(){
        List<CardInfo> takeCard = getAllCardInfo();
        takeCard.forEach( cardInfo -> {
            executor.scheduleWithFixedDelay(
                    ()->{ cardInfo.m();},  // Lambda runnable
                    2, 3, TimeUnit.SECONDS
                    );
        });
    }

    private static List<CardInfo> getAllCardInfo() {
        List<CardInfo> cardInfos = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            CardInfo cardInfo = new CardInfo();
            cardInfos.add(cardInfo);
        }
        return cardInfos;
    }

    public static void main(String[] args) throws InterruptedException {
        executor.setMaximumPoolSize(50);

        for(;;){
            modelFit();
            Thread.sleep(100);
        }
    }
}
```

### 3.1.2 目的与现象

线程池操作，从数据库读取大量信息，并放入模型（一个方法）中处理，返回结果。

分配30MB，内存不够用，提示Allocation Failure 不断地 Full GC，无法工作。

## 3.2 调试方法

### 3.2.1 看GC分配情况

`-XX:+PrintGC`  或 `-XLog:gc`   先不断的监视GC情况才能发现问题 。

`-XX:PrintCommandLineFlags` 打印项目问题。

### 3.2.2 查看线程

- `jps` 直接返回所有运行的 Java 的线程
- `top -Hp pid` 动态监控该线程的CPU和内存占用
- `ps a` 查看线程情况

ps 就是 process status 的缩写，即进程的状态。

### 3.2.3 看JVM状态

拿到pid后 可以使用 `jstat -gc pid 1000` 动态查看JVM状态了，表示每隔 1 秒就打印出 gc 的状态信息。

也可以使用 `jinfo` 查看JVM相关参数时候有异常。

### 3.2.4 分析线程问题

`jstack pid`  打印出线程栈信息和所有线程信息，可以看到线程运行状态和是否发生死锁。

`jviusalvm` 一个提供图形界面的信息窗口，但在生产环境下没法使用

`arthas`  一款功能强大的开源调试工具，因为是基于java编写的运行还是要用jvm，所以当jvm快要崩溃的时候是无法进入无法调试的。

其提供了一些指令：

- `dashboard`  动态显示CPU和内存的变化
- `thread pid`   可以通过指定线程pid 直接看到该线程栈的情况
- `sc`  search class 找到该类加载的所有方法和类信息
- `trace`  跟踪方法调用的过程 和调用的时间
- `heapdump` 打印堆信息，实际生产中不能使用

### 3.2.5 分析内存情况

` hmap -histo  pid | head -20`  生成内存分布图，把当前时刻内存中所有的实例打印出来。多次间隔打印，查看实例最多的方法或对象，就是要排查的大头。生产环境不能随便用，影响大至卡顿，电商不适用，但很多服务器备用（高可用），这台卡顿没多大影响，可以用于测试。

`arthas heapdump` 记录好日志，怀疑那块就跟踪那块，解决，jad 动态反编译，排查版本问题，redefine 热替换class文件。

## 3.3 解决办法

### 3.3.1 一个是CPU飙升

CPU问题肯定是线程引起的，查找是否线程开过多，或线程阻塞，或CAS操作。

通过 `jstack` 查看。

### 3.3.2 一个是OOM

OOM 即out of memory 超过了内存区域，内存溢出。先推断内存设置的是否过小。

还有就是内存是否被容器分配完而没有及时释放，GC问题（频繁GC）。

通过`jmap` 打印出堆内存。

## 3.4 设置日志文件

日志文件是保护案发现场最好的工具，也是案后分析问题的最有力的依据。在设置日志是有技巧的，不能一概设置一个文件而是要多设置几个文件轮流存储。不然日志会太大打不开。

```shell
-Xloggc:/var/log/javagc/my-log-%t.log
-XX:+UseGCFileRotaion
-XX:NumberOfGCLogFiles=5
-XX:GCLogFileSize=20M
-XX:PrintGCDetails
-XX:PrintGCDetailStamps
-XX:PrintGCCause
-XX:HeapDumpOnOutOfMemoryError
```

