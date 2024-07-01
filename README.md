Simple Reactor Implemention
====================
This is a simple implementation of reactor which use epoll as the event multiplexer and the min-heap as the manage container of timed-task. If you want to use it in industrial environment, i think the "ACE" is best solution.

You can compile the code by using the following shell command:

>g++ -o reactor_server reactor_server_test.cc event_demultiplexer.cc reactor.cc global.cc   
>g++ -o reactor_client reactor_client_test.cc event_demultiplexer.cc reactor.cc global.cc   

After that, two executable programs will be made.

You can use them in this way:

>./reactor_server 127.0.0.1 6852   
>./reactor_client 127.0.0.1 6852   

## 参考链接

https://blog.csdn.net/u012398613/article/details/51964897

## 类图

![image](https://github.com/Neojan/SimpleReactorImplemention/assets/13540636/2f2ab4ab-05da-4645-afe8-a3bd9464c752)


## epoll 标志位

- EPOLLIN       连接到达；有数据来临；
- EPOLLOUT      有数据要写
- EPOLLRDHUP    这个好像有些系统检测不到，可以使用EPOLLIN，read返回0，删除掉事件，关闭close(fd);
如果有EPOLLRDHUP，检测它就可以直到是对方关闭；否则就用上面方法。
Stream socket peer closed connection, or shut down writing half
of connection. (This flag is especially useful for writing sim-
ple code to detect peer shutdown when using Edge Triggered moni-
toring.)
- EPOLLPRI      外带数据 There is urgent data available for read(2) operations.

- EPOLLERR      只有采取动作时，才能知道是否对方异常。即对方突然断掉，是不可能
有此事件发生的。只有自己采取动作（当然自己此刻也不知道），read，write时，出EPOLLERR错，说明对方已经异常断开。

- EPOLLONESHOT 标志的目的是在一个事件被触发后，自动从 epoll 的监听列表中移除该事件，这样下次再触发该事件时，epoll 不会再次通知应用程序。这在某些场景下非常有用，比如当应用程序需要频繁地从一个文件描述符读取数据时，但每次读取的数据量并不大，或者当应用程序需要处理一个非常短暂的事件时。

## 水平触发和边沿触发

ev.events = EPOLLIN; // LT
ev.events = EPOLLIN | EPOLLET; // ET

Level-triggered ：水平触发，缺省模式
edge-triggered ：边缘触发

比如redis用LT模式，nginx用ET模式

通知模式：

LT模式时，事件就绪时，假设对事件没做处理，内核会反复通知事件就绪

ET模式时，事件就绪时，假设对事件没做处理，内核不会反复通知事件就绪

事件通知的细节：

1.调用epoll_ctl，ADD或者MOD事件EPOLLIN

LT：如果此时缓存区没有可读数据，则epoll_wait不会返回EPOLLIN，如果此时缓冲区有可读数据，则epoll_wait会持续返回EPOLLIN

ET：如果此时缓存区没有可读数据，则epoll_wait不会返回EPOLLIN，如果此时缓冲区有可读数据，则epoll_wait会返回一次EPOLLIN

2.调用epoll_ctl，ADD或者MOD事件EPOLLOUT

LT：如果不调用epoll_ctl将EPOLLOUT修改为EPOLLIN，则epoll_wait会持续返回EPOLLOUT(前提条件是写缓冲区未满)

ET：epoll_wait只会返回一次EPOLLOUT
