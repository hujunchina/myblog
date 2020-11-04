---
title: JAVAIO
date: 2020-10-26 13:58:44
categories:
	- 必备知识
tags:
	- 编程	
	- JAVA
---

# 1. BIO 阻塞的IO

## 1.1. IO模型构建

常规IO是对本地本机内存到硬盘的读写操作。如果是远程异构机器如何实现IO读写操作？

需要一个能够网络传输的读写协议，如Socket套接字。Socket套接字就是在进程上监听一个端口来完成数据的发送和接收。socket建立了进程和网卡等硬件的关系。一般的进程无法监听远程数据的，而数据可以通过网卡适配器达到内存或CPU缓存等区域，这时候socket可以绑定端口来把相应的数据传输到该进程对应的内存区域中。

在网络中，socket是指两个连接端。

### 1.1.1 Unix中的Socket

操作系统中也有使用到Socket这个概念用来进行进程间通信，它和通常说的基于TCP/IP的Socket概念十分相似，代表了在操作系统中传输数据的两方，只是它不再基于网络协议，而是操作系统本身的文件系统。

一个socket对应一个文件，比如f5d。

### 1.1.2 网络中的Socket

通常所说的Socket API，是指操作系统中（也可能不是操作系统）提供的对于传输层（TCP/UDP）抽象的接口。现行的Socket API大致都是遵循了BSD Socket规范（包括Windows）。这里称规范其实不太准确，规范其实是POSIX，但BSD Unix中对于Socket的实现被广为使用，所以成为了实际的规范。

如果你要使用HTTP来构建服务，那么就不需要关心Socket，如果你想基于TCP/IP来构建服务，那么Socket可能就是你会接触到的API。

![image-20201026155204539](/images/io01.png)

从上图中可以看到，HTTP是基于传输层的TCP协议的，而Socket API也是，所以只是从使用上说，可以认为Socket和HTTP类似（但一个是成文的互联网协议，一个是一直沿用的一种编程概念），是对于传输层协议的另一种*直接*使用，因为按照设计，网络对用户的接口都应该在应用层。

### 1.1.3 Socket名称的由来

和很多其他Internet上的事物一样，Socket这个名称来自于大名鼎鼎的ARPANET（Advanced Research Projects Agency），早期ARPANET中的Socket指的是一个源或者目的地址——大致就是今天我们所说的IP地址和端口号。最早的时候一个Socket指的是一个40位的数字（RFC33中说明了此用法，但在RFC36中并没有*明确*地说使用40位数字来标识一个地址），其中前32为指向的地址（socket number，大致相当于IP），后8位为发送数据的源（link，大致相当于端口号）。

随着ARPANET的发展，后来（RFC433，Socket Number List）socket number被明确地定义为一个40位的数字，其中后8位被用来制定某个特定的应用使用（比如1是Telnet）。这8位数有很多名字：link、socket name、AEN（another eight number，看到这个名字我也是醉了）。

后来在Internet的规范制定中，才真正的用起了port number这个词。至于为什么端口号是16位的，我想可能有两个原因，一是对于当时的工程师来说，如果每个端口号来标识一个程序，65535个端口号也差不多够用了。二可能是为了对齐吧。

## 1.2. BIO原理

BIO 是指传统阻塞的 Socket 通信，该阻塞体现在两个方面：1. 服务端监听accept时，2. IO读写时

BIO 可以分为客户端和服务端。

服务端：ServerSocket初始化，构造InetSocketAddress地址，bind绑定地址，accept监听。

客户端： Socket初始化（可以直接绑定server地址不用connect）。

注意accept会返回一个socket对象，而每个socket绑定一个流，所以一个accept只能接受一个客户端请求并生成一个socket来操作io。其实socket在linux中就是一个文件，是对文件的读写，来一个建立一个对于的文件。

两端初始化后，要getInputStream，getOutputStream来获得输入输出流才能完成读写。

初始化就是建立了一个socket文件，需要建立通道即IO流才能对文件读写。

### 1.2.2 以下描述使用的是 UNIX/Linux 系统的 API

首先，我们创建 `ServerSocket` 后，内核会创建一个 socket。这个 socket 既可以拿来监听客户连接，也可以连接远端的服务。由于 `ServerSocket` 是用来监听客户连接的，紧接着它就会对内核创建的这个 socket 调用 `listen` 函数。这样一来，这个 socket 就成了所谓的 listening socket，它开始监听客户的连接。

接下来，我们的客户端创建一个 `Socket`，同样的，内核也创建一个 socket 实例。内核创建的这个 socket 跟 `ServerSocket` 一开始创建的那个没有什么区别。不同的是，接下来 `Socket` 会对它执行 `connect`，发起对服务端的连接。前面我们说过，socket API 其实是 TCP 层的封装，所以 `connect` 后，内核会发送一个 `SYN` 给服务端。

现在，我们切换角色到服务端。**服务端的主机在收到这个 `SYN` 后，会创建一个新的 socket**，这个新创建的 socket 跟客户端继续执行三次握手过程。

