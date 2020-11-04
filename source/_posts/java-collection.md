---
title: JAVA集合类
date: 2020-10-26 11:20:12
categories:
	- 必备知识
tags:
	- JAVA
	- 编程
---

# 1. ArrayList

## 1.1. ArrayList是什么

java.util.ArrayList是一个动态可变长数组，本质上是一个初始长度为10的定长 Object 数组。当超过初始长度时，使用重新分配堆大小方式对数组扩容1.5倍，以实现变长的特性。

使用泛型<E>表示可以接受不同类型元素。

## 1.2. ArrayList继承关系

```java
public class ArrayList<E> extends AbstractList<E> 
implements List<E>, RandomAccess, Cloneable, Serializable
---------------
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E>
---------------
public abstract class AbstractCollection<E> implements Collection<E>
---------------
public interface List<E> extends Collection<E>
```

ArrayList继承AbstractList抽象类，该抽象类又实现List接口，List接口又继承Collection接口（Idea中按Ctrl+B快速定位到类声明文件）。

Java类的设计方式，一般是一个统一的接口，然后让一个抽象类去实现接口，最后再让具体的类继承抽象类（接口不能实现接口，因为接口没有实现的能力，里面是抽象方法）。

![](/images/arraylist01.png)

## 1.3. ArrayList成员属性

```java
private static final long serialVersionUID = 8683452581122892189L;
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] EMPTY_ELEMENTDATA = new Object[0];
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];
transient Object[] elementData;
private int size;
private static final int MAX_ARRAY_SIZE = 2147483639;
```

因为ArrayList实现了 Serializable 接口，所以设置了 serialVersionUID，避免反序列化时因不同序列号而找不到类报错。

关于序列化，会给序列化类关联一个ID，称为序列号，在反序列化过程中，用于验证序列化类的发送者和接收者是否已经加载了该序列化类。如果不指明序列号，会自动生成一个序列号。这个生成过程根据 JVM 版本、类的定义等不同导致序列号不一样，那么多次加载的时候就会报错 InvalidClassException。

如A引用了B，同时对A和B序列化，A是接受者会保留B的序列号，B是发送者。如果对B修改再次序列化，那么反序列化时，A就会报错找不到B。因为对B的两次序列化，得到的序列号不一样。

数组容量为0，只有真正对数据进行添加add时，才分配DEFAULT_CAPACITY 大小为10，并把初始数组类型定为 Object，这也解释了为什么不能用基本类型，而要用int 的引用类型 Integer。因为放入 ArrayList 中的元素必须要继承 Object，其默认数组类型就是 Object，不然无法存储。

定义了一个实时大小 size，和最大的容量 2147483639，21亿多。

实验不停插入，结果报 `java.lang.OutOfMemoryError: Java heap space` 错误。

