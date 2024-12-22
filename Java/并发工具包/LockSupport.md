# LockSupport
- 每个线程都有一个许可值(permit)，permit只有两个值1和0,默认是0。
- park()：只能挂起当前线程，当permit为1时，置为0然后返回；当为0时挂起线程直到permit被别的线程修改为1，然后重新置为0返回。
- parkNanos()：限时挂起
- unPark(Thread)：将目标参数的线程permit设置为1，多次调用不会累加