三次握手完成后，我们执行的 `serverSocket.accept()` 会返回一个 `Socket` 实例，这个 socket 就是上一步内核自动帮我们创建的。

所以说，在一个客户端连接的情况下，其实有 3 个 socket。

关于内核自动创建的这个 socket，还有一个很有意思的地方。它的端口号跟 `ServerSocket` 是一毛一样的。咦！！不是说，一个端口只能绑定一个 socket 吗？其实这个说法并不够准确。

前面我说的TCP 通过端口号来区分数据属于哪个进程的说法，在 socket 的实现里需要改一改。Socket 并不仅仅使用端口号来区别不同的 socket 实例，而是使用 <peer addr:peer port, local addr:local port>这个四元组。

在上面的例子中，我们的 `ServerSocket` 长这样：`<*:*, *:9877>`。意思是，可以接受任何的客户端，和本地任何 IP。

`accept` 返回的 `Socket` 则是这样： `<127.0.0.1:xxxx, 127.0.0.1:9877>`，其中`xxxx` 是客户端的端口号。

如果数据是发送给一个已连接的 socket，内核会找到一个完全匹配的实例，所以数据准确发送给了对端。

如果是客户端要发起连接，这时候只有 `<*:*, *:9877>` 会匹配成功，所以 `SYN` 也准确发送给了监听套接字。

`Socket/ServerSocket` 的区别我们就讲到这里。如果读者觉得不过瘾，可以参考《TCP/IP 详解》卷1、卷2。


## 1.3. BIO操作

```java
static class Client implements Runnable{
    Socket socket = null;
    String name;
    public Client(Socket socket, String name) {
        this.socket = socket;
        this.name = name;
    }
    @Override
    public void run() {
        while(true){
            try {
                String msg = String.format("%s [%s]", new Date().toString(), name);
                socket.getOutputStream().write(msg.getBytes());
                Thread.sleep(10000);
            } catch (IOException | InterruptedException e) {
                System.out.println("客户端写失败");
            }
        }
    }
}

static class Server implements Runnable{
    ServerSocket serverSocket = null;
    public Server(ServerSocket serverSocket){
        this.serverSocket = serverSocket;
    }
    @Override
    public void run() {
        System.out.format("服务端启动成功 %n");
        while(true){
            Socket acceptSocket = null;
            try {
                acceptSocket = serverSocket.accept();
                System.out.format("客户端 %s:%d 连接成功%n", acceptSocket.getLocalAddress().getHostAddress(), acceptSocket.getLocalPort());

                InputStream inputStream = acceptSocket.getInputStream();
                byte[] msg = new byte[1024];
                int len = 0;
                while((len = inputStream.read(msg))!=-1){
                    System.out.println(new Date()+" [server] "+new String(msg));
                }
            } catch (IOException e) {
                System.out.println("服务端错误");
            }
        }
    }
}

public static void main(String[] args) throws IOException {
    //         实例化服务端
    ServerSocket serverSocket = new ServerSocket();
    //        构建地址
    SocketAddress address = new InetSocketAddress("127.0.0.1", 60000);
    //        绑定地址
    serverSocket.bind(address);

    new Thread(new Server(serverSocket)).start();

    Socket socket = new Socket("127.0.0.1", 60000);
    new Thread(new Client(socket, "client01")).start();

    Socket socket2 = new Socket("127.0.0.1", 60000);
    new Thread(new Client(socket2, "client02")).start();
}
```

## 1.4. BIO源码

多个IO读写操作，处理accept问题

从上文的运行结果中我们可以看到，服务器端在启动后，首先需要等待客户端的连接请求（第一次阻塞），

如果没有客户端连接，服务端将一直阻塞等待，然后当客户端连接后，服务器会等待客户端发送数据（第二次阻塞），

如果客户端没有发送数据，那么服务端将会一直阻塞等待客户端发送数据。服务端从启动到收到客户端数据的这个过程，将会有两次阻塞的过程。

这就是BIO的非常重要的一个特点，BIO会产生两次阻塞，第一次在等待连接时阻塞，第二次在等待数据时阻塞。