## 1.4. ArrayList构造方法

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else {
        if (initialCapacity != 0) {
            throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
        }
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
public ArrayList(Collection<? extends E> c) {
    this.elementData = c.toArray();
    if ((this.size = this.elementData.length) != 0) {
        if (this.elementData.getClass() != Object[].class) {
            this.elementData = 
            Arrays.copyOf(this.elementData, this.size, Object[].class);
        }
    } else {
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

ArrayList 构造方法有三个。前两个是指定数组大小的，说明在初始化时可以控制数组大小。但并没有给size=initialCapacity，所以我们获得大小还是存多少是多少，并不是初始大小。

由于没有用到泛型，还说明可以在 arraylist 中混合放入不同类型。

第三个构造方法，可以传递一个 Collection 子类集合，来把元素直接复制到里面。这里使用了泛型的 extends 表示上界，表示集合中的元素范围要比 ArrayList 声明的元素小，？extends  extends E 。原因是保证加入的元素可以放入 ArrayList 中，不会报类型错误。

这里还使用 getClass 获得方法区的类结构信息与object数组类结构比较。

如果 toArray 方法返回的不是 Object[] 类型，比如我们自己定义一个实现 Collection 的类并覆盖重写了 toArray，但返回的是 String[] 类型，这个时候就会调用 Arrays.copyOf 方法，并传入参数 Object[].class 强制转为 Object[] 类型。

```java
public static <T, U> T[] copyOf(
    U[] original, int newLength, Class<? extends T[]> newType) {
    T[] copy = 
        newType == Object[].class ? 
        new Object[newLength] : 
        (Object[])Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

## 1.5. ArrayList add方法

当我们实例化一个未指定大小的 ArrayList 的对象时，其默认大小不是10而是0，只有在第一次放入元素时才更改大小为10。这样设计很巧妙，节省堆内存并提高运行效率，因为实例化后不知道用不用呢，object  数组大小为0即节省堆内存又快速。那么 add 方法是如何做的呢？

```java
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length) {
        elementData = this.grow();
    }
    elementData[s] = e;
    this.size = s + 1;
}
public boolean add(E e) {
    ++this.modCount;
    this.add(e, this.elementData, this.size);
    return true;
}
public void add(int index, E element) {
    this.rangeCheckForAdd(index);
    ++this.modCount;
    int s;
    Object[] elementData;
    if ((s = this.size) == (elementData = this.elementData).length) {
        elementData = this.grow();
    }
    // 空出一个位置
    System.arraycopy(elementData, index, 
                     elementData, index + 1, 
                     s - index);
    // 插入元素
    elementData[index] = element;
    this.size = s + 1;
}
```

ArrayList提供了三个add方法，一个private，两个public，其实我们只能用后两个add方法。

- add(E e) 直接把元素插入到ArrayList中
- add(int index, E element) 指定插入下标，并插入元素

代码写的很严谨，上来就把modCount加1，这个变量是在父类AbstractList声明的。

`protected transient int modCount = 0;`

源代码使用 protected 修饰表示最大访问权限到子类，然后使用 transient 修饰表示不序列化这个属性。modCount 表示操作次数，用于并发。

随后调用私有 add 方法。在其方法中调用了 grow() 方法来增加长度，并把要插入元素放入最后并更新size。

所以准确讲，是容量**等于**长度时，才扩容。

对第二个add方法，下标必定在新扩容长度的范围内。这就引出了一个问题，我们如何高效的插入这个元素呢？肯定不是一个一个往后移然后插入。

![image-20201026112908208](/images/arraylist02.png)

这里使用arraycopy方法后半段整体复制的方式插入。

```java
arraycopy(Object src, int srcPos, 
          Object dest, int destPos, 
          int length)
Copies an array from the specified source array, 
beginning at the specified position, 
to the specified position of the destination array.
```

Java文档写的从起始下标位置复制一定长度的元素到目的下标位置。

即直接把[index, size]后半段数组整体复制到[index+1, size+1]位置，这样就留出了index位置，插入元素。

## 1.6. ArrayList grow方法

在add方法中使用grow方法来扩容ArrayList。

```java
private Object[] grow(int minCapacity) {
    return this.elementData = 
    Arrays.copyOf(this.elementData, this.newCapacity(minCapacity));
}
private Object[] grow() {
    return this.grow(this.size + 1);
}
private int newCapacity(int minCapacity) {
    int oldCapacity = this.elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (this.elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(10, minCapacity);
        } else if (minCapacity < 0) {
            throw new OutOfMemoryError();
        } else {
            return minCapacity;
        }
    } else {
        return newCapacity - 2147483639 <= 0 ? newCapacity : hugeCapacity(minCapacity);
    }
}
```

grow 方法调用了 copyof 方法来复制数组，参数中又调用 newCapacity 方法。

在 newCapacity 方法中，我们可以看到` oldCapacity+(oldCapacity >> 1)`，表示数组容量变为原来的1.5倍。如果没有超过10，则设置为10。

```java
public static <T, U> T[] copyOf(
    U[] original, int newLength, Class<? extends T[]> newType) {
    T[] copy = 
        newType == Object[].class ? 
        new Object[newLength] : 
        (Object[])Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0, Math.min(original.length, newLength));
    return copy;
}
```

这里copyof方法是先实例化一个新容量大小的object数组，然后把旧的复制过去。这样说明了ArrayList扩容的方式不是直接分配堆内存，而是实例化新的对象并复制元素。

copyof 底层也是用 System.arraycopy 来操作复制的。

## 1.7. ArrayList 应用场景

谈应用场景，并不需要特别指明某一个业务，只要要结合ArrayList自身特点指明应用一个大的方面即可。

- ArrayList可变长，可以用于存储无法确定长度的场景
- ArrayList可快速获得数据，可以用于频繁查找和获取数据上
- ArrayList支持泛型，可以用于对象存储，方便管理对象。

## 1.8. ArrayList 小结

需要知道如下知识点：

- ArrayList是什么？
- ArrayList扩容怎么实现的？
- ArrayList容量满的时候，如何高效插入？
- ArrayList应用场景？

## 1.9. 补充

虽然初始化了大小，但是对 object 数组的大小，而不是 size，故不能 set(5, 1) 。

![image-20201026113441916](/images/arraylist03.png)

论遍历ArrayList要比LinkedList快得多，ArrayList遍历最大的优势在于内存的连续性，CPU的内部缓存结构会缓存连续的内存片段，可以大幅降低读取内存的性能开销。

ArrayList 不适合用作队列，适合用作堆栈；数组适合用作队列。

可以放入空值 null 。

```java
public static void main(String[] args) {
//  构造方法
//        transient Object[] elementData;
//        private int size;
    List<Integer> list = new ArrayList<>();
    List<Integer> list2 = new ArrayList<>(10);

    for (int i = 0; i < 10; i++) {
        list.add(i);
    }
    list.add(6, 10);
    //System.arraycopy(elementData, 3, elementData, 2, numMoved);
    //        把【3，end】直接复制到【2，end】位置
    list.remove(2);

    //        测试能不能放入空值
    List<Object> list3 = new ArrayList<>(5);
    list3.add("hujun");
    list3.add(null);
    //        可以放入null，占位置，可以输出
    System.out.println(list3.size());
    System.out.println(list3.get(1));
}

