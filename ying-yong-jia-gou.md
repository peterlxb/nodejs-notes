![](/assets/屏幕快照 2018-04-12 下午2.45.11.png)

大致上把应用分为三个层面，前端React App部分，后端Express部分及数据库。

React并不直接与数据库进行对话，而是通过Web服务器也就是Express来获取数据。

Express处理应用逻辑，针对前端发起的请求来向数据库获取数据，通过HTTP返回给React App。

### 了解Node与Express的区别

![](/assets/屏幕快照 2018-04-12 下午2.58.08.png)

##### 当用户发起一个请求时背后的逻辑如下，抛开数据库层

![](/assets/屏幕快照 2018-04-12 下午3.13.13.png)

> **Node首先通过port接收到HTTP请求，传递给Express。所以没有Express,Node也可以做所有的事。**
>
> **Express里面定义不同的route handler来处理不同的请求。**
>
> **一般会有很多不同的handler。比如用来处理authenticate user的。用来处理logging out 的handler。用来处理创建和保存survey的handler。**

### 分析Express route handlers

示例代码

![](/assets/屏幕快照 2018-04-12 下午3.22.08.png)

![](/assets/屏幕快照 2018-04-12 下午3.32.36.png)

不同的方法

![](/assets/屏幕快照 2018-04-12 下午3.34.10.png)





