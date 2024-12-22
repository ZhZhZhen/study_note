# 1 Init进程
- 1.创建一些文件夹并挂载设备
- 2.初始化和启动属性服务  （属性服务用于记录软件的使用信息，用于下一次重启系统时的信息恢复和初始化）
- 3.解析init.rc配置文件并启动zygote进程 （在class_start命令对应的start()函数中，使用fork()创建子进程，并在后续的分支中通过AndroidRunTime(cpp文件)启动Zygote）