public static void array2(){
    int[] a = {1,2,3};
    int[] b;
    b = new int[]{1,2,3};
    int[] c = new int[5];
    System.out.println(c[0]);
    //        int[] 有长度属性
    int len = c.length;

    ArrayList<Integer> test = new ArrayList<>();
    for(int i=0; i<2247; i++){
        for(int j=0; j<83; j++){
            test.add(i+j);
        }
    }
    //        2 dims array
    int[][] arr2d = {{1,2,3},{4,5,6}};
    //       局部变量未初始化不会赋初值，编译报错
    //        int x;
    //        System.out.println(x);

    String[] as = {"a","C","b","D"};
    Arrays.sort(as,String.CASE_INSENSITIVE_ORDER);
    for(int i = 0; i < as.length; i ++) {
        System.out.print(as[i] + "\t");
    }
    Arrays.sort(as, Collections.reverseOrder());//Collections 集合类中的工具类
    for(int i = 0; i < as.length; i ++) {
        System.out.print(as[i] + "\t");
    }
}
```

# 2. LinkedList

## 2.1. LinkedList是什么?

java.util.LinkedList 是 Java 实现的一个双链表数据结构。支持泛型，可以存储 Object 及其子类作为元素。双向链表即可以访问前后元素的一种线性存储结构，每个节点存储了上一个元素引用和下一个元素的引用，保证可以前后访问到。双链表优点是插入和删除快速，无需移动其他元素即可完成，但查找效率低，需要遍历整个链表。

可以用 LinkedList 实现堆栈、队列等。

## 2.2. LinkedList继承关系?

```java
public class LinkedList<E> extends AbstractSequentialList<E> 
implements List<E>, Deque<E>, Cloneable, Serializable
------------------
public abstract class AbstractSequentialList<E> extends AbstractList<E>
------------------
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E>
------------------
public abstract class AbstractCollection<E> implements Collection<E>
------------------
```

LinkedList 继承了 AbstractSequentialList 抽象类，实现了List接口，AbstractSequentialList 又继承了AbstractList 抽象类，AbstractList 继承了 AbstractCollection 抽象类，AbstractCollection 实现 Collection接口。

## 2.3. LinkedList成员变量

```java
transient int size;
transient LinkedList.Node<E> first;
transient LinkedList.Node<E> last;
private static final long serialVersionUID = 876323262645176354L;
```

四个变量分别表示长度大小，前节点引用，后节点引用，序列化号。

为了实现链表功能，LinkedList定义了一个内部静态类Node。

```java
private static class Node<E> {
    E item;
    LinkedList.Node<E> next;
    LinkedList.Node<E> prev;
    Node(LinkedList.Node<E> prev, E element, LinkedList.Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

在Node类里面定义与C语言声明节点结构体一样。

## 2.4. LinkedList构造方法

```java
public LinkedList() {
    this.size = 0;
}
public LinkedList(Collection<? extends E> c) {
    this();
    this.addAll(c);
}
```

与 ArrayList 不同点是少了一个直接设置长度大小的构造函数，因为链表没有必要初始化大小。第二个方法参数是一个集合对象引用，然后调用 addAll 方法把集合中元素插入到链表中。

```java
public boolean addAll(Collection<? extends E> c) {
    return this.addAll(this.size, c);
}
public boolean addAll(int index, Collection<? extends E> c) {
    this.checkPositionIndex(index);
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0) {
        return false;
    } else {
        LinkedList.Node pred;
        LinkedList.Node succ;
        if (index == this.size) {
            succ = null;
            pred = this.last;
        } else {
            succ = this.node(index);
            pred = succ.prev;
        }
        Object[] var7 = a;
        int var8 = a.length;
        for(int var9 = 0; var9 < var8; ++var9) {
            Object o = var7[var9];
            LinkedList.Node<E> newNode = 
                new LinkedList.Node(pred, o,(LinkedList.Node)null);
            if (pred == null) {
                this.first = newNode;
            } else {
                pred.next = newNode;
            }
            pred = newNode;
        }
        if (succ == null) {
            this.last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }
        this.size += numNew;
        ++this.modCount;
        return true;
    }
}
```

在 addAll 方法中，首先把参数集合转为一个 object 类型数组，然后遍历这个数组取出元素，并加入pre和succ前驱和后驱构造 Node 节点。

## 2.5. LinkedList add方法

```java
public boolean add(E e) {
    this.linkLast(e);
    return true;
}
void linkLast(E e) {
    LinkedList.Node<E> l = this.last;  // last是全局位置标记
    LinkedList.Node<E> newNode = 
          new LinkedList.Node(l, e, (LinkedList.Node)null);
    this.last = newNode;  // 先更新last标记
    if (l == null) {     //调制链接
        this.first = newNode;
    } else {
        l.next = newNode;
    }
    ++this.size;
    ++this.modCount;
}
```

链表插入方法是 linkLast 插入到链表后面，因为有 last 记录了最后指针位置，所以可以直接插入。

## 2.6. LinkedList get方法

```java
public E get(int index) {
    this.checkElementIndex(index);
    return this.node(index).item;
}
LinkedList.Node<E> node(int index) {
    LinkedList.Node x;
    int i;
    if (index < this.size >> 1) {
        x = this.first;
        for(i = 0; i < index; ++i) {
            x = x.next;
        }
        return x;
    } else {
        x = this.last;
        for(i = this.size - 1; i > index; --i) {
            x = x.prev;
        }
        return x;
    }
}
```

我们可以指定下标顺序来获得元素，这里先比较下标与长度的一半的大小，来决定是前序遍历，还是后序遍历找到元素。

```java
public E getFirst() {
    LinkedList.Node<E> f = this.first;
    if (f == null) {
        throw new NoSuchElementException();
    } else {
        return f.item;
    }
}
public E getLast() {
    LinkedList.Node<E> l = this.last;
    if (l == null) {
        throw new NoSuchElementException();
    } else {
        return l.item;
    }
}
```

还提供了得到第一个，和得到最后一个元素方法。

```java
public E peek() {
    LinkedList.Node<E> f = this.first;
    return f == null ? null : f.item;
}
public E poll() {
    LinkedList.Node<E> f = this.first;
    return f == null ? null : this.unlinkFirst(f);
}
```

另外，peek 方法直接得到链表第一个节点元素，poll方法不仅得到第一个节点元素，还会把节点删除。（peek表示偷看，即偷偷看一眼链表，poll表示获得民意选票）。

## 2.7. LinkedList set方法

```java
public E set(int index, E element) {
  this.checkElementIndex(index);
  LinkedList.Node<E> x = this.node(index);
  E oldVal = x.item;
  x.item = element;
  return oldVal;
}
```

先通过 get 方法的到当前下标的 node 节点，然后直接赋值即可。

## 2.8. LinkedList remove方法

```java
public E remove() {
    return this.removeFirst();
}
public E remove(int index) {
    this.checkElementIndex(index);
    return this.unlink(this.node(index));
}
public boolean remove(Object o) {
    LinkedList.Node x;
    if (o == null) {
        for(x = this.first; x != null; x = x.next) {
            if (x.item == null) {
                this.unlink(x);
                return true;
            }
        }
    } else {
        for(x = this.first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                this.unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

如果指定下标很简单，直接unlink即可；如果不指定下标而指定元素，就需要遍历一遍链表后找到元素node节点再unlink。

```java
public E removeFirst() {
    LinkedList.Node<E> f = this.first;
    if (f == null) {
        throw new NoSuchElementException();
    } else {
        return this.unlinkFirst(f);
    }
}
public E removeLast() {
    LinkedList.Node<E> l = this.last;
    if (l == null) {
        throw new NoSuchElementException();
    } else {
        return this.unlinkLast(l);
    }
}
```

这两个方法均调用unlink方法实现删除。

```java
E unlink(LinkedList.Node<E> x) {
    E element = x.item;
    LinkedList.Node<E> next = x.next;
    LinkedList.Node<E> prev = x.prev;
    if (prev == null) {
        this.first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    if (next == null) {
        this.last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;
    --this.size;
    ++this.modCount;
    return element;
}
```

## 2.9. LinkedList 使用场景

根据LinkedList特点：

- 长度没有限制，适合不确定长度存储
- 插入删除快速，适合频繁修改场景
- 双向均可遍历，使遍历速度减少一半

## 2.10. 补充

`LinkedList` 集合底层实现的数据结构为双向链表

`LinkedList` 集合中元素允许为 null

`LinkedList` 允许存入重复的数据

`LinkedList` 中元素存放顺序为存入顺序。

`LinkedList` 是非线程安全的，如果想保证线程安全的前提下操作 `LinkedList`，可以使用 `List list = Collections.synchronizedList(new LinkedList(...));` 来生成一个线程安全的 `LinkedList`

```java
public static void main(String[] args) {
    //        简单列表
    List<String> list = new LinkedList<>();
    list.add("hujun");
    //        链表可以放入null，空值，但是站空间，长度会增长，还可以输出
    list.add(null);
    System.out.println(list.get(0));
    System.out.println(list.size());
    System.out.println(list.get(1));
    //      简单列表
    LinkedList<Object> list2 = new LinkedList<>();
    list2.add("12");
    list2.add(0,"hu");
    list2.add(null);
    System.out.println(list2.get(0));
    System.out.println(list2.size());

    //
    // 链表长度
    //transient int size = 0;
    //// 链表头节点
    //transient Node<E> first;
    //// 链表尾节点
    //transient Node<E> last;
    //        基于链表的队列
    Queue<String> queue = new LinkedList<>();
    queue.offer("hujun");
    System.out.println(queue.peek());
    queue.poll();

    LinkedList<Integer> doubleLink = new LinkedList<>();
    doubleLink.addFirst(1);
    doubleLink.addLast(2);
    doubleLink.removeFirst();
    doubleLink.removeLast();

    DequeTest();
}


public static void DequeTest(){
    Queue<String> queue = new LinkedList<>();
    queue.offer("a"); // 入队
    queue.offer("b"); // 入队
    queue.offer("c"); // 入队

    for (String q : queue) {
        System.out.println(q);
    }
    //      可以加入null
    queue.offer(null);
    queue.poll();
}
```

# 3. Vector

## 3.1. 介绍

底层实现与 ArrayList 类似，不过Vector 是线程安全的，而ArrayList 不是。

```java
public class Vector<E>  extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

**Vector** 是一个矢量队列，它的依赖关系跟 **ArrayList**  是一致的，因此它具有一下功能：

- 1、**Serializable**：支持对象实现序列化，虽然成员变量没有使用 transient 关键字修饰，Vector 还是实现了 writeObject() 方法进行序列化。
- 2、**Cloneable**：重写了 clone（）方法，通过 Arrays.copyOf（） 拷贝数组。
- 3、**RandomAccess**：提供了随机访问功能，我们可以通过元素的序号快速获取元素对象。
- 4、**AbstractList**：继承了AbstractList ，说明它是一个列表，拥有相应的增，删，查，改等功能。
- 5、**List**：留一个疑问，为什么继承了 AbstractList 还需要 实现List 接口？

### 3.1.1 成员变量

```java
/**
        与 ArrayList 中一致，elementData 是用于存储数据的。
     */
protected Object[] elementData;

/**
     * The number of valid components in this {@code Vector} object.
      与ArrayList 中的size 一样，保存数据的个数
     */
protected int elementCount;

/**
     * 设置Vector 的增长系数，如果为空，默认每次扩容2倍。
     *
     * @serial
     */
protected int capacityIncrement;

// 数组最大值
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

### 3.1.2 构造函数

```java
//默认构造函数
public Vector() {
    this(10);
}

//带初始容量构造函数
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}

//带初始容量和增长系数的构造函数
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```

## 3.2. 原理

### 3.2.1 Add 方法

```java
public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

```java
public void add(int index, E element) {
    insertElementAt(element, index);
}

public synchronized void insertElementAt(E obj, int index) {
    modCount++;
    if (index > elementCount) {
        throw new ArrayIndexOutOfBoundsException(index
                                                 + " > " + elementCount);
    }
    ensureCapacityHelper(elementCount + 1);
    System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
    elementData[index] = obj;
    elementCount++;
}
```

### 3.2.2 Grow 方法

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    //区别与ArrayList 中的位运算，这里支持自定义增长系数
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 3.2.3 删除方法

```java
public synchronized void removeElementAt(int index) {
    modCount++;
    if (index >= elementCount) {
        throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
    }
    else if (index < 0) {
        throw new ArrayIndexOutOfBoundsException(index);
    }
    int j = elementCount - index - 1;
    if (j > 0) {
        System.arraycopy(elementData, index + 1, elementData, index, j);
    }
    elementCount--;
    elementData[elementCount] = null; /* to let gc do its work */
}

public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return oldValue;
}
```

## 3.3. 原理

Vector 中的每一个独立方法都是线程安全的，因为它有着 synchronized 进行修饰。但是如果遇到一些比较复杂的操作，并且多个线程需要依靠 vector 进行相关的判断，那么这种时候就不是线程安全的了。

```java
if (vector.size() > 0) {
    System.out.println(vector.get(0));
}
```

如上述代码所示，Vector 判断完 size（）>0 之后，另一线程如果同时清空vector 对象，那么这时候就会出现异常。因此，在复合操作的情况下，Vector 并不是线程安全的。

vector 只有在插入删除是线程安全的，而对于长度和容量的处理不是线程安全的。

```java
Vector<String> v = new Vector<>();
v.add("hujun");
System.out.println(v.capacity());
System.out.println(v.get(0));
```

底层存储以 arraylist 一样，所以可以存储 null。

# 4. HashMap

## 4.1. HashMap 是什么

HashMap定义，以散列方式存储键值对，允许使用空值和空键。有两个影响其性能的参数：初始容量和负载因子，底层以桶存储。

- 容量是哈希表中桶的数量，初始容量就是哈希表创建时的容量。
- 加载因子是散列表在容量自动扩容之前被允许的最大饱和量。当哈希表中的 entry 数量超过负载因子和容量的乘积时，散列表就会被重新映射（即重建内部数据结构），一般散列表大约是存储桶数量的两倍。

桶大小默认16，负载因子0.75，即桶内元素饱和度，最多可用存12个，多了就要再散列，扩大为原来的2倍。

为什么默认加载因子是0.75？经验，0.75是时间和空间良好的平衡结果。如果大了，会减少空间开销，但查找成本高（重复的键多）。

## 4.2. HashMap 继承关系

```java
public class HashMap<K,V> extends AbstractMap<K,V> 
    implements Map<K,V>, Cloneable, Serializable
```

![image-20201026114740532](/images/hashmap01.png)

支持泛型，继承抽象类，实现必要的方法get、set

## 4.3. HashMap 成员属性

```java
private static final long serialVersionUID = 362498820763181265L;
static final int DEFAULT_INITIAL_CAPACITY = 16;
static final int MAXIMUM_CAPACITY = 1073741824;
static final float DEFAULT_LOAD_FACTOR = 0.75F;
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
static final int MIN_TREEIFY_CAPACITY = 64;
transient HashMap.Node<K, V>[] table;
transient Set<Entry<K, V>> entrySet;
transient int size;
transient int modCount;
int threshold;
final float loadFactor;
```

固定的声明为 static final，不想序列化的声明为 transient。

```java
static class Node<K, V> implements Entry<K, V> {
    final int hash;
    final K key;
    V value;
    HashMap.Node<K, V> next;
}
```

在Java7叫Entry，在Java8中叫Node。

数据存储的实体，设计思路是包裹一个静态内部类来实现具体数据结构。

## 4.4. HashMap 构造方法

**java8之前是头插法**，就是说新来的值会取代原有的值，原有的值就顺推到链表中去，就像上面的例子一样，因为写这个代码的作者认为后来的值被查找的可能性更大一点，提升查找的效率。

但是，在java8之后，都是所用尾部插入了。因为扩容容易产生环形链表。

![image-20201026114943231](/images/hashmap02.png)

因为resize的赋值方式，也就是使用了**单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置**，在旧数组中同一条Entry链上的元素，通过重新计算索引位置后，有可能被放到了新数组的不同位置上。

![image-20201026115015482](/images/hashmap03.png)

一旦几个线程都调整完成，就可能出现环形链表：

![image-20201026115124376](/images/hashmap04.png)

## 4.5. HashMap put方法

```java
if (((HashMap.Node)p).hash == hash && ((k = ((HashMap.Node)p).key) == key 
|| key != null && key.equals(k))) {
// 注意是&操作
if ((p = tab[i = n - 1 & hash]) == null) {
    tab[i] = this.newNode(hash, key, value, (HashMap.Node)null);
 }
 static final int hash(Object key) {
    int h;
    return key == null ? 0 : (h = key.hashCode()) ^ h >>> 16;
 }
```

- 对key做hash运算，得到key的hash值；key的hashCode先无符号左移16位，然后与hashCode按位异或。（hash值与hashCode不一样）

- 计算桶的index（数组的下标）；(n-1)&hash值

- 如果没有下标冲突，直接放入数组中，如果有，以链表形式放入桶后面（当前键值对的后面）next指针。

- 如果链表过长超过默认的阈值8，就会把链表转为红黑树。为什么？

  考虑到查找效率问题 log8=3。

- 如果数组中节点key已经存在了，就替换value值；

- 如果x>16*0.75=12 就调用resize自动扩容一倍。

### 4.5.1 Hash 为什么是 & ？

![image-20201026115219945](/images/hashmap05.png)

高16位不变，低16位和高16位做一个异或。如果不异或操作，因为与0x1111进行与操作有效位只有4位，重复率很高，很容产生冲突。

高位与低位异或一下可以让高位参与计算数组下标。hashCode分布已经很均匀了，高位也参与减少冲突。

一是比取余效率高，二是直接取hashcode的最后几位，借用hashcode的散列性来增加 put 到桶的散列性。

### 4.5.2 HashMap resize方法

影响因子饱和度问题，如果饱和了，就把数组扩大2倍。从16到32。

![image-20201026115350746](/images/hashmap06.png)

![image-20201026115420686](/images/hashmap07.png)

在扩充 HashMap 的时候，不需要重新计算 hash。

只需要看看原来的 hash 值新增的那个 bit 是 1 还是 0 就好了。

是 0 的话索引没变，是 1 的话索引变成 “原索引 + oldCap”。可以看看下图为 16 扩充为 32 的 resize 示意图：

![image-20201026115504532](/images/hashmap08.pn.png)

## 4.6. HashMap get方法

```java
if ((tab = this.table) != null && (n = tab.length) > 0 
    && (first = tab[n - 1 & hash]) != null) {
   Object k;
   if (first.hash == hash && ((k = first.key) == key || key != null && key.equals(k))) {
        return first;
   }
   if ((e = first.next) != null) {
        if (first instanceof HashMap.TreeNode) {
            return ((HashMap.TreeNode)first).getTreeNode(hash, key);
        }

        do {
            if (e.hash == hash && ((k = e.key) == key || key != null && key.equals(k))) {
                return e;
            }
        } while((e = e.next) != null);
    }
}
```

- 对key做hash计算，并找到数组下标；
- 比较如果hash值一样并且key一样，直接返回键值对；
- 对比发现不一样，dowhile继续比较hash和key两个条件知道相同。

## 4.7. HashMap 扩容方式

### 4.7.1 那为啥用16不用别的呢？

因为在使用是2的幂的数字的时候，Length-1的值是所有二进制位全为1，这种情况下，index的结果等同于HashCode后几位的值。

只要输入的HashCode本身分布均匀，Hash算法的结果就是均匀的。

这是为了**实现均匀分布**。

### 4.7.2 为啥我们重写equals方法的时候需要重写hashCode方法呢？

因为在java中，所有的对象都是继承于Object类。Ojbect类中有两个方法equals、hashCode，这两个方法都是用来比较两个对象是否相等的。

在未重写equals方法我们是继承了object的equals方法，**那里的 equals是比较两个对象的内存地址**，显然我们new了2个对象内存地址肯定不一样。

如果我们对 equals 方法进行了重写，建议一定要对 hashCode 方法重写，以保证相同的对象返回相同的hash值，不同的对象返回不同的hash值。

不然一个链表的对象，到时候发现hashCode都一样，无法确定那个元素！

还有一种情况，写了 equals 但没有写 hashcode，那么存入的元素取不出来。

### 4.7.3 冲突处理 红黑树

冲突后，使用链表形式存储，如果链表长度大于8，会转为红黑树，如果长度小于8，还是红黑树，不可逆。

## 4.8. HashMap 应用场景

```java
public static void main(String[] args) {
    System.out.println("do");
    Map<String, String> map = new HashMap<>();
    map.put("hujun", "liujihong");
    map.put("hujun", "covered");

    map.put(null, null);
    map.put(null, "covered null key");
    map.put("null", null);
    //        hashmap都可以放入null，占长度，均可输出
    System.out.println(map.get(null));
    System.out.println(map.get("null"));

    int h=5;
    int hash1 = (h^h)>>>16;
    int hash2 = h^h>>>16;
    System.out.format("%d, %d, %d", hash1, hash2, 5>>>16);
    System.out.println(4-1&3);
}

public static void iteratorTest(){
    Map<Integer, String> map = new HashMap<>();
    map.put(1, "12");
    map.put(2, "23");
    map.put(3, "34");

    Set<Map.Entry<Integer, String>> set = map.entrySet();
    Iterator<Map.Entry<Integer, String>> it = set.iterator();
    while (it.hasNext()){
        Map.Entry<Integer, String> entry = it.next();
        Integer key = entry.getKey();
        String val = entry.getValue();
        System.out.println(key+"=>"+val);
    }

    for(Map.Entry<Integer, String> entry:set){
        Integer key = entry.getKey();
        String val = entry.getValue();
        System.out.println(key+"=>"+val);
    }

    sortMap(map);
}

public static void sortMap(Map m){
    Set<Integer> set = m.keySet();
    for(Integer i : set){
        System.out.println(i+"=>"+m.get(i));
    }
}
```

# 5. LinkedHashMap

## 5.1. LinkedHashMap是什么

只要不涉及线程安全问题，Map基本都可以使用HashMap，不过HashMap有一个问题，就是迭代HashMap的顺序并不是HashMap放置的顺序，也就是无序。HashMap的这一缺点往往会带来困扰，因为有些场景，我们期待一个有序的Map。这就是我们的LinkedHashMap，看个小Demo:

```java
public static void main(String[] args) {
    Map<String, String> map = new LinkedHashMap<String, String>();
    map.put("apple", "苹果");
    map.put("watermelon", "西瓜");
    map.put("banana", "香蕉");
    map.put("peach", "桃子");

    Iterator iter = map.entrySet().iterator();
    while (iter.hasNext()) {
        Map.Entry entry = (Map.Entry) iter.next();
        System.out.println(entry.getKey() + "=" + entry.getValue());
    }
}
输出为：
apple=苹果
watermelon=西瓜
banana=香蕉
peach=桃子
```

**可以看到，在使用上，LinkedHashMap和HashMap的区别就是LinkedHashMap是有序的。** 上面这个例子是根据插入顺序排序，此外，LinkedHashMap还有一个参数决定**是否在此基础上再根据访问顺序(get,put)排序**,记住，是在插入顺序的基础上再排序，后面看了源码就知道为什么了。

## 5.2. LinkedHashMap继承关系

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

底层使用双向链表存储kv，与HashMap区别是插入的顺序是有序的，可以按照顺序找到

本身没有put方法，使用的是hashmap的，以双链表形式，保存所有数据。

把hashmap中数组构成的桶变成了一个双链表结构，但是hash冲突时还是有问题

LinkedHashMap还有一个参数决定是否在此基础上再根据访问顺序(get,put)排序,记住，是在插入顺序的基础上再排序

其实顺序和存储无关，存储还是hashmap，而插入顺序被保存在了一个Entry<K,V>的节点中，即双向链表，有前后关系了

用迭代器访问时，使用linkedhashmap自己实现的迭代器，直接按照双向链表顺序输出即可。

## 5.3. LinkedHashMap成员属性

```java
private static class Entry<K,V> extends HashMap.Entry<K,V> {
    // These fields comprise the doubly linked list used for iteration.
    Entry<K,V> before, after;

Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
        super(hash, key, value, next);
    }
    ...
}
```

![image-20201026115708145](/images/hashmap09.png)

可以看到继承自HashMap的Entry，并且多了两个指针before和after，这两个指针说白了，就是为了维护双向链表新加的两个指针。 列一下新Entry的所有成员变量吧:

- K key
- V value
- Entry<K, V> next
- int hash
- **Entry before**
- **Entry after**

其中前面四个，是从HashMap.Entry中继承过来的；后面两个，是是LinkedHashMap独有的。不要搞错了next和before、After，next是用于维护HashMap指定table位置上连接的Entry的顺序的，before、After是用于维护Entry插入的先后顺序的(为了维护双向链表)。

## 5.4. LinkedHashMap构造方法

```java
 public LinkedHashMap() {
 2 super();
 3     accessOrder = false;
 4 }
1 public HashMap() {
2     this.loadFactor = DEFAULT_LOAD_FACTOR;
3     threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
4     table = new Entry[DEFAULT_INITIAL_CAPACITY];
5     init();
6 }
 1 void init() {
 2     header = new Entry<K,V>(-1, null, null, null);
 3     header.before = header.after = header;
 4 }
```

这里出现了第一个钩子技术,尽管init()方法定义在HashMap中，但是由于LinkedHashMap重写了init方法，所以根据多态的语法，会调用LinkedHashMap的init方法，该方法初始化了一个**Header**,**这个Header就是双向链表的链表头**。

## 5.5. LinkedHashMap put方法

```java
 1 public V put(K key, V value) {
 2     if (key == null)
 3         return putForNullKey(value);
 4     int hash = hash(key.hashCode());
 5     int i = indexFor(hash, table.length);
 6     for (Entry<K,V> e = table[i]; e != null; e = e.next) {
 7         Object k;
 8         if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
 9             V oldValue = e.value;
10             e.value = value;
11             e.recordAccess(this);
12             return oldValue;
13         }
14     }
15 
16     modCount++;
17     addEntry(hash, key, value, i);
18     return null;
19 }
```

LinkedHashMap中的addEntry(又是一个钩子技术) ：

```java
 1 void addEntry(int hash, K key, V value, int bucketIndex) {
 2     createEntry(hash, key, value, bucketIndex);
 3 
 4     // Remove eldest entry if instructed, else grow capacity if appropriate
 5     Entry<K,V> eldest = header.after;
 6     if (removeEldestEntry(eldest)) {
 7         removeEntryForKey(eldest.key);
 8     } else {
 9         if (size >= threshold)
10             resize(2 * table.length);
11     }
12 }
1 void createEntry(int hash, K key, V value, int bucketIndex) {
2     HashMap.Entry<K,V> old = table[bucketIndex];
3     Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
4     table[bucketIndex] = e;
5     e.addBefore(header);
6     size++;
7 }
private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}
```

好了，addEntry先把数据加到HashMap中的结构中(数组+单向链表),然后调用addBefore，这个我就不和大家画图了，**其实就是挪动自己和Header的Before与After成员变量指针把自己加到双向链表的尾巴上。** 同样的，无论put多少次，都会把当前元素加到队列尾巴上。这下大家知道怎么维护这个双向队列的了吧。

上面说了LinkedHashMap在新增数据的时候自动维护了双向列表，这要还要提一下的是LinkedHashMap的另外一个属性，**根据查询顺序排序**,**说白了，就是在get的时候或者put(更新时)把元素丢到双向队列的尾巴上。这样不就排序了吗**？这里涉及到LinkedHashMap的另外一个构造方法:

```java
public LinkedHashMap(int initialCapacity,
         float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

第三个参数，accessOrder为是否开启**查询排序功能的开关**，默认为False。如果想开启那么必须调用这个构造方法。 然后看下get和put(更新操作)时是如何维护这个队列的。

## 5.6. LinkedHashMap get方法

```java
public V get(Object key) {
    Entry<K,V> e = (Entry<K,V>)getEntry(key);
    if (e == null)
        return null;
    e.recordAccess(this);
    return e.value;
}
void recordAccess(HashMap<K,V> m) {
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
    if (lm.accessOrder) {
        lm.modCount++;
        remove();
        addBefore(lm.header);
    }
}
private void remove() {
    before.after = after;
    after.before = before;
}
private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}
```

看到每次recordAccess的时候做了两件事情：

1. **把待移动的Entry的前后Entry相连**
2. **把待移动的Entry移动到尾部**

当然，这一切都是基于accessOrder=true的情况下。 假设现在我们开启了accessOrder，然后调用get("111");看下是如何操作的:



## 5.7. 利用LinkedHashMap实现LRU缓存

**LRU即Least Recently Used，最近最少使用，也就是说，当缓存满了，会优先淘汰那些最近最不常访问的数据**。我们的LinkedHashMap正好满足这个特性，为什么呢？当我们开启accessOrder为true时，**最新访问(get或者put(更新操作))的数据会被丢到队列的尾巴处，那么双向队列的头就是最不经常使用的数据了**。比如:

如果有1 2 3这3个Entry，那么访问了1，就把1移到尾部去，即2 3 1。每次访问都把访问的那个数据移到双向队列的尾部去，那么每次要淘汰数据的时候，双向队列最头的那个数据不就是最不常访问的那个数据了吗？换句话说，双向链表最头的那个数据就是要淘汰的数据。

此外，LinkedHashMap还提供了一个方法，这个方法就是为了我们实现LRU缓存而提供的，**removeEldestEntry(Map.Entry eldest) 方法。该方法可以提供在每次添加新条目时移除最旧条目的实现程序，默认返回 false**。

来，给大家一个简陋的LRU缓存:

```java
public class LRUCache extends LinkedHashMap
{
    public LRUCache(int maxSize)
    {
        super(maxSize, 0.75F, true);
        maxElements = maxSize;
    }

    protected boolean removeEldestEntry(java.util.Map.Entry eldest)
    {
        //逻辑很简单，当大小超出了Map的容量，就移除掉双向队列头部的元素，给其他元素腾出点地来。
        return size() > maxElements;
    }

    private static final long serialVersionUID = 1L;
    protected int maxElements;
}
```

# 6. TreeMap

## 6.1. TreeMap是什么

直接基于红黑树实现，完成对key的排序存储，输出也是有序的，非线程安全
因为对key排序，所以不需要hash解决冲突，直接存储即可。

## 6.2. TreeMap继承关系

```java
public class TreeMap<K,V> extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

从类的定义来看，HashMap和TreeMap都继承自AbstractMap，不同的是HashMap实现的是Map接口，而TreeMap实现的是NavigableMap接口。NavigableMap是SortedMap的一种，实现了对Map中key的排序。

这样两者的第一个区别就出来了，TreeMap是排序的而HashMap不是。

TreeMap是一个泛型类

TreeMap继承自AbstractMap

TreeMap实现NavigableMap接口，表示TreeMap具有方向性，支持导航

TreeMap实现Cloneable接口，表示TreeMap支持克隆

TreeMap实现java.io.Serializable接口，表示TreeMap支持序列化

## 6.3. TreeMap成员属性

```java
//比较器，TreeMap的顺序由比较器决定，如果比较器为空，顺序由key自带的比较器决定
private final Comparator<? super K> comparator;
//根节点，Entry是一个红黑树结构
private transient Entry<K,V> root;
//节点的数量
private transient int size = 0;
//修改统计
private transient int modCount = 0;
//缓存键值对集合
private transient EntrySet entrySet;
//缓存key的Set集合
private transient KeySet<K> navigableKeySet;
private transient NavigableMap<K,V> descendingMap;
```

从字段属性中可以看出

- TreeMap是一个红黑树结构
- TreeMap保存了根节点root

TreeMap和HashMap不同的是，TreeMap的底层是一个Entry root。

## 6.4. TreeMap构造方法

```java
//默认的构造方法
public TreeMap() {
    	//比较器默认为null
        comparator = null;
    }
//传入一个比价器对象
public TreeMap(Comparator<? super K> comparator) {
    	//设置比较器
        this.comparator = comparator;
    }
//传入一个Map对象
public TreeMap(Map<? extends K, ? extends V> m) {
    	//比较器设为null
        comparator = null;
    	//添加Map对象的数据
        putAll(m);
    }
//传入一个SortedMap对象
public TreeMap(SortedMap<K, ? extends V> m) {
    	//把SortedMap的比较器赋值给当前的比较器
        comparator = m.comparator();
        try {
            //添加SortedMap对象的数据
            buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
        } catch (java.io.IOException cannotHappen) {
        } catch (ClassNotFoundException cannotHappen) {
        }
    }

```

从构造方法中可以看出

- TreeMap默认比较器为null，实质上使用的是Key自带的比较器，如果默认比较器为空，Key的对象必须实现Comparable接口
- TreeMap可以指定比较器进行初始化
- TreeMap可以接收Map对象来初始化
- TreeMap可以接收SortedMap对象来初始化

## 6.5. TreeMap put方法

```java
//添加键值对元素
public V put(K key, V value) {
    //获取根节点副本
    Entry<K,V> t = root;
    if (t == null) {
        //如果根节点为null
        //类型检查和空检查，如果当前比较器为null，key的对象必须实现Comparable接口
        compare(key, key); // type (and possibly null) check
        //根据key和value创建新的节点，并把创建的新节点作为根节点
        root = new Entry<>(key, value, null);
        //把元素数量置为1
        size = 1;
        //修改统计加1
        modCount++;
        //返回null
        return null;
    }
    //根节点不为null的情况
    int cmp;
    //parent 插入节点的父节点
    Entry<K,V> parent;
    // split comparator and comparable paths
    //获取当前的比较器
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        //如果存在比较器
        //while循环遍历查找
        do {
            //把parent指向当前节点
            parent = t;
            //表当前节点的key和传入的key
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                //如果传入的key小于当前节点的key，往左查找
                //把当前节点指向左子节点
                t = t.left;
            else if (cmp > 0)
                //如果传入的key大于当前节点的key，往右查找
                //把当前节点指向右子节点
                t = t.right;
            else
                //如果查找到了，把当前节点的值替换为传入的值
                return t.setValue(value);
        } while (t != null);
    }
    //不存在比较器，使用key的比较器
    else {
        if (key == null)
            //如果传入key为空，抛出异常
            throw new NullPointerException();
        //强转key
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        //while循环遍历查找
        do {
            //把parent指向当前节点
            parent = t;
            //使用key自带的比较器进行比较
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                //如果传入的key小于当前节点的key，往左查找
                //把当前节点指向左子节点
                t = t.left;
            else if (cmp > 0)
                //如果传入的key大于当前节点的key，往右查找
                //把当前节点指向右子节点
                t = t.right;
            else
                //如果查找到了，把当前节点的值替换为传入的值
                return t.setValue(value);
        } while (t != null);
    }
    //把最后遍历的节点作为父节点，创建新的节点
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        //如果插入的key比parent的key小，新的节点作为左子节点
        parent.left = e;
    else
        //如果插入的key比parent的key大，新的节点作为右子节点
        parent.right = e;
    //插入过后进行平衡操作
    fixAfterInsertion(e);
    //元素数量加1
    size++;
    //修改统计加1
    modCount++;
    //返回null
    return null;
}
```

## 6.6. TreeMap get方法

```java
//根据传入的key查找值
public V get(Object key) {
    //通过getEntry来查找对应key的节点
    Entry<K,V> p = getEntry(key);
    //如果节点不存在返回null，存在返回节点的value
    return (p==null ? null : p.value);
}
//根据传入的key获取对应的节点
final Entry<K,V> getEntry(Object key) {
    if (comparator != null)
        //如果存在比较器，调用getEntryUsingComparator方法来查找节点
        return getEntryUsingComparator(key);
    //检查key的合法性
    if (key == null)
        throw new NullPointerException();
    //key的对象必须实现Comparable接口
    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key;
    //获取根节点的副本
    Entry<K,V> p = root;
    //通过while循环查找
    while (p != null) {
        //key进行比较
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            //如果传入的key比当前节点的key小，往左边查找
            //把当前节点指向左子节点
            p = p.left;
        else if (cmp > 0)
            //如果传入的key比当前节点的key大，往右边查找
            //把当前节点指向右子节点
            p = p.right;
        else
            //查找成功，返回当前节点
            return p;
    }
    //没找到，返回null
    return null;
}
```

源码 https://juejin.im/post/5e621592f265da573d61b1fb

## 6.7. TreeMap 与 HashMap 区别

### 6.7.1 Null值的区别

HashMap可以允许一个null key和多个null value。而TreeMap不允许null key，但是可以允许多个null value。

```java
@Test
public void withNull() {
    Map<String, String> hashmap = new HashMap<>();
    hashmap.put(null, null);
    log.info("{}",hashmap);
}
@Test
public void withNull() {
    Map<String, String> hashmap = new TreeMap<>();
    hashmap.put(null, null);
    log.info("{}",hashmap);
}
```

### 6.7.2 性能区别

HashMap的底层是Array，所以HashMap在添加，查找，删除等方法上面速度会非常快。而TreeMap的底层是一个Tree结构，所以速度会比较慢。

另外HashMap因为要保存一个Array，所以会造成空间的浪费，而TreeMap只保存要保持的节点，所以占用的空间比较小。

HashMap如果出现hash冲突的话，效率会变差，不过在java 8进行TreeNode转换之后，效率有很大的提升。

TreeMap在添加和删除节点的时候会进行重排序，会对性能有所影响。

```java
public class A06TreeMapTest {
    private static final String[] chars = "A B C D E F G H I J K L M N O P Q R S T U V W X Y Z".split(" ");

    public static void main(String[] args) {
        TreeMap<Integer, String> treeMap = new TreeMap<>();
        for (int i = 0; i < chars.length; i++) {
            treeMap.put(i, chars[i]);
        }

//       key 不能为 null，因为红黑树要比较key大小，为null怎么比较？
//        treeMap.put(null, "2");
//       但 value 可以为null
        treeMap.put(2, null);

        System.out.println(treeMap);
        Integer low = treeMap.firstKey();
        Integer high = treeMap.lastKey();
        System.out.println(low);
        System.out.println(high);
        Iterator<Integer> it = treeMap.keySet().iterator();
        for (int i = 0; i <= 6; i++) {
            if (i == 3) { low = it.next(); }
            if (i == 6) { high = it.next(); } else { it.next(); }
        }
        System.out.println(low);
        System.out.println(high);
        System.out.println(treeMap.subMap(low, high));
        System.out.println(treeMap.headMap(high));
        System.out.println(treeMap.tailMap(low));
    }
}
```

# 7. Set

## 7.1. HashSet

hashset底层是hashmap，是无序散列的，允许null，非线程安全
HashSet 的核心，通过维护一个 HashMap 实体来实现 HashSet 方法
private transient HashMap<E,Object> map;
PRESENT 是用于关联 map 中当前操作元素的一个虚拟值
`private static final Object PRESENT = new Object();`

## 7.2. LinkedHashSet

hashset是无序，散列的，而linkedhashset是按照输入顺序保存的

非线程安全，调用hashmap的构造方法实例化，底层是双链表存储插入的节点的次序



## 7.3. TreeSet

treeset底层基于treemap实现，是元素有序的，把set中的元素当做key存在map中

因为对key排序，所以不能加入null值

# 8. ArrayDeque

