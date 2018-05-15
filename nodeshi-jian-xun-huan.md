Node事件循环

![](/assets/屏幕快照 2018-05-12 下午8.44.30.png)

### Node Event Loop内部发生的过程

![](/assets/屏幕快照 2018-05-13 上午10.25.54.png)



官网上的关于什么是Event Loop

> The event loop is what allows Node.js to perform non-blocking I/O operations — despite the fact that JavaScript is single-threaded — by offloading operations to the system kernel whenever possible.

现在的内核都是多线程multi-threaded，可以在背后同时执行多个操作。  
当其中的一个操作完成，内核告诉Node.js相应的callback就可以加入 poll queue等待最后执行。

官方文档中的图表

![](/assets/屏幕快照 2018-05-15 上午11.30.27.png)

大致Event Loop发生上面的步骤，也称阶段。

> _**note: each box will be referred to as a "phase" of the event loop.**_

**Each phase has a FIFO queue of callbacks to execute. \(每个阶段都有一个先进先出的回调函数队列要执行\)**

While each phase is special in its own way, generally, when the event loop enters a given phase, it will perform any operations specific to that phase, then execute callbacks in that phase's queue until the queue has been exhausted or the maximum number of callbacks has executed.

































