[https://nodejs.org/en/docs/guides/dont-block-the-event-loop/](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/)

> **Node.js runs JavaScript code in the Event Loop \(initialization and callbacks\), and offers a Worker Pool to handle expensive tasks like file I/O.**

关于Event Loop的一些误解

#### Misconception1 The event loop runs in a separate thread than the user code {#6304}

> **There is a main thread where the JavaScript code of the user \(userland code\) runs in and another one that runs the event loop. Every time an asynchronous operation takes place, the main thread will hand over the work to the event loop thread and once it is done, the event loop thread will ping the main thread to execute a callback.**

#### Reality {#8647}

> **There is only one thread that executes JavaScript code and this is the thread where the event loop is running. The execution of callbacks \(know that every userland code in a running Node.js application is a callback\) is done by the event loop. We will cover that in depth a bit later.**

### Misconception 2: Everything that’s asynchronous is handled by a thread pool {#07f5}

> **Asynchronous operations, like working with the filesystems, doing outbound HTTP requests or talking to databases are always loaded off to a thread pool provided by libuv.**

#### Reality {#d6dc}

> **Libuv by default creates a thread pool with four threads to offload asynchronous work to. Today’s operating systems already provide asynchronous interfaces for many I/O tasks \(**[**e.g. AIO on Linux**](http://man7.org/linux/man-pages/man7/aio.7.html)**\).**
>
> **Whenever possible, libuv will use those asynchronous interfaces, avoiding usage of the thread pool. The same applies to third party subsystems like databases. Here the authors of the driver will rather use the asynchronous interface than utilizing a thread pool.**
>
> **In short: Only if there is no other way, the thread pool will be used for asynchronous I/O.**

### Misconception 3: The event loop is something like a stack or queue {#19be}

#### Misconception {#f686}

> **The event loop continuously traverses a FIFO of asynchronous tasks and executes the callback when a task is completed.**

#### Reality {#10ca}

> **While there are queue-like structures involved, the event loop does not run through and process a stack. The event loop as a process is a set of phases with specific tasks that are processed in a round-robin manner.**



