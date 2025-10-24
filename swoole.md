### swoole 协程与线程差异
* Swoole 协程与线程的交互
* Swoole 协程环境：Swoole 运行在单线程事件循环中，协程是用户态的“虚拟线程”，通过事件循环调度。PHP 的 Swoole 扩展通过 libeio 或 libuv 处理异步 I/O，协程在 I/O 等待
  ![alt text](images/img_1.png "图1")
* ![alt text](images/img_2.png "图2")
* ![alt text](images/img_3.png "图3")
* ![alt text](images/img_4.png "图4")
