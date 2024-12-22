# 3 SystemServer
- forkSystemServer()方法中。1、会最终调用nativeZygoteInit()去创建Binder线程池用于SystemServer的跨进程通信。2、返回包含SystemServer.main()的Runnable(在run中运行SystemServer.main())。
- SystemServer.main()-> ()：该方法中。1、创建SystemServiceManager去创建、启动、管理各个SystemService。2、使用SystemServiceManager去启动三大类系统服务（BootstrapSystemServices\CoreSystemServices\OtherSystemServices） [SystemServer.run]("http://SystemServer.run")