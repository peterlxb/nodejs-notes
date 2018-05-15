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

When the queue has been exhausted or the callback limit is reached, the event loop will move to the next phase, and so on.

Since any of these operations may **schedule\(调度\) **\_more \_operations and new events **processed\(处理\)** in the **poll\(轮训\) **phase are queued by the kernel, poll events can be queued while polling events are being processed. As a result, long running callbacks can allow the poll phase to run much longer than a timer's** threshold\(阀值\)**.

## Phases Overview各个阶段预览 {#header-phases-overview}

**timers **: **\(执行由setTimeout和setInterval调度的回调\)**

**pending callbacks **: executes I/O callbacks deferred to the next loop iteration.

**idle, prepare **: only used internally.

**poll **: retrieve\(检索\) new I/O events; execute I/O related callbacks \(almost all with the exception of close callbacks, the ones scheduled by timers, and`setImmediate()`\); node will block here when appropriate.

**check **: `setImmediate()`callbacks are invoked here.

**close callbacks **: some close callbacks, e.g. `socket.on('close', ...)`

### timers {#header-timers}

> A timer specifies the threshold after which a provided callback may be executed rather than the exact time a person wants it to be executed.

timer用来指定在到达指定的阀值时可能执行提供的回调函数而不是希望它执行的精确时间。

> Timers callbacks will run as early as they can be scheduled after the specified amount of time has passed; however, Operating System scheduling or the running of other callbacks may delay them.

定时器的回调在它们指定的时间到达时会尽可能快的去执行。因为操作系统的调度或者要执行其它回调，这个时间可能会延迟。

> Note: Technically, the poll phase controls when timers are executed.

示例。指定了一个100ms后会执行的定时器。然后你的脚本开始异步读取文件，需要95ms。

```JS
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);


// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```
> When the event loop enters the poll phase, it has an empty queue (fs.readFile() has not completed), so it will wait for the number of ms remaining until the soonest timer's threshold is reached. While it is waiting 95 ms pass, fs.readFile() finishes reading the file and its callback which takes 10 ms to complete is added to the poll queue and executed. When the callback finishes, there are no more callbacks in the queue, so the event loop will see that the threshold of the soonest timer has been reached then wrap back to the timers phase to execute the timer's callback. In this example, you will see that the total delay between the timer being scheduled and its callback being executed will be 105ms.

当event loop进入poll阶段，它有一个空的队列(fs.readFile还没有执行完)。所以它会等待最近的定时器的阀值到达。所以等待95ms后，fs.readFile()完成读文件，它的回调就会被放入poll队列中用10ms去执行。当回调执行完，队列中就没有对调存在，然后event loop就会发现最近的定时器的阀值已经到了，然后回到timer阶段执行这个timer的回调函数。所以在示例中timer执行会是105ms.

### poll {#header-poll}

The poll phase has two main functions:
  * Calculating how long it should block and poll for I/O, then
  * Processing events in the poll queue.

> When the event loop enters the poll phase and there are no timers scheduled, one of two things will happen:

* If the poll queue is not empty, the event loop will iterate through its queue of callbacks executing them synchronously until either the queue has been exhausted, or the system-dependent hard limit is reached.

* If the poll queue is empty, one of two more things will happen:
  1. If scripts have been scheduled by setImmediate(), the event loop will end the poll phase and continue to the check phase to execute those scheduled scripts.
  2. If scripts have not been scheduled by setImmediate(), the event loop will wait for callbacks to be added to the queue, then execute them immediately.

> Once the poll queue is empty the event loop will check for timers whose time thresholds have been reached. If one or more timers are ready, the event loop will wrap back to the timers phase to execute those timers' callbacks.

当poll队列空了的时候，event loop 会检查timers中谁的time发着已经到了。如果一个或多个timer的阀值到了，event loop会绕回到timers阶段去执行这些timers的回调函数。
