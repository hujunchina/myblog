---
title: Java 笔记
date: 2020-10-22 21:11:37
categories:
	- 笔记
tags: 
	- JAVA
	- 编程
---

# 1. Java 基础知识

- 2020-06-25 整理旧笔记

## 1.1 Lambda 表达式

```java
interface CheckPerson {
    boolean test(Person p);
}
printPersons(Roster roster, CheckPerson checkPerson);  // 定义方法
printPersons(
    roster,
    new CheckPerson() {
        public boolean test(Person p) {
            return p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25;
        }
    }
);  // 传统调用方式
printPersons(
    roster,
    (Person p) -> p.getGender() == Person.Sex.MALE
        && p.getAge() >= 18
        && p.getAge() <= 25
);   // 使用 lambda 表达式
roster
    .stream()
    .filter(
        p -> p.getGender() == Person.Sex.MALE
            && p.getAge() >= 18
            && p.getAge() <= 25)
    .map(p -> p.getEmailAddress())
    .forEach(email -> System.out.println(email));  // 使用 lambda + 流
```

Lambda 表达式的特点：

> funtion_name(args) { body }
>
> (args) -> { body } 【因为调用指定了方法名，所以完全可以省略】
>
> Args -> body        【如果只有一个参数，或只有一条执行语句】

### 1.1.1 用Lambda表达式表示实现接口的匿名函数（实现里面的抽象方法）

```java
btn.setOnAction(new EventHandler<ActionEvent>() {
    @Override
    public void handle(ActionEvent event) {
        System.out.println("Hello World!");
    }
});
-----
btn.setOnAction(
    event -> System.out.println("Hello World!")
);

public interface Runnable(){
    void run();
}
public interface Callable<T>{
    T call();
}
------------------------------
void invoke(Runnable r){
    r.run();
}
T invoke(Callable<T> c){
    c.call();
}
-----------------------------
String str = invoke(()->"done");
```

### 1.1.2 demo

```java
static <T> void sort(T[] a, Comparator<? super T> c)
Arrays.sort(rosterAsArray,
    (Person a, Person b) -> {
        return a.getBirthday().compareTo(b.getBirthday());
    }
);
---------------upgrade----------------
Arrays.sort(rosterAsArray,
    (a, b) -> Person.compareByAge(a, b)
);
Arrays.sort(rosterAsArray, Person::compareByAge);

//静态方法
ContainingClass::staticMethodName
eg: Person::compareByAge
//一般方法
containingObject::instanceMethodName
ComparisonProvider myComparisonProvider = new ComparisonProvider();
Arrays.sort(rosterAsArray, myComparisonProvider::compareByName);
//类型中的方法
ContainingType::methodName
Arrays.sort(stringArray, String::compareToIgnoreCase);
//构造方法
ClassName::new
Set<Person> rosterSetLambda = transferElements(roster, () -> { return new HashSet<>(); });
Set<Person> rosterSet = transferElements(roster, HashSet::new);
```

## 1.2 JVM 参数

| 参数                                | 缩写                       | 含义                                                         | 使用                                                         |
| ----------------------------------- | -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -Xss                                | stack space                | 设置栈内存大小                                               | -Xss100M                                                     |
| -Xms                                | memory space               | 设置初始堆内存大小                                           | -Xms500M                                                     |
| -Xmx                                | memory max space           | 设置堆最大内存                                               | -Xmx500M                                                     |
| -Xmn                                | Memory  naive              | 设置年轻代大小                                               | -Xmn250M                                                     |
| --XX:MaxMetaspaceSize               | MaxMetaspaceSize           | 设置元区域大小                                               |                                                              |
| -Xnoclassgc                         | no class gc                | 使GC不对方法区类信息卸载                                     |                                                              |
| -XX:+UseConcMarkSweepGC             | use CMS gc                 | 设置使用CMS GC                                               |                                                              |
| -XX:+UseG1GC                        | use G1 gc                  | 设置使用G1 GC                                                |                                                              |
| -XX:+UseCMSCompact AtFullCollection | use compact at full gc     | 在fullgc下对内存整理                                         |                                                              |
| -XX:CMSFullGCs BeforeCompaction     | one of fullgc compact      | 执行多次fullgc后来一次压缩                                   |                                                              |
| -XX:DisableExplicitGC               | disable explicit full gc   | 禁止调用full gc                                              |                                                              |
| -XX:PretenureSizeThreshold          | new to old gen size        | 晋升年老代的对象大小, 10M                                    |                                                              |
| -XX:MaxTenuringThreshold            | new to old gen max-age     | 晋升老年代的最大年龄,15                                      |                                                              |
| jps                                 | jvm process state tool     | 显示 HotSpot 虚拟机进程。                                    | jps -l -m                                                    |
| jstat                               | jvm statistics Monitoring  | 监视虚拟机运行时状态，类装载、内存、垃圾收集、JIT 编译等运行数据。 | jstat -gc/-gccapacity/-gcnew/-gcnewcapacity/-gcold/-gcutil pid |
| jmap                                | jvm memory map             | 用于生成堆转储快照                                           |                                                              |
| jstack                              | Stack Trace for java       | 生成 java 虚拟机当前时刻线程快照                             |                                                              |
| jhat                                | java hat                   | 用来分析 jmap 生成的 dump 文件                               |                                                              |
| jinfo                               | jvm information            | 用于实时查看和调整虚拟机运行参数                             |                                                              |
| -XX:NewRatio                        | new / old gen ratio        | 设置新生代老年代比例                                         |                                                              |
| -XX:NewSize                         | new gen size               | 新生代内存大小                                               |                                                              |
| -XX:SurviorRatio                    | eden / survior arear ratio | eden/存活区内存大小比例                                      |                                                              |

## 1.3 切面

```java
public class StartEventHandler
   implements ApplicationListener<ContextStartedEvent>{
   public void onApplicationEvent(ContextStartedEvent event) {
      System.out.println("ContextStartedEvent Received");
   }
}
public class StopEventHandler
   implements ApplicationListener<ContextStoppedEvent>{
   public void onApplicationEvent(ContextStoppedEvent event) {
      System.out.println("ContextStoppedEvent Received");
   }
}
public class AOPService {
    public void beforeService() {
        System.out.println("before aop service");
    }
}
---------------- 
<bean id="startEventHandler" class="cn.edu.zju.StartEventHandler"/>
<bean id="stopEventHandler" class="cn.edu.zju.StopEventHandler"/>
<aop:config>
  <aop:aspect id="myAOP" ref="aopService">
    <aop:pointcut id="getName" expression="execution(*HelloWorld.getName(..))"/>
      <aop:before pointcut-ref="getName" method="beforeService"></aop:before>
    </aop:aspect>
</aop:config>
<bean id="aopService" class="cn.edu.zju.AOPService"></bean>
```

(动态代理)

切面有哪些标签注解？