```java
static class Client implements Runnable{
    Socket socket = null;
    String name;
    public Client(String name) throws IOException {
        this.socket = new Socket("127.0.0.1", 60000);;
        this.name = name;
    }
    @Override
    public void run() {
        while(true){
            try {
                String msg = String.format("%s [%s]", new Date().toString(), name);
                socket.getOutputStream().write(msg.getBytes());
                Thread.sleep(10000);
            } catch (IOException | InterruptedException e) {
                System.out.println("客户端写失败");
            }
        }
    }
}

static class Server implements Runnable{
    ServerSocket serverSocket = null;
    public Server() throws IOException {
        //         实例化服务端
        serverSocket = new ServerSocket();
        //        构建地址
        SocketAddress address = new InetSocketAddress("127.0.0.1", 60000);
        //        绑定地址
        serverSocket.bind(address);
    }
    @Override
    public void run() {
        System.out.format("服务端启动成功 %n");
        Socket[] acceptSocket = new Socket[10];
        int num = 0;
        while(true){
            try {
                acceptSocket[num] = serverSocket.accept();      // 这个也是阻塞状态
                System.out.format("客户端 %s:%d 连接成功 %d %n", acceptSocket[num].getInetAddress().getHostAddress(), acceptSocket[num].getPort(), num);

                InputStream inputStream = acceptSocket[num].getInputStream();

                if(num<10){
                    num++;
                    System.out.println(num);
                }

                byte[] msg = new byte[1024];
                int len = 0;
                //  这个loop已经把线程io阻塞了！，不停的等待读！！
                //  while((len = inputStream.read(msg))!=-1){
                //   System.out.println(new Date()+" [server] "+new String(msg) + num);
                //    }
                new Thread(new AsyncRead(inputStream)).start();

            } catch (IOException e) {
                System.out.println("服务端错误");
            }
        }
    }
}

//    把读写改为多线程的，这样就可以完成读写了
static class AsyncRead implements Runnable{

    InputStream is = null;
    public AsyncRead(InputStream is){
        this.is = is;
    }
    @Override
    public void run() {
        byte[] msg = new byte[1024];
        int len = 0;
        try{
            while((len = is.read(msg))!=-1){
                System.out.println(new Date()+" [server] "+new String(msg));
            }
        } catch (IOException io){
            System.out.println("Thread read error");
        }
    }
}
```

## 1.5. BIO优缺点

简单高效，适合传统CS模式，对于高并发形式无法满足，阻塞且只能连接固定多个客户端。

## 1.6. BIO改进

前面两个案例，一个简单的使用BIO（一个客户端一个服务端通信），还有一个把读写阻塞放在了线程中（一个服务端多个客户端）。

这次使用线程池来管理线程,在jdk1.5之前，nio还没出现的时候，都是使用线程池+队列模拟出伪异步的模式缓解服务器端压力。

正常写服务端和客户端，然后把服务端的socket丢给线程池处理。

```java
final static int PORT = 60000;

public static void Msg(String msg){
    System.out.println(msg);
}
//serversocket服务端
static class SocketServer implements Runnable{
    ServerSocket serverSocket = null;
    HandlerThreadPool threadPool = null;
    Socket socket = null;
    public SocketServer() throws IOException {
        // 服务端启动默认是本地主机，不需要额外指定IP，只需要绑定端口号
        serverSocket = new ServerSocket(PORT);
        threadPool = new HandlerThreadPool(5000, 500);
        Msg("服务端正在监听端口： "+PORT);
    }
    @Override
    public void run() {
        while(true){
            try {
                socket = serverSocket.accept();
                //  需要把socket封装为runnable对象在线程池中执行
                threadPool.execute(new SocketHandler(socket));
                Msg("一个客户端已连接");
            } catch (IOException e) {
                Msg("服务端无法响应请求accept失败");
            }
        }
    }
}

//   线程池，处理accept的客户端
static class HandlerThreadPool{
    ThreadPoolExecutor threadPoolExecutor = null;
    ExecutorService executorService = null;
    public HandlerThreadPool(int maxSize, int queueSize){
        executorService = new ThreadPoolExecutor(
            Runtime.getRuntime().availableProcessors(),   //核心线程数=CPU线程数
            maxSize,        // 最大线程数 = 核心+非核心的
            120L,   //响应时间长短，120秒后非核心线程如果未运行就结束
            TimeUnit.MINUTES,
            new ArrayBlockingQueue<>(queueSize));  //使用link类型阻塞队列
    }
    //        需要把socket封装为runnable对象在线程池中执行
    public void execute(Runnable task){
        executorService.execute(task);
    }
}

//     需要把socket封装为runnable对象在线程池中执行
static class SocketHandler implements Runnable{
    Socket socket = null;
    BufferedReader in = null;
    PrintWriter out = null;
    public SocketHandler(Socket socket){
        this.socket = socket;
    }
    @Override
    public void run() {
        //            int len = 0;  不用字节流，改用字符流
        //            byte[] msg = new byte[1024];
        try {
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream());
            String msgs = null;
            while(true){
                // Msg("sockethandler");
                //   服务端是readnline按行读取的，所以客户端写入要println！
                msgs = in.readLine();
                Msg(getDate()+msgs);
                if (msgs==null){
                    break;
                }
                //      out.write("Service response");
            }
        } catch (IOException e) {
            Msg("server handler broken");
        }
    }
}

//   客户端
static class SocketClient implements Runnable{
    Socket socket = null;
    String name = null;
    PrintWriter out = null;
    public SocketClient(String name) throws IOException {
        this.name = name;
        socket = new Socket("127.0.0.1", PORT);
        out = new PrintWriter(socket.getOutputStream(), true);
        Msg("客户端"+name+"启动");
    }
    @Override
    public void run() {
        while(true){
            out.println(getDate()+" MSG from "+name);
            //                Msg("客户端已写入数据");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                Msg("client sleep interrupted");
            }
        }
    }
}

public static void main(String[] args) throws IOException, InterruptedException {
    new Thread(new SocketServer()).start();
    for (int i = 0; i < 5000; i++) {
        Thread.sleep(100);
        new Thread(new SocketClient("client"+i)).start();
    }
}

public static String getDate(){
    return new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss]").format(new Date());
}
```

# 2 NIO 同步IO控制

## 2.1. NIO概念

回答NIO，从四点出发：非阻塞、Buffer缓冲区、Channel通道、Selector选择器。

NIO需要解决的最根本的问题就是存在于BIO中的两个阻塞，分别是等待连接时的阻塞和等待数据时的阻塞。

在真实NIO中，并不会在Java层上来进行一个轮询，而是将轮询的这个步骤交给我们的操作系统来进行，他将轮询的那部分代码改为操作系统级别的系统调用（select函数，在linux环境中为epoll），在操作系统级别上调用select函数，主动地去感知有数据的socket。

NIO 的 IO 行为还是同步的,在 IO 操作准备好时，业务线程得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。

测试了一下，读写确实还是单线程的，很不好，而且空轮询bug会使CPU飙升，从1%到40%左右。

NIO以上两点缺点+代码复杂容易出bug，很难用，改用NIO 2.0 即 AIO。

## 2.2. NIO原理

NIO需要解决的**最根本的问题**就是存在于BIO中的两个阻塞，分别是**等待连接时的阻塞**和**等待数据时的阻塞**。

我们需要再老调重谈的一点是，如果单线程服务器**在等待数据时阻塞**，那么第二个连接请求到来时，服务器是**无法响应**的。如果是多线程服务器，那么又会有**为大量空闲请求产生新线程**从而造成线程占用系统资源，**线程浪费**的情况。

那么我们的问题就转移到，**如何让单线程服务器在等待客户端数据到来时，依旧可以接收新的客户端连接请求**。

我们在之前实现了一个使用Java做多个客户端连接轮询的逻辑，但是在真正的NIO源码中其实并不是这么实现的，NIO使用了操作系统底层的轮询系统调用 select/epoll(windows:select,linux:epoll)，那么为什么不直接实现而要去调用系统来做轮询呢？

![image-20201026155401802](/source/images/io02.png)

假设有A、B、C、D、E五个连接同时连接服务器，那么根据我们上文中的设计，程序将会遍历这五个连接，轮询每个连接，获取各自数据准备情况，那么**和我们自己写的程序有什么区别呢**？

首先，我们写的Java程序其本质在轮询每个Socket的时候也需要去调用系统函数，那么轮询一次调用一次，会造成不必要的上下文切换开销。

而Select会将五个请求从用户态空间**全量复制**一份到内核态空间，在内核态空间来判断每个请求是否准备好数据，完全避免频繁的上下文切换。所以效率是比我们直接在应用层写轮询要高的。

如果select没有查询到到有数据的请求，那么将会一直阻塞（是的，select是一个阻塞函数）。如果有一个或者多个请求已经准备好数据了，那么select将会先将有数据的文件描述符**置位**，然后select返回。返回后通过**遍历**查看哪个请求有数据。

**select的缺点**：

1. 底层存储依赖bitmap，处理的请求是有上限的，为1024。
2. 文件描述符是会置位的，所以如果当被置位的文件描述符需要重新使用时，是需要重新赋空值的。
3. fd（文件描述符）从用户态拷贝到内核态仍然有一笔开销。
4. select返回后还要再次遍历，来获知是哪一个请求有数据。

### 2.2.2 poll函数底层逻辑

poll的工作原理和select很像，先来看一段poll内部使用的一个结构体。

```c++
struct pollfd{
    int fd;
    short events;
    short revents;
}
```

poll同样会将所有的请求拷贝到内核态，和select一样，poll同样是一个阻塞函数，当一个或多个请求有数据的时候，也同样会进行置位，但是它置位的是结构体pollfd中的events或者revents置位，而不是对fd本身进行置位。

所以在下一次使用的时候不需要再进行重新赋空值的操作。poll内部存储**不依赖bitmap**，而是使用pollfd**数组**的这样一个数据结构，数组的大小肯定是大于1024的。解决了select 1、2两点的缺点。

## 2.2.3 epoll

epoll是最新的一种多路IO复用的函数。这里只说说它的特点。

epoll和上述两个函数最大的不同是，它的fd是**共享**在用户态和内核态之间的，所以可以不必进行从用户态到内核态的一个拷贝，这样可以节约系统资源；

另外，在select和poll中，如果某个请求的数据已经准备好，它们会将所有的请求都返回，供程序去遍历查看哪个请求存在数据，但是epoll只会返回存在数据的请求。

