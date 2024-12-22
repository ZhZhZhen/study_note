# Android的线程运用
- HandlerThread
    - 继承至Thread，重写了run()，在其中通过Looper.prepare创建Looper，并调用Looper.loop()
    - 提供了quit()用于退出和Handler的获取
- IntentService
    - onCreate()：在onCreate中创建了HandlerThread，并创建ServiceHandler(该handler使用HandlerThread的looper)
    - onStartCommand()：当回调onStartCommand的时候，1、会使用ServiceHandler发送包含Intent和startId的消息。2、ServiceHandler.handleMessage()中，调用IntentService.onHandleIntent，并调用stopSelf(startId)关闭服务(之所以使用startId，是因为短时间可能发送多个消息，使用stopSelf()会直接关闭)
    - onDestroy()：调用了HandlerThread.quit()退出Looper