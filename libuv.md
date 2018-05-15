### libuv相关

![](/assets/屏幕快照 2018-05-15 上午9.22.24.png)

> The I/O \(or event\) loop is the central part of libuv.

> **The event loop follows the rather usual single threaded asynchronous I/O approach: all \(network\) I/O is performed on non-blocking sockets which are polled using the best mechanism available on the given platform: epoll on Linux, kqueue on OSX and other BSDs, event ports on SunOS and IOCP on Windows.**

下面的图表显示loop的每个阶段

![](/assets/屏幕快照 2018-05-15 上午9.44.30.png)

## 重点

> **libuv uses a thread pool to make asynchronous file I/O operations possible, but network I/O isalwaysperformed in a single thread, each loop’s thread.**













