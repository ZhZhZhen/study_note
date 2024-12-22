# CountDownLatch
- 基于抽象队列同步器的共享锁接口来实现
- 构造方法：根据参数int来设置同步器的初始state
- countDown()：调用Sync.releaseShared(1)，其Sync.tryReleaseShared()逻辑为通过循环+原子操作维护state-1，在state为0时返回true。
- await()：调用Sync.acquireSharedInterruptibly(1)，其Sync.tryAcquireShared()逻辑为1、判断state是否为0，0则返回1，使得阻塞队列中的节点能够进入同步状态。2、因为tryAcquireShared()返回为1，所以会通过传递机制继续唤醒阻塞队列的后续节点，使得他们也能通过tryAcquireShared()获取同步状态。然后又再次重复该操作唤醒后续节点