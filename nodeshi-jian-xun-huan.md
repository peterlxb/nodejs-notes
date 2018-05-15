Node事件循环

![](/assets/屏幕快照 2018-05-12 下午8.43.19.png)

![](/assets/屏幕快照 2018-05-12 下午8.44.30.png)

### Node Event Loop内部发生的过程

![](/assets/屏幕快照 2018-05-13 上午10.25.54.png)

会执行上面五个步骤。

官网上的关于什么是Event Loop
> The event loop is what allows Node.js to perform non-blocking I/O operations — despite the fact that JavaScript is single-threaded — by offloading operations to the system kernel whenever possible.

现在的内核都是多线程multi-threaded，可以在背后同时执行多个操作。
当其中的一个操作完成，内核告诉Node.js相应的callback就可以加入 poll queue等待最后执行。

官方文档中的图表

![](/assets/屏幕快照 2018-05-13 上午10.28.13.png)[https://github.com/nodejs/node/blob/v6.x/doc/topics/event-loop-timers-and-nexttick.md](https://github.com/nodejs/node/blob/v6.x/doc/topics/event-loop-timers-and-nexttick.md)
