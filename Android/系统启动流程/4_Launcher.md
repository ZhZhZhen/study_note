# 4 Launcher
- Launcher就是手机的桌面，用于显示其他App的图标，本质也是一个应用
- 在SystemServer其他系统服务后，由ActivityManagerServices.systemReady()启动。Launcher启动后会在其onCreate()函数中获取其他应用的图标信息并显示出来
- Launcher启动其他App的流程（与App自身打开自己的页面流程类似，都经过AMS处理）