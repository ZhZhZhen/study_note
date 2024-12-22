# 2 Zygote进程
- 在AndroidRuntime(cpp文件).start()中。1、创建虚拟机，注册JNI。2、使用JNI调用ZygoteInit(Java文件).main()方法
- 在ZygoteInit(Java文件).main()中。1、通过创建ZygoteServer创建服务端Socket。2、使用forkSystemServer()函数启动SystemServer。3、通过ZygoteServer.runSelectLoop()函数在循环中等待ActivityManagerService的请求来创建新的应用程序进程。