这是因为epoll在发现某个请求存在数据时，首先会进行一个**重排**操作，将所有有数据的fd放到最前面的位置，然后返回（返回值为存在数据请求的个数N），那么我们的上层程序就可以不必将所有请求都轮询，而是直接遍历epoll返回的前N个请求，这些请求都是有数据的请求。

## 2.3. NIO 实践

```java
final static int PORT = 60000;

static class NIOServer implements Runnable{
    //   1. serverSelector负责轮询是否有新的连接，服务端监测到新的连接之后，不再创建一个新的线程，
    //    // 而是直接将新连接绑定到clientSelector上，这样就不用 IO 模型中 1w 个 while 循环在死等
    //        轮询服务端accept客户端
    Selector selectorServer;
    //        轮询客户端IO读写
    Selector selectorClient;
    //       服务端socket类似ServerSocket
    ServerSocketChannel serverSocketChannel = null;
    //        获得一个socket来操作
    Socket socket = null;

    public NIOServer() throws IOException {
        selectorServer = Selector.open();
        selectorClient = Selector.open();
        //            初始化服务端通道
        serverSocketChannel = ServerSocketChannel.open();
        //            绑定通道内socket端口
        serverSocketChannel.socket().bind(new InetSocketAddress(PORT));
        //            设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        //            注册到选择器上
        serverSocketChannel.register(selectorServer, SelectionKey.OP_ACCEPT);
    }
    public Selector getSelectorClient(){
        System.out.println("服务端启动");
        return selectorClient;
    }
    @Override
    public void run(){
        while(true) {
            // 监测是否有新的连接，这里的1指的是阻塞的时间为 1ms
            try {
                if(selectorServer.select(1) > 0){
                    //                        选择器获得有读写信号的channel
                    Set<SelectionKey> selectionKeys = selectorServer.selectedKeys();
                    for(SelectionKey key : selectionKeys){
                        if( key.isAcceptable()){
                            // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                            SocketChannel socketChannel = ((ServerSocketChannel)key.channel()).accept();
                            socketChannel.configureBlocking(false);
                            socketChannel.register(selectorClient, SelectionKey.OP_READ);
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

static class NIOOperator implements Runnable{
    Selector selectorClient;
    public NIOOperator(Selector selectorClient){
        this.selectorClient = selectorClient;
    }
    @Override
    public void run() {
        //            select返回后还要再次遍历，来获知是哪一个请求有数据。
        while(true){
            try {
                // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为 1ms
                if( selectorClient.select(1) > 0){
                    Set<SelectionKey> selectedKeys = selectorClient.selectedKeys();
                    for(SelectionKey key : selectedKeys){
                        if(key.isReadable()){
                            SocketChannel socketChannel = (SocketChannel) key.channel();
                            //                                缓冲区
                            ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
                            // (3) 面向 Buffer
                            socketChannel.read(buffer);
                            //                                转为写
                            buffer.flip();
                            System.out.println(Charset.defaultCharset().newDecoder().decode(buffer).toString());
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
public static String getDate(){
    return new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss]").format(new Date());
}
public static void Msg(String msg){
    System.out.println(msg);
}
//   客户端
static class SocketClient implements Runnable{
    Socket socket = null;
    String name = null;
    PrintWriter out = null;
    public SocketClient(String name) throws IOException {
        this.name = name;
        socket = new Socket("127.0.0.1", PORT);
        out = new PrintWriter(socket.getOutputStream(), true);
        Msg("客户端"+name+"启动");
    }
    @Override
    public void run() {
        while(true){
            out.println(getDate()+" MSG from "+name);
            //                Msg("客户端已写入数据");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                Msg("client sleep interrupted");
            }
        }
    }
}

public static void main(String[] args) throws IOException {
    NIOServer nioServer = new NIOServer();
    new Thread(nioServer).start();
    new Thread(new NIOOperator(nioServer.getSelectorClient())).start();
    for (int i = 0; i < 5; i++) {
        new Thread(new SocketClient("client"+i)).start();
    }
}
```

## 2.4 NIO总结

### 2.4.1 **Java流概述** 

流stream是数据流动的方式，In表示从源流入到内存中（Java进程中），out表示从内存流出到目标中。In输入有read方法，读取源到内存中，out输出有write方法，从内存写出到目标中。

根据数据的流向可分为输入流和输出流，InputStream，OutputStream，Reader，Writer有4个最基本的流

根据读取的位大小分为字节流和字符流，字节是Stream结尾，字符是XXer结尾，可以使用InputStreamReader，OutputStreamWriter两个类做字节流和字符流的转换。

### 2.4.2 **Java流之BIO** 

BIO即Blocked InputOutput，同步阻塞流，阻塞的输入输出流。特点是其读写会阻塞线程。Linux中应用进程通过调用recvfrom获取磁盘或内存中数据时，内核如果没有准备好数据就不会立即返回，这样该进程就要一直等待，直到内核返回读写结果。这里的阻塞是因为，Java进程线程读写数据时，CPU会停下来等待磁盘读写而造成CPU等待，造成资源浪费。BIO专指Java中基本的流，因为他们都是阻塞的非异步的。

