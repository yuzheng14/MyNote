# Ubisoft Connect失去连接解决办法

原视频地址：[育碧平台失去连接100%解决_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nN411X7XX/)

首先打开服务，有两种方式

- <kbd>win</kbd>+<kbd>R</kbd>打开命令窗口输入`services.msc`
- 打开任务管理器切换到服务选项卡

然后找到`Spooler`服务，停止运行，此时再登录即可

登录成功后别忘了重启`Spooler`服务