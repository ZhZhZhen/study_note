# epoll
- 6.0前采用epoll+pipe，6.0以后采用epoll+eventfd。epoll是IO多路复用的一种实现方式(另外的方式是select，poll)，IO多路复用系统开销小
- epoll+pipe：1、创建pipe进行管道读写，创建epoll监听该pipe。2、通过epoll_wait开始监听并进入休眠。3、对管道的写入会唤醒时epoll_wait休眠的线程
- epoll+eventfd：1、打开eventfd并返回文件描述符。然后创建epoll并添加描述符为监听对象(epoll_ctl进行添加)。2、使用epoll_wait查询就绪队列是否有需要处理的事件，无则阻塞等待。3、对描述符发起读写就会唤醒epoll_wait等待的线程
- epoll解决了，IO阻塞中多个线程阻塞读(缓存区空的时候)，多个线程阻塞写(缓冲区满的时候)的效率问题。通过epoll_wait阻塞等待去查询就绪队列中是否有要处理的事件，(通过注册输入流输出流的回调来写入队列)，之后循环就绪队列的事件，开启连接并交给其他工作线程处理。这样只需要一个线程阻塞就可以处理多个流的读写等待问题，降低了创建线程的开销。