BIO缺点很明显，不适合高并发场景。典型的CPU与外设速度不匹配。

### 2.4.3 Java流之NIO

NIO即Non-blocked InputOutput，同步非阻塞流。

同步非阻塞IO ：同步即Java线程发出读写请求一直等待返回结果才执行下一步，非阻塞即不在让CPU等待了，CPU可以去帮其他线程。虽然CPU资源利用起来了，但需要频繁的切换线程，直到这个线程获得读写资源。也是很低效浪费资源。

NIO支持面向缓冲的|基于通道的 I/O 操作方法。

NIO 提供了与传统 BIO 模型中的socket 和 serverSocket 相对应的 socket-Channel 和 serverSocketChannel两种不同的套接字通道实现。可开发高负载、高并发的应用。

![image-20201026155504994](/images/io03.png)

NIO 底层是通过Selecter线程管理多路复用IO的多个连接，当其中一个channel把数据准备好的时候会发送一个可读事件给selecter线程，然后线程调用socket进行读操作。

![image-20201026155549226](/images/io04.png)

### 2.4.4 NIO与BIO区别 

1.  NIO非阻塞，BIO阻塞
2.  NIO面向缓冲区buffer，BIO面向流，上图所示NIO中的channel准备好数据后都是在buffer中，NIO读写都是基于一个buffer对象。
3.  NIO通过channel双向读写，BIO通过流单向，通道只有和buffer结合才能实现异步非阻塞。
4.  NIO有select选择器线程，BIO没有

- 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据
- 从通道进行数据写入 ：创建一个缓冲区，填充数据并要求通道写入数据

# 3 常规IO类

## 3.1 RandomAccessFile类

文件读写, RandomAccessFile会打开文件从头开始写，然后覆盖文件内容，严格按照字符长度来读写。

用此类可以存储固定长度，并读取固定长度的数据，对空间极大的利用，压缩存储空间。

```java
public static void main(String[] args) throws IOException {
    String filename = "randomaccessfile.txt";
    File file = new File(filename);
    RandomAccessFile rdf = new RandomAccessFile(file, "rw");

    String name = null;
    int age = 0;

    name = "zhangsan"; // 字符串长度为8
    age = 30; // 数字的长度为4
    rdf.writeBytes(name); // 将姓名写入文件之中
    rdf.writeInt(age); // 将年龄写入文件之中

    name = "lisi    "; // 字符串长度为8
    age = 31; // 数字的长度为4
    rdf.writeBytes(name); // 将姓名写入文件之中
    rdf.writeInt(age); // 将年龄写入文件之中

    name = "wangwu  "; // 字符串长度为8
    age = 32; // 数字的长度为4
    rdf.writeBytes(name); // 将姓名写入文件之中
    rdf.writeInt(age); // 将年龄写入文件之中

    rdf.close(); // 关闭

    readFile(filename);
}

public static void readFile(String filename) throws IOException {
    File f = new File(filename);    // 指定要操作的文件
    RandomAccessFile rdf = null;        // 声明RandomAccessFile类的对象
    rdf = new RandomAccessFile(f, "r");// 以只读的方式打开文件
    String name = null;
    int age = 0;
    byte b[] = new byte[8];    // 开辟byte数组
    // 读取第二个人的信息，意味着要空出第一个人的信息
    rdf.skipBytes(12);        // 跳过第一个人的信息
    for (int i = 0; i < b.length; i++) {
        b[i] = rdf.readByte();    // 读取一个字节
    }
    name = new String(b);    // 将读取出来的byte数组变为字符串
    age = rdf.readInt();    // 读取数字
    System.out.println("第二个人的信息 --> 姓名：" + name + "；年龄：" + age);
    // 读取第一个人的信息
    rdf.seek(0);    // 指针回到文件的开头
    for (int i = 0; i < b.length; i++) {
        b[i] = rdf.readByte();    // 读取一个字节
    }
    name = new String(b);    // 将读取出来的byte数组变为字符串
    age = rdf.readInt();    // 读取数字
    System.out.println("第一个人的信息 --> 姓名：" + name + "；年龄：" + age);
    rdf.skipBytes(12);    // 空出第二个人的信息
    for (int i = 0; i < b.length; i++) {
        b[i] = rdf.readByte();    // 读取一个字节
    }
    name = new String(b);    // 将读取出来的byte数组变为字符串
    age = rdf.readInt();    // 读取数字
    System.out.println("第三个人的信息 --> 姓名：" + name + "；年龄：" + age);
    rdf.close();                // 关闭
}
```



## 3.2 输入输出字节流

流分为字节流和字符流两种，有流向就有两个端，这两个端就是内存和硬盘，从内存到硬盘叫输出流，反过来叫输入流。

字节流都是继承inputstream和Outputstream抽象类。

根据流的用途，分为二进制、文件、数据、管道、序列化、过滤。

