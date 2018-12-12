# AppOnCar
  本目录下将介绍以智能小车为载体开发的App应用，其中不乏一些让人眼前一亮的技术。
  
  其中有基于百度语音控制的作品，
  
  有基于QT5开发的APP，
  
  还有基于微信小程序，
  
  还有的作品引入了重力感应控制技术。
  
  这些在本目录下降详细介绍。
  
  同样的，该小车以 Raspberry 3B 为核心控制单元。车体运行通过超声波避障模块检测识别迷宫，并利用不同的编程语言实现相关程序，实现了智能车自主走迷宫的功能。

该款智能小车主要由一块树莓派核心控制板、四个直流减速电机、两节锂电池、三个超声波传感器、两个不怕光红外传感器和两个底部的光传感器组成。超声波传感器和红外传感器可以实现避障功能，底部的光传感器可以实现循迹功能。核心控制板采用树莓派三代 B 型主板，速度更快、性能更强大，系统基于 Linux，已具备了电脑的所有基本功能，通过编写相应程序便可实现所需的功能。

障碍物检测是智能小车导航研究中很重要的一个部分。在小车实际运行中，传感器相当于小车的“眼睛"，必须得到障碍物及其距离的信息，才能相应的规划自动避障导航算法。超声波测距是通过测量超声波从发射到遇到障物反射到被接收这整个过程中的时间差来确定距离，超声波传感器使用比较方便、信息处理简单、实时性强等优点。

硬件设计：

核心控制板：青科智能车核心控制板采用树莓派三代 B 型主板，速度更快、性能更强大，只有信用卡大小的卡片式电脑，其系统基于 Linux。核心控制板参数配置：

1.2 GHz 四核 Broadcom BCM2837 64 位 ARMv8 处理器

板载 BCM43143 WiFi

板载低功耗蓝牙 4.1 适配器(BLE)

1 GB RAM

4 个 USB 2 端口

微型 SD 端口，用于存储操作系统及数据

CSI 摄像头端口用于连接树莓派摄像头

电机：

L298N 电机驱动芯片(可直接驱动智能车底盘四个电机并可提供PWM 使能信号)

4 个 1:48 抗干扰直流减速电机（工作电压 3-6 V）

电源：

LM2596S 开关电源稳压电路(支持 6~12 V 宽电压输入，5 V输出)

1个船型电源控制开关(可对整个系统起到很好的电源管理作用)

2 节 5000 mA 大容量 26650 电池，支持小车持续运转一个小时

超声波传感器：

电压：DC 5V；静态电流：小于 2 mA

感应角度：不大于 15 度；探测距离：2 cm-450 cm

高精度：可达 0.3 cm
