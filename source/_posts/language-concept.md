---
title: 语言设计理念
date: 2020-10-22 21:06:43
categories:
	- 笔记
tags: 
	- JAVA	
	- 编程
---

# 1. JDK 中设计理念

掌握了语言基础后，下一步就应该关注并收集语言的设计理念和设计模式。JDK 本身就是很好的学习范本，里面有很多设计模式和设计理念，当遇到问题时如何从代码切入。

## 1.1 如何实现有返回值的线程调用

Java 中提供了 Runnable 接口以便实现线程的构造，并在 Thread 中调用，创建新的线程执行。但 Runnable 是没有返回值的，如何设计有返回值的线程调用呢？

分析：首先，设计新功能时尽量基于旧功能改造，比如继承类、实现接口等，在原有功能改造的好处是系统设计更合理紧凑，不会有重复代码，代码复用率高；其次，更省时省力，节省成本和工作量。

通过比较，其实需求就是让 Runable  返回值，其他功能是可以完全让 Runnable 完成的。这样可以类比 Runnable 起个有区别的名字叫 Callable，但两者的区别只是 Callable 的接口方法有个返回值而已。

```java
// runnable
public void run(){}

// callable
public T call(){}
```

调用：构造线程后，需要专门的启动线程的类，Runnable 用 Thread，那么 Callable 用什么？这里一个鲜明的设计理念就出来了，尽一切可能复用。也要让 Thread 类启动 Callable。

问题：Thread 的参数是 Runnable，要想复用就需要把 Callable 转为 Runnable，如何转换呢？

这里又一个设计理念：通过接口赋能的方式来构建转换类。实现接口来满足 Thread 参数。即 定义一个类 FutureTask，让这个类来实现 Runnable 就可以传递给 Thread 了。

```java
public class FutureTask implements Runnable{
  private Callable<T> callable;
  public FutureTask(Callable<T> callable){
    this.callable = callable;
  }
}
```

FutureTask 用依赖的形式把 callable 作为成员变量，然后用构造方法传值。

在设计类时，还要遵循一个设计理念，那就是面向接口编程。为了以后扩展和维护，就像Runnable 赋能一样。我们需要设计一个接口 Future 来规范化如何得到线程的返回值。

```java
public interface Future {
  T get();
}
```

定义一个得到返回值的方法，这样只要实现 Future 类就能够得到返回值，以接口编程实现功能。

现在方案：Callable 是面向使用者的，用于构造有返回值的线程， 通过 FutureTask 类包装转为 Thread 去执行。而 FutureTask 通过实现 Runnable 和 Future 接口来赋能，就有了运行能力和返回能力。

改进：实现两个接口有点繁琐，可以再抽象一下，让一个接口来实现两个接口，然后去实现这个接口。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> { }

public class FutureTask implements RunnableFuture<V> {}
```

## 1.2 关系

复用包括组合和继承，直接在类里声明类是组合，继承类并重写方法是继承；

所以重写和重载的区别是：重写只在继承中出现，不改变方法名修饰符参数只是方法体不同，

​                          重载是除了方法名相同以外其他都可以不同。

关于C#中的委托：只要把组合和继承一部分结合起来就是委托，即在类里声明类并在方法体内调用该类方法而不暴露给main。

如何在不污染源代码的前提下使用现存代码是需要技巧的？

1. 在新类中创建现有类的对象。这种方式叫做“组合”（Composition），通过这种方式复用代码的功能，而非其形式
2. 采用现有类形式，又无需在编码时改动其代码，这种方式就叫做“继承”（Inheritance），编译器会做大部分的工作。继承是面向对象编程（OOP）的重要基础之一。



# 2. 设计模式

[网上有很好的学习平台](https://refactoringguru.cn/design-patterns/catalog)

![](/images/pattern-1.png)

![](/images/pattern-2.png)

## 2.1 工厂方法

主要用于多个同质不同形类创建，集成实例化操作。

### 2.1.1 UML图和原则

![](/images/pattern-3.png)

问题：多个事件类，如何根据DP点，自动调用相应的处理器来处理事件？

解决：直接调用工厂方法，传入DP点作为参数，维护一个key为DP点的HashMap，返回事件处理器。

```java
public class MqttEventFactory {

    /** dp点定义*/
    private Integer dp;

    /** 事件处理类*/
    private MqttEventHandler eventHandler;

    /** 事件集合*/
    Map<Integer, MqttEventHandler> handlerMap;

    /** 先加载各个事件处理器*/
    public void init(){
        handlerMap = new HashMap<>();
        // 状态上报（上报）
        handlerMap.put(125, new MqttStatusUploadEventHandler());
        // 通行事件（上报）
        handlerMap.put(126, new MqttPassOverEventHandler());
        // 开门事件（下发）
        handlerMap.put(127, new MqttDoorOpenEventHandler());
        // 云可视对讲事件（下发）
        handlerMap.put(128, new MqttCallAppEventHandler());
    }

    /** 根据 dp 点得到相应的事件处理器 */
    // 异常的处理，dp点不存在
    public MqttEventHandler getEventHandler(Integer dp){
        if (!handlerMap.containsKey(dp)) {
            return new MqttExceptionEventHandler();
        }
        return handlerMap.get(dp);
    }

}
```

### 2.1.2 实现细节和特点

- 一个工厂类，提供不同事件类的实例化对象
- 调用类，直接调用工厂类并传入不同的参数得到实例对象，而不是自己创建



## 2.2 模版方法模式

主要用于暴露子环节功能，让三方去实现具体功能

### 2.2.1 UML图和原则

![](/images/pattern-4.png)

业务：APP下发指令要新增人脸，设备端存储人脸，并返回结果

问题：三方要保存人脸图片到本地设备，如何拿到Url下载图片？

解决：提供一个接口方法，让三方实现，然后调用这个方法即可。

### 2.2.2 实现与特点

```java
// 三方实现这个类的方法的方法体
public class FaceImageReceiveEventImpl implements FaceImageReceiveEvent{
    @Override
    public void addFaceImage(FaceImageRequest f, EventContext e) {
        String url = f.getFaceImgUrl();
        log.info("{} : get url {}", ConstTag.FACE_IMAGE_DEVICE, url);
        log.info("{} : save image done !", ConstTag.FACE_IMAGE_DEVICE);
    }
}

// SDK 中构造参数并调用方法
public class FaceImageReceiveSDK {
    public void addFaceImageEventHandler(String uid, String fid, String url){
        FaceImageRequest f = new FaceImageRequest();
        f.setUid(uid);
        f.setFaceId(fid);
        f.setFaceImgUrl(url);
        EventContext e = new EventContext();
        FaceImageReceiveEvent event = new FaceImageReceiveEventImpl();
        event.addFaceImage(f, e);
    }
}
```

