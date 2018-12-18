1 系统定义

1.1设计实现的功能  
（1）小车自动走迷宫  
（2）手机端远程控制，实现小车的走迷宫、前进、后退、左转、右转动作控制。  

1.2可行性分析  
1.2.1技术可行性  
（1）小车走迷宫需要设计迷宫算法：可以选择沿右墙走、两边距离局部最优算法。  
（2）用c/c++语言编写安卓手机App可以使用Qt开发环境，Qt5.11支持安卓4.2以上系统。  

1.3需求分析  
1.3.1功能性需求分析  
（1）走迷宫功能，走迷宫过程中不能碰壁。  
（2）手机App包括服务器连接，服务器断开，前进，后退，左转，右转，开始走迷宫，停止几个按钮，以及两个文本框分别用来输入IP和端口号。  
（3）按下按键，小车动作；松开按键，小车停止。  
1.3.2非功能性需求分析  
（1）性能  
小车以尽量少的时间走出迷宫，并且不碰壁；手机控制准确灵敏无延迟  
（2）远程通讯  
通信方式：小车服务端和手机客户端采用wifi连接；  
距离：大于5米；  
仅允许一个远程控制终端。  
（3）电源，可供小车运行至少一个小时。  

2 系统总体设计  
2.1 总体设计方案的确定  
让小车与手机在同一个局域网内，利用TCP传输（socke编程），让客户端（手机）给服务、端（树莓派）发送信息，以达到控制小车的作用。小车移动用到四个直流减速电机和L298N电机控制模块，电机正反转即可控制前进后退，原地左转右转；位于前、左、右三个方向的超声波测距传感器用于判断三个方向的障碍物距离。  
确定通信协议，手机发送命令，树莓派接收并解析命令。  
2.2 软硬件功能划分  
硬件部分：四个直流减速电机用于小车的前进后退和转向功能；三个超声波测距模块用于用于检测障碍物距离，两个光传感器用于近距离检测障碍物，WiFi模块用于和电脑与手机客户端连接。  
软件部分：迷宫程序设计，实现小车自动走迷宫；手机App设计用于远程遥控。  
2.3 硬件体系架构设计  
如下图所示：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/1.PNG)
2.4 软件体系架构设计  
（1）操作系统选择：小车基于Linux操作系统，采用c/c++编程。  
（2）开发环境：手机App编程使用Qt开发环境。  
（3）软件体系结构设计  
手机客户端向树莓派小车发送命令，小车根据接收的命令进行相应的动作：前进、后退、左转、右转、走迷宫、停止。   
软件体系结构图如下图所示：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/2.PNG)

3软件详细设计    
3.1  QT on Android 开发环境的搭建  
使用的QT版本QT5.11：  
	http://download.qt.io/archive/qt/  
	安装程序：qt-opensource-windows-x86-5.11.2  
	安装组件选择：  
Android 开发环境的搭建  
在QT 5.11“选项->设备->Android”配置中，点击绿色的下载箭头即可跳转到下载页面：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/3.png)    
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/4.png)  

1.JDK的安装：  
https://www.oracle.com/technetwork/java/javase/downloads/index.html  
所用JDK版本：jdk-8u191-windows-x64  
环境变量设置：  
变量名：Path  
变量值：;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin  

变量名：JAVA_HOME  
变量值：C:\Program Files\Java\jdk1.8.0_191  

变量名：CLASS_PATH  
变量值：.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar  

最后验证JDK是否已正确安装：JAVAC -version  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/5.png)  

2.Android SDK：  
https://developer.android.com/studio/  
或者：  
https://www.androiddevtools.cn/  
安装后打开SDK Tools，下载必要组件：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/6.png)  
选择需要的API：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/7.png)
