# Event

除了异步`Server`和`Client`库之外，`Swoole`扩展还提供了直接操作底层`epoll/kqueue`事件循环的接口。可将其他扩展创建的`socket`，`PHP`代码中`stream/socket`扩展创建的`socket`等加入到`Swoole`的`EventLoop`中。

事件优先级
----
1. 通过`Process::signal`设置的信号处理回调函数
2. 通过`Event::defer`设置的延迟执行函数
2. 通过`Timer::tick`和`Timer::after`设置的定时器回调函数
3. 通过`Event::cycle`设置的周期回调函数

新版本
---
在`2.1.2`或`1.10.3`版本中调整了`2`和`3`的顺序，优先执行定时器。

1. 通过`Process::signal`设置的信号处理回调函数
2. 通过`Timer::tick`和`Timer::after`设置的定时器回调函数
3. 通过`Event::defer`设置的延迟执行函数
3. 通过`Event::cycle`设置的周期回调函数