```java
//    内存字节流，把数据存储到内存中，在内存中存储一些临时信息，所以参数是数组
//    从内存A到内存B
public static void BinaryStream() throws IOException {
    byte[] memData = {1,1,1,1,1,1,1,1,1,1,1,1,1,-1};
    // 不用写入，memData本身就在内存中，只要指定流的入口
    InputStream is = new ByteArrayInputStream(memData);     

    int tag = 0;
    //流循环读取，最后一个是-1标识文件结束，这是流的出口，到tag上了
    while((tag = is.read())!=-1){       
        System.out.println((byte)tag);  // 输出
    }
    is.close();
}
```

```java
//    文件流，对文件读写
public static void FileStream() throws IOException {
    String filename = "randomaccessfile.txt";

    FileInputStream fis = new FileInputStream(filename);
    FileOutputStream fos = new FileOutputStream(filename);

    fos.write("hujun".getBytes());
    fos.close();

    //        int tag = 0;
    //        while((tag=fis.read())!=-1){
    //            System.out.println((char)tag);
    //        }
    //        上面代码，已经把文件指针读到尾了

    File f = new File(filename);
    byte[] ret = new byte[(int) f.length()];
    fis.read(ret);
    System.out.println(new String(ret));
    fis.close();
}
```

```java
//    管道流，管道流的主要作用是可以进行两个线程间的通信
//    如果要进行管道通信，则必须把 PipedOutputStream 连接在 PipedInputStream 上。
//    为此，PipedOutputStream 中提供了 connect() 方法
public static void PipedStream() throws IOException {

    Send sender = new Send();
    Receive receiver = new Receive();

    sender.getConnect(receiver.putPipe());
    new Thread(sender).start();
    new Thread(receiver).start();

    //   每个汉字转为字节是不一样的，有的是3个有的是4个
    System.out.println("息".length()+ " "+ "息".getBytes().length);  // 1 3
    System.out.println("一".length()+ " "+ "一".getBytes().length);  // 1 3
    System.out.println("a".length()+ " "+ "a".getBytes().length);  // 1 1
    System.out.println("这是来自另一个线程的信".getBytes().length);  // 33
    System.out.println("这是来自另一个线程的信息".getBytes().length);  // 36
}
static class Send implements Runnable{
    PipedOutputStream pos = new PipedOutputStream();;

    public void getConnect(PipedInputStream pis) throws IOException {
        pos.connect(pis);
    }

    @Override
    public void run() {
        try {
            pos.write("这是来自另一个线程的信息".getBytes());
            pos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

static class Receive implements Runnable{
    PipedInputStream pis = new PipedInputStream();
    public PipedInputStream putPipe(){
        return pis;
    }
    @Override
    public void run() {
        try {
            // 12个汉字，转为byte是多长？ 一个汉字3个字节，12个一共36个字节,  存储在数组中占了36个字节
            byte[] ret = new byte[36];
            pis.read(ret);
            System.out.println(ret.length+" "+Arrays.toString(ret));
            System.out.println(new String(ret));
            pis.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 3.3 输入输出字符流

### 3.3.1 字符流

字符流有两个顶层抽象类Reader和Writer，派生出字符串、缓冲区、管道、文件、过滤、转换等流。

一个字符等于两个字节,  但是转为byte时是占3个长度。

计算机一次读的字节称为编码单位(英文名叫Code Unit)，也叫码元， byte中汉字占3个码元。

UTF-8中，如果首字节以1110开头，肯定是三字节编码(3个码元)。

字节流在操作时本身不会用到缓冲区（内存），是文件本身直接操作的。

字符流在操作时是使用了缓冲区，通过缓冲区再操作文件。

除了纯文本数据文件使用字符流以外，其他文件类型都应该使用字节流方式。

### 3.3.2 实战

```java
//    文件字符流
public static void FileRW() throws IOException {
    File file = new File("randomaccessfile.txt");
    FileReader fr = new FileReader(file);
    FileWriter fw = new FileWriter(file);

    fw.write("一个字符等于两个字节");
    fw.close();

    char[] ret = new char[10];
    fr.read(ret);
    System.out.println(new String(ret));
    fr.close();
}
//    字节转字符
public static void StreamToChar() throws FileNotFoundException {
    File file = new File("randomaccessfile.txt");
    InputStreamReader isr = new InputStreamReader(new FileInputStream(file));
    OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(file));
}
```

### 3.3.3 文件操作

```java
static class FileOperator{
    private File f = null;
    public FileOperator(File f){
        this.f = f;
    }
    public void creteFile(){
        try {
            f.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public void deleteFile(){
        f.delete();
    }
    public void createFold(){
        if(f.isDirectory()){
            f.mkdir();
        }
    }
    public void listFiles(){
        String str[] = f.list();
        for(String s:str){
            System.out.println(str);
        }
    }
    public void show(){
        System.out.println(File.pathSeparator);
        System.out.println(File.separator);
    }

    public void writeLine(String line) throws IOException {
        OutputStream out = new FileOutputStream(f);
        out.write(line.getBytes());
        out.close();
    }
    public void writeAppendLine(String line) throws IOException {
        OutputStream out = new FileOutputStream(f, true);
        out.write(line.getBytes());
        out.close();
    }
    public String readLine() throws IOException {
        String line = null;
        InputStream input = new FileInputStream(f);
        byte[] b = new byte[1024];
        input.read(b);
        input.close();
        line = new String(b);
        input.close();
        return line;
    }
    public byte[] readBytes() throws IOException {
        byte[] line = new byte[(int)f.length()];
        InputStream input = new FileInputStream(f);
        int i=0;
        while(input.read()!=-1){
            line[i++] = (byte)input.read();
        }
        input.close();
        return line;
    }
    public void writeWord(String line) throws IOException {
        Writer writer = new FileWriter(f);
        writer.write(line);
        writer.append(line);
        writer.close();
    }
    public String readWord() throws IOException {
        String line = null;
        Reader reader = new FileReader(f);
        char c[] = new char[1024];
        int len = reader.read(c);
        reader.close();
        line = new String(c, 0, len);
        return line;
    }
    public void streamWriter(String line) throws IOException {
        Writer out = new OutputStreamWriter(new FileOutputStream(f));
        out.write(line);
    }
}
```

## 3.4 文件通道

文件通道，NIO不仅有网络传输的socket，也有文件传输的通道。

NIO是非阻塞IO，面向Buffer缓冲区，以块为单位传输数据，支持双向读写。

虽然是面向缓冲区的，但是还是面向字节而不是字符。FileChannel是阻塞的。

```java
public static void fastCopy(String filename, String dist) throws IOException {
    FileInputStream fis = new FileInputStream(filename);
    FileChannel fcin = fis.getChannel();

    FileOutputStream fos = new FileOutputStream(dist);
    FileChannel fcout= fis.getChannel();

    // 通道读写需要先设置缓冲区
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
    while (true) {
        /* 从输入通道中读取数据到缓冲区中 */
        int r = fcin.read(buffer);
        /* read() 返回 -1 表示 EOF */
        if (r == -1) {
            break;
        }
        /* 切换读写 */
        buffer.flip();
        /* 把缓冲区的内容写入输出文件中 */
        fcout.write(buffer);
        /* 清空缓冲区 */
        buffer.clear();
    }
}
```

## 3.5 数据流

```java
public static void readData(String file) throws IOException {
    FileInputStream fis = new FileInputStream(file);
    DataInputStream dis = new DataInputStream(fis);
    int i = dis.readInt();
    boolean b = dis.readBoolean();
    char c = dis.readChar();
    //        can not read string
    String str = dis.readUTF();
    //        String str = "can not read string";
    double pi = dis.readDouble();
    System.out.format("%d,%b,%c,%s,%f%n", i, b, c, str, pi);
}
public static void main(String[] args) throws IOException {
    String file ="Base01/src/cn/edu/zju/how2jcn/middle/datastream.txt";
    FileOutputStream fos = new FileOutputStream(file);
    DataOutputStream dos = new DataOutputStream(fos);
    int i = 10;
    boolean b = true;
    char c = '胡';
    String str = "hujun";
    double pi = 3.14;
    dos.writeInt(i);
    dos.writeBoolean(b);
    dos.writeChar(c);
    dos.writeUTF(str);
    dos.writeDouble(pi);
    dos.close();

    readData(file);
}
```

## 3.6 标准Stream写法

```java
public static void GBK2UTF8(String filename) throws IOException {
    BufferedReader br = 
        new BufferedReader(new InputStreamReader(new FileInputStream(filename), Charset.forName("GBK")));
    BufferedWriter bw = 
        new BufferedWriter(new OutputStreamWriter(new FileOutputStream(filename), StandardCharsets.UTF_8));
    String line = null;
    while((line = br.readLine())!=null) {
        bw.write(line);
    }
    br.close();
    bw.close();
}
```

选取流的标准，如果是对纯文本文件如日志等读写，使用字符流。而且要使用buffer缓存。

其他的文件如二进制等要使用字节流，但是也要buffer缓存，就要多做一个转换（字节流转字符流）。

```java
FileInputStream fis = new FileInputStream("readme.txt");
FileOutputStream fos = new FileOutputStream("reademe.txt");
FileReader fr = new FileReader("readme.txt");
FileWriter fw = new FileWriter("readme.txt");
InputStreamReader isr = new InputStreamReader(is);
OutputStreamWriter osw = new OutputStreamWriter(os);
DataInputStream dis = new DataInputStream(fis);
DataOutputStream dos = new DataOutputStream(fos);
ObjectInputStream ois = new ObjectInputStream(is);
ObjectOutputStream oos = new ObjectOutputStream(os);
BufferedReader br = new BufferedReader(isr);
BufferedWriter bw = new BufferedWriter(osw);
PrintStream ps = new PrintStream("readme.txt");
PrintWriter pw = new PrintWriter("readme.txt");
```

