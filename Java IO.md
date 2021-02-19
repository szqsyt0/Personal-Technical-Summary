## Java IO

### IO模型 - UNIX IO模型

#### 阻塞式IO

应用程序被阻塞，指导数据复制到应用程序缓冲区内才返回

![img](https://www.pdai.tech/_images/io/java-io-model-0.png)

#### 非阻塞式IO

应用进程执行系统调用之后，内核返回一个错误码，应用进程可以继续执行，但是需要不断的执行系统调用来获知IO是否完成，这种方式称为轮询（polling），由于CPU要处理很多的系统调用，因此这种模型比较低效。

![img](https://www.pdai.tech/_images/io/java-io-model-1.png)

#### IO复用

使用select和poll等待数据，并且可以等待多个socket中任何一个变为可读，这一过程会被阻塞，当某一个socket可读时返回，之后再使用recvfrom把数据从内核复制到进程内。IO复用可以使单个进行具有处理多个IO事件的能力。

如果一个Web服务器没有IO复用，那么每一个socket连接都需要创建一个线程去处理，如果同时有几万个连接，那么就需要创建相同数量的线程。并且相比于多进程和多线程技术，IO复用不需要进程线程创建和切换的开销。

![img](https://www.pdai.tech/_images/io/java-io-model-2.png)

#### 信号驱动式IO(SIGIO)

应用进程使用sigaction系统调用，内核立即返回，应用程序可以继续执行，也就是说等待数据阶段应用进程是非阻塞的，内核在数据到达时向应用进程发送SIGIO信号，应用程序收到之后在信号处理程序中调用recvfrom将数据从内核复制到应用程序中。相比于非阻塞式IO的轮询方式，信号驱动IO的CPU利用率更高。

![img](https://www.pdai.tech/_images/io/java-io-model-3.png)

#### 异步IO(AIO)

进行aio_read系统调用会立即返回，应用程序继续执行，不会阻塞，内核会在所有操作完成后向应用程序发送信号。异步IO和信号驱动IO的区别在于异步IO的信号是通知应用程序IO完成，而信号驱动IO的信息时通知应用程序可以开始IO。

![img](https://www.pdai.tech/_images/io/java-io-model-4.png)

### BIO

#### 基本概念

Block-IO: InputStream和OutStream，Reader和Writer。属于同步阻塞模型

同步阻塞：一个请求占用一个进程处理，先等待数据准备好，然后从内核向进程复制数据，最后处理完数据返回

#### 传统BIO通信方式

以前大多数网络通信方式都是阻塞模式的，即：

- 客户端向服务器端发送请求后，客户端会一直等待，指导服务端返回结果或者网络出现问题
- 服务端当在处理某个客户端A发来的请求时，其他客户端发来的请求会等待，直到客户端A的请求处理完毕

![img](https://www.pdai.tech/_images/io/java-io-bio-1.png)

#### 传统BIO的问题

- 同一时间，服务端只能处理一个客户端的请求，因此在高并发情况下，不能采用BIO
- 客户端需要阻塞等待服务端的返回结果

#### 问题根源

重点问题不是是否使用了多线程，而是为什么accept()、read()方法会被阻塞。

API文档中对serverSocket.accept()方法描述如下：

> Listens for a connection to be made to this socket and accepts it. The method blocks until a connection is made.

阻塞式同步IO的工作原理：

- 服务器线程发起一个accept动作，询问操作系统，是否有新的socket套接字信息从端口X发送过来
- 如果操作系统没有发现有套接字从指定端口X发送过来，那么操作系统就会等待，这样accept方法也会等待。所以accept方法会阻塞的原因是它内部的实现是使用的操作系统级别的同步IO

### NIO

#### 基本概念

NonBlock-IO: Channel、Buffer、Selector。属于IO多路复用的同步非阻塞模型

同步非阻塞：进程先将一个套接字在内核中设置成非阻塞再等待数据准备好，在这个过程中反复轮询内核数据是否准备好，准备好之后处理数据返回

IO多路复用：同步非阻塞的优化版本，区别在于IO多路复用阻塞在select，epoll这样的系统调用之上，而没有阻塞在真正的IO系统调用上。也就是说，轮询机制被优化成通知机制，多个连接共用一个阻塞对象，进程只需要在一个阻塞对象上等待，无需再轮询所有连接

在Java的NIO中，是基于Channel和Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector用于监听多个通道的事件（比如：连接打开、数据到达）。因此，单个线程可以监听多个数据通道，Selector的底层实现是epoll/poll/select的IO多路复用模型，select方法会一直阻塞，指导channel中有事件就绪。

#### IO多路复用

##### 典型多路复用IO实现

目前流程的多路复用IO实现主要包括四种: `select`、`poll`、`epoll`、`kqueue`。下表是他们的一些重要特性的比较:

| IO模型 | 相对性能 | 关键思路         | 操作系统      | JAVA支持情况                                                 |
| ------ | -------- | ---------------- | ------------- | ------------------------------------------------------------ |
| select | 较高     | Reactor          | windows/linux | 支持，Reactor模式（反应器设计模式）。Linux操作系统的kernel2.4内核版本之前默认使用select；windows对同步IO的支持也是select模型 |
| poll   | 较高     | Reactor          | Linux         | Linux下的Java NIO框架，Linux kernel2.6内核版本之前使用poll进行支持，也是使用的Reactor模式 |
| epoll  | 高       | Reactor/Proactor | Linux         | Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持。由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO |
| kqueue | 高       | Proactor         | Linux         | 目前Java版本不支持                                           |

IO多路复用适用于高并发场景，所谓高并发是指1ms内至少同时又上千个连接请求准备好，其他情况下IO多路复用发挥不出其优势。

##### Channel

Channel是一个应用程序和操作系统事件交互、内容传递的渠道。一个通道会有一个专属的文件状态描述符。

Java NIO框架中，自有Channel通道包括：

![img](https://www.pdai.tech/_images/io/java-io-nio-2.png)

所有注册到Selector的通道，只能是继承了SelectableChannel类的子类，如上图所示

- ServerSocketChannel: 应用程序服务器程序的监听通道，只有通过这个通道，应用才能向操作系统注册支持多路复用IO的端口监听，同时支持UDP和TCP协议
- SocketChannel： TCP Socket套接字的监听通道，一个Socket套接字对应了一个客户端IP:端口到服务器IP：端口的通信连接
- DatagramChannel： UDP数据报文的监听通道

##### Buffer

为了保证每个通道的数据读写速度，Java NIO框架为每一种需要支持数据读写的通道集成了Buffer的支持。Buffer有两种工作模式：写模式和读模式。在读模式下，应用程序只能从Buffer读取数据，不能进行写操作。在写模式下，应用程序可以进行读操作，这意味着可能出现脏读，所以一旦要从Buffer读取数据，一定要将Buffer切换到读模式。如下图：

![img](https://www.pdai.tech/_images/io/java-io-nio-3.png)

- position：缓冲区目前在操作的数据库位置
- limit：缓冲区最大可以进行操作的位置。读写状态由该属性控制
- capacity：缓存区的最大容量

##### Selector

- 事件订阅和Channel管理

应用向Selector对象注册需要它关注的Channel，以及具体的某一个Channel会对哪些IO事件感兴趣。Selector也会维护一个注册的Channel数组

- 轮询代理

应用层不再通过阻塞模式或者非阻塞模式直接询问操作系统事件有没有发生，而是Selector代其询问

- 实现不同操作系统的支持

由于多路复用IO需要操作系统进行支持，其特点就是操作系统可以同时扫描同一个端口上不同网络连接的事件，所以作为上层的JVM，需要为不同操作的多路复用IO实现编写不同的代码

##### Java NIO框架简要设计分析

Java通过面向对象来适配不同的操作系统上的多路复用IO技术。Java NIO中对各种多路复用IO的支持，主要的基础是java.nio.channels.spi.SelectorProvider抽象类，其中的几个主要抽象方法包括:

```java
public abstract DatagramChannel openDatagramChannel() // 这个操作系统匹配的UDP 通道实现。 
public abstract AbstractSelector openSelector() // 创建和这个操作系统匹配的NIO选择器，就像上文所述，不同的操作系统，不同的版本所默认支持的NIO模型是不一样的。 
public abstract ServerSocketChannel openServerSocketChannel() // 创建和这个NIO模型匹配的服务器端通道。
public abstract SocketChannel openSocketChannel() // 创建和这个NIO模型匹配的TCP Socket套接字通道(用来反映客户端的TCP连接)
```

##### IO多路复用的优缺点

- 不再使用多线程进行IO处理（包括操作系统内核IO管理模块和应用程序进程），当然在实际业务处理中，还是可以引入线程池技术
- 同一个断开可以处理多种协议
- 操作系统级别的优化：多路复用IO技术可以是操作系统在一个端口上同时接收多个客户端的IO事件。同时具有阻塞式同步IO和非阻塞式同步IO的所有特点，Selector的一部分作用相当于轮询代理器。
- 都是同步IO：BIO、NIO、多路复用IO，这些都是基于操作系统对同步IO的实现。同步IO一句话理解：只有上层（包括上层的某种代理机制）系统询问我是否有某个事件发生了，否则我不会主动告诉上层系统事件发生了。

#### NIO和BIO的区别

1. 通过缓冲区而非流的方式进行数据的交互，流是进行直接的传输没有对数据操作的余地，缓冲区提供了灵活的数据处理方式
2. NIO是非阻塞的，意味着每个socket连接可以让底层操作系统帮我们完成而不需要每次开线程去保持连接，使用selector监听所有channel的状态
3. NIO提供直接内存复制方式，消除了JVM与操作系统之间读写内存的损耗

### AIO

#### 基本概念

Asynchronnous IO：属于事件和回调机制的异步非阻塞模型

AIO得到结果的方式：

1. 基于回调：实现ComopletionHandler接口，调用时触发回调函数
2. 返回Future：通过isDone()查看是否准备好，通过get()等待返回数据

![img](https://www.pdai.tech/_images/io/java-io-aio-1.png)

和同步IO一样，异步IO也是由操作系统进行支持的。微软的windows系统提供了一种异步IO技术: IOCP(I/O Completion Port，I/O完成端口)；

Linux下由于没有这种异步IO技术，所以使用的是epoll(上文介绍过的一种多路复用IO技术的实现)对异步IO进行模拟。
