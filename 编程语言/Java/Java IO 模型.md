## 1、阻塞I/O：（BIO）

应用进程向内核发起 I/O 请求，发起调用的线程一直等待内核返回结果。一次完整的 I/O 请求称为BIO（BlockingIO，阻塞 I/O），所以 BIO 在实现异步操作时，只能使用多线程模型，一个请求对应一个线程。但是，线程的资源是有限且宝贵的，创建过多的线程会增加线程切换的开销。

<div style="width:67%">
<img src="assert/Pasted image 20260906155348.png" alt="" />
</div>

## 2、同步非阻塞I/O（NIO）：

应用进程向内核发起 I/O 请求后不再会同步等待结果，而是会立即返回，通过轮询的方式获取请求结果。NIO 相比 BIO 虽然大幅提升了性能，但是轮询过程中大量的系统调用导致上下文切换开销很大。所以，单独使用非阻塞 I/O 时 网络传输 效率并不高，而且随着并发量的提升，非阻塞 I/O 会存在严重的性能浪费。


<div style="width:67%">
<img src="assert/Pasted image 20260906155443.png" alt="" />
</div>


## 3、多路复用I/O（select和poll）：
多路复用实现了一个线程处理多个 I/O 句柄的操作。

多路指的是多个数据通道，复用指的是使用一个或多个固定线程来处理每一个 Socket。select、poll、epoll 都是 I/O 多路复用的具体实现，线程一次 select 调用可以获取内核态中多个数据通道的数据状态。

其中，select只负责等，recvfrom只负责拷贝，阻塞IO中可以对多个文件描述符进行阻塞监听，是一种非常高效的 I/O 模型。

<div style="width:67%">
<img src="assert/Pasted image 20260906155916.png" alt=""/>
</div>