# 1.系统定义  
## 1.1.设计实现的功能    
### 1.1.1.小车基本行驶功能   
包括前进、后退、原地转向等。    
### 1.1.2.小车高阶行驶功能    
包括线性动力和线性差速转向等。   
### 1.1.3.迷宫寻路功能   
在小车基本行驶功能的基础上，配合超声波测距仪，实现小车在迷宫内的自动寻路，达到走出迷宫的目的。    
### 1.1.4.（跨平台）远程遥控功能    
利用Windows 10或者Windows 10 Mobile的跨平台UWP客户端程序配合小车基本、高阶行驶行驶功能，实现对小车的远程控制，包括：    
控制迷宫模式的启停；    
重力感应控制：利用手机内部加速度计控制线性动力和线性差速转向，实现任意角度可变动力行驶；  
按键控制：触摸/键盘按键控制前进、后退、原地转向；  
模拟摇杆：模拟摇杆控制线性动力和线性差速转向，实现任意角度可变动力行驶；  
## 1.2.可行性分析  
采用了功能、性能强的大树莓派开发板，虽然只有一个GPIO引脚可以输出PWM调制波，但借助强大的wiringPi树莓派IO控制库，可以实现其他引脚的软PWM输出，从而配合电机驱动芯片，达到控制4个直流电机的目的。  
### 1.2.1.基本行驶功能  
基本行驶功能可以直接将两侧电机设置成同时前进、同时后退、分别前进后退，来实现。  
### 1.2.2.高阶行驶功能  
由于没有类似汽车的偏转轮转向系统，故采用差速转向。  
差速转向原理：两侧轮速不同。  
通过调整每个引脚PWM调制波的占空比来控制动力和差速比，以实现高阶行驶功能。  
### 1.2.3.迷宫寻路功能  
实验平台提供前、左、右三个超声波测距模块，对于各处均为直角墙体的迷宫可以较容易地实现迷宫寻路功能。只需在行驶过程中根据超声波传感器数据调整行驶方向，再配合一定的寻路逻辑，即可实现。    
### 1.2.4.（跨平台）远程遥控功能    
Linux、Android、iOS、Windows等平台都可以提供Socket，Bluetooth等远程连接方式，这无疑给本设计的实现提供了非常大的便利性以及普适性。   
# 2.系统总体设计  
## 2.1.总体设计方案的确定  
### 2.1.1.硬件平台  
小车端：采用青科智能小车硬件平台，包括树莓派3B开发板、一系列传感器、控制芯片、锂电池以及直流电机。  
控制端：安装有Windows 10操作系统的PC机、Windows Phone、平板电脑、Hololens或者电视机。  
### 2.1.2.软件平台  
小车端：采用Linux系统以及相应编译工具  
控制端：采用Windows 10或Windows 10 Mobile操作系统，并使用Visual Studio IDE进行开发。  
## 2.2.软硬件功能划分  
### 2.2.1.硬件功能  
超声波发射、接收  
直流电机控制  
检测加速度值（手机）  
远程控制输入，包括触摸或按键  
提供网络物理接口（WLAN）  
### 2.2.2.软件功能  
利用超声波模块测距  
利用PWM调制波控制直流电机  
解析三轴加速度值并量化为、打包控制信号  
将滑动触摸量化、打包为控制信号  
将按键、按钮操作打包为控制信号  
建立Socket连接  
发送、接受以及解析IP数据包  
## 2.3.硬件体系架构设计  
青科智能小车主要由一块树莓派核心控制板、四个直流减速电机、两节锂电池、三个超声波传感器组成。  

## 2.4.软件体系架构设计    
### 2.4.1.小车端  
采用C语言，结合wiringPi控制函数库、pthread线程函数库等实现软件功能，包括：  
初始化  
控制函数定义  
侦听Socket连接  
读取数据包  
解析数据包  
迷宫寻路线程  
测距回传线程  
### 2.4.2.控制端  
采用C#语言，结合.Net Framework的UWP应用实现，主要包括以下功能：  
初始化  
解析键盘按键（PC）、触摸（手机，包括触摸按键和模拟摇杆）  
解析三轴加速度计数据（手机）  
建立Socket连接  
打包、发送数据包  
读取数据包  
解析、回显数据  
UWP 应用：（Universal Windows Platform, 通用Windows平台）  
能够在运行Windows10的所有设备上使用常见的API。  
可以使用设备的特定功能并让UI适应不同的设备屏幕尺寸、分辨率和DPI。  
可使用C#、C++、Visual Basic和Javascript编程。对于UI，使用XAML、HTML或DirectX。  
# 3.系统详细设计   
## 3.1.硬件详细设计    
核心控制板：青科智能车核心控制板采用树莓派三代B型主板，速度更快、性能更强大，是为学生计算机编程教育而设计，只有信用卡大小的卡片式电脑，其系统基于Linux。  
核心控制板参数配置：  
1.2 GHz 四核Broadcom BCM2837 64 位ARMv8 处理器  
板载BCM43143 WiFi  
板载低功耗蓝牙4.1 适配器(BLE)  
1 GB RAM  
4 个USB 2 端口  
微型SD 端口，用于存储操作系统及数据  
CSI 摄像头端口用于连接树莓派摄像头   
4个1:48抗干扰直流减速电机（工作电压3-6V）  
L298N电机驱动芯片(可直接驱动智能车底盘四个电机并可提供PWM使能信号)  
电源：  
LM2596S开关电源稳压电路(支持6~12V宽电压输入，5V输出)  
1个船型电源控制开关(可对整个系统起到很好的电源管理作用)  
2节2500mA容量18650电池  
超声波传感器：  
电压：DC 5V；静态电流：小于2 mA  
感应角度：不大于15 度；探测距离：2 cm-450 cm  
高精度：可达0.3 cm  
## 3.2.软件详细设计  
### 3.2.1.小车端  
程序结构图如下：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/58.png)   
程序流程图如下：
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/59.png)   
#### 3.2.1.0.全局变量和编译预处理指令  
#define Front	28 //前方超声波模块Trig引脚  
#define Left	24 //左方超声波模块Trig引脚  
#define Right	21 //右方超声波模块Trig引脚  
int maze, ret, t_left, t_right;  
上述全局变量分别为：迷宫寻路使能信号，距离回传使能信号，左转时间和右转时间。  
左转时间和右转时间分别控制迷宫寻路模式中的直角左右转弯时间，由于小车左右电机存在工艺或者供电偏差，动力会有些许差异，因此通过这两个变量进行校正。  
#### 3.2.1.1.初始化函数void ultraInit(void)  
```
wiringPiSetup();	//使用wiringPi引脚编号
GAS = 0; 			//将定义的全局变量初始化
HEADING = 0;
maze = 0;
ret = 0;
t_left = t_right = 530; 
pinMode(1, OUTPUT);	//配置各引脚IO模式
pinMode(4, OUTPUT);
pinMode(5, OUTPUT);
pinMode(6, OUTPUT);
pinMode(Front + 1, INPUT);
pinMode(Front, OUTPUT);
pinMode(Left + 1, INPUT);
pinMode(Left, OUTPUT);
pinMode(Right + 1, INPUT);
pinMode(Right, OUTPUT);
softPwmCreate(1, 1, 500);//定义1、4、5、6引脚为软件PWM输出引脚
softPwmCreate(4, 1, 500);//并设置PWM值上限为500，初始值为1
softPwmCreate(5, 1, 500);
softPwmCreate(6, 1, 500);
```
#### 3.2.1.2.超声波测距函数float disMeasure(int a)  
```
digitalWrite(a, LOW);
delayMicroseconds(2);
digitalWrite(a, HIGH);
delayMicroseconds(10);	  //发出超声波脉冲
digitalWrite(a, LOW);
while (!(digitalRead(a + 1) == 1));
gettimeofday(&tv1, NULL);		   //获取当前时间
while (!(digitalRead(a + 1) == 0));
gettimeofday(&tv2, NULL);		   //获取当前时间
start = tv1.tv_sec * 1000000 + tv1.tv_usec;   //微秒级的时间
stop = tv2.tv_sec * 1000000 + tv2.tv_usec;
dis = (float)(stop - start) / 1000000 * 34000 / 2;  //求出距离
return dis;
```
#### 3.2.1.3.基础行驶功能函数   
包括前进、后退、原地左转、原地右转、原地左转（保持）、原地右转（保持）、停止等函数。其中后退和左转/右转（保持）将锁定状态并持续一定时间，时间可以通过参数控制。   
#### 3.2.1.4.高阶行驶功能函数void go(double gas, float turn)   
```
{
	if (gas > 0)
	{
		softPwmWrite(1, 500 * gas*turn * 2);
		softPwmWrite(4, 0);
		softPwmWrite(5, 500 * gas* (1 - turn) * 2);
		softPwmWrite(6, 0);
	}
	else
	{
		softPwmWrite(1, 0);
		softPwmWrite(4, 500 * (-gas)*turn * 2);
		softPwmWrite(5, 0);
		softPwmWrite(6, 500 * (-gas)* (1 - turn) * 2);
	}
}
```
参数gas的取值范围[-1，1]，turn的取值范围[0,1]，两者精度均为0.1。  
通过分割gas正负，确保行驶、转向逻辑正确。  
此函数主要用于远程遥控，实现线性动力控制和线性差速转向控制。  
#### 3.2.1.5.测距回传函数void *RetDis(void *args)  

#####测距回传流程图如下：
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/60.)   
```
while (ret)
{
	int disF, disL, disR;
	char sendbuff[12];
	sendbuff[0] = 'F';
	disF = disMeasure(Front);
	sprintf(&sendbuff[1], "%03d", disF);
	sendbuff[4] = 'L';
	disL = disMeasure(Left);
	sprintf(&sendbuff[5], "%03d", disL);
	sendbuff[8] = 'R';
	disR = disMeasure(Right);
	sprintf(&sendbuff[9], "%03d", disR);
	send(snd_fd, sendbuff, 12, 0);
	delay(100);
}
```
在新线程中每隔100ms测距一次并将结果通过Socket连接回传到控制端，精度为1。  
通过线程取消函数pthread_cancel()可以直接终止。  
#### 3.2.1.6.迷宫寻路函数 void *MazeRunner(void *args)  

迷宫寻路流程图如下：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/61.PNG)    
通过线程取消函数pthread_cancel()可以直接终止。 
```
while (maze) 
{
	disF = disMeasure(Front);
	disL = disMeasure(Left);
	disR = disMeasure(Right);//测距
	if (disMeasure(Front) > 25)
	{
		run(500);			//若无障碍则前进
	}
	else if (disF <= 25 && disL > 40 && disR < 30)
	{
		brake(100);
		if (disF <= 25 && disL > 40 && disR < 30)
		{
			left_t(t_left + 100);
			brake(100);
			run(300);
			delay(100);
			brake(100);
		}		//前方有障碍， Double Check左侧是否无障碍，若无则左转
	}
	else if (disF <= 25 && disL < disR)
	{
		brake(100);
		if (disF <= 25 && disR > 40 && disL < 30)
		{
			right_t(t_right + 100);
			brake(100);
			run(300);
			delay(100);
			brake(100);
		}		//前方有障碍， Double Check右侧是否无障碍，若无则右转
	}
	else if (disR < 10)
		left_t(100);//行驶过程中，若右侧距墙小于10cm则向左修正
	else if (disL < 10)
		right_t(100); //行驶过程中，若左侧距墙小于10cm则向右修正
	else
		brake(1); //其他未知情况，停车
}
```

#### 3.2.1.7.数据包解析函数void ParseFrame(char *bf)  

数据包全部都是以字符‘2’开头，字符‘#’结尾的，长度8或9字节的字符串指令。  
指令包括控制指令和设置指令两种  
控制指令包括：(‘?’表示不敏感)  
迷宫寻路指令			2CM????#  
线性动力控制指令 		2CG?±x.y#  
线性差速转向控制指令 	2CH?±x.y#  
前进指令 				2CF?????#  
后退指令 				2CB?????#  
左转指令 				2CL?????#  
右转指令 				2CR?????#  
停止指令 				2CS?????#  
设置指令包括：  
设置左转时间			2GL??xyz#  
设置右转时间			2GR??xyz#  
```
if (*(bf) =='2'&&*(bf + 1) =='C' &&*(bf + 2) =='M' && *(bf + 7) =='#')
{
	maze = 1;
	pthread_t mazerunner;
	int ser = pthread_create(&mazerunner, NULL, MazeRunner, NULL);
	if (ser != 0)
		printf("pthread create mazerunner error!\n");
	else
		printf("mazerunner pthread created!/\n");
}
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='G'&& *(bf + 8) =='#')
{
	char temp[4];
	temp[0] = *(bf + 4);
	temp[1] = *(bf + 5);
	temp[2] = *(bf + 6);
	temp[3] = *(bf + 7);
	GAS = atof(temp);
	printf("  GAS:%lf   Heading:%lf\n", GAS, HEADING);

	go(GAS, HEADING);

}
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='H'&& *(bf + 8) =='#')
{
	char temp[4];
	temp[0] = *(bf + 4);
	temp[1] = *(bf + 5);
	temp[2] = *(bf + 6);
	temp[3] = *(bf + 7);
	if (GAS < 0)
		HEADING = (atof(temp) + 1) / 2;
	else
		HEADING = (1 - (atof(temp) + 1) / 2);
	printf("  GAS:%lf   Heading:%lf\n", GAS, HEADING);
	go(GAS, HEADING);
}
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='F'&& *(bf + 8) =='#')
	run(500);
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='B'&& *(bf + 8) =='#')
	back_t(1);
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='R'&& *(bf + 8) =='#')
	right();
	//right_t(t_right + 100);
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='L'&& *(bf + 8) =='#')
	left();
	//left_t(t_left + 100);
else if (*(bf) =='2'&&*(bf + 1) =='C' && *(bf + 2) =='S'&& *(bf + 8) =='#')
{
	maze = 0;
	GAS = 0;
	HEADING = 0.5;
	brake(1);
}
else if (*(bf) =='2'&&*(bf + 1) =='G' && *(bf + 2) =='L'&& *(bf + 8) =='#')
{
	char temp[4];
	temp[0] = *(bf + 4);
	temp[1] = *(bf + 5);
	temp[2] = *(bf + 6);
	temp[3] = *(bf + 7);
	t_left = atoi(temp);
}
else if (*(bf) =='2'&&*(bf + 1) =='G' && *(bf + 2) =='R'&& *(bf + 8) =='#')
{
	char temp[4];
	temp[0] = *(bf + 4);
	temp[1] = *(bf + 5);
	temp[2] = *(bf + 6);
	temp[3] = *(bf + 7);
	t_right = atoi(temp);
}
```
#### 3.2.1.8.main函数  

程序入口函数，包括了变量定义，初始化以及Socket相关逻辑。  
连接建立后，读取控制端发送进来的数据包，调用数据包解析函数进行解析并完成相应的控制功能。  
```
int fd, new_fd, numbytes;
char buff[256];
struct sockaddr_in server_addr;
struct sockaddr_in client_addr;
socklen_t struct_len;
ultraInit();
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(8000);
server_addr.sin_addr.s_addr = INADDR_ANY;
bzero(&(server_addr.sin_zero), 8);
struct_len = sizeof(struct sockaddr_in);
fd = socket(AF_INET, SOCK_STREAM, 0);
Bind(fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
printf("Bind Success!\n");
Listen(fd, 2);
printf("Listening....\n");
printf("Ready for Accept,Waitting...\n");
new_fd = Accept(fd, (struct sockaddr *)&client_addr, &struct_len);
printf("Get the Client.\n");
ret = 1;
pthread_t retdis;
int ser = pthread_create(&retdis, NULL, RetDis, &new_fd);
if (ser != 0)
	printf("pthread create RetDis error!\n");
else
	printf("RetDis pthread created!/\n");
while (1)
{
	if ((numbytes = Read(new_fd, buff, sizeof(buff))) > 0)
	{
		Write(STDOUT_FILENO, buff, numbytes);
		ParseFrame(buff);
	}
	else if ((numbytes = Read(new_fd, buff, sizeof(buff))) <= 0)
	{
		brake(1);
		ret = 0;
		pthread_cancel(retdis);
		printf("Connetion closed!\n");
		close(new_fd); //继续侦听客户端连接
		new_fd = Accept(fd, (struct sockaddr *)&client_addr, &struct_len);
		printf("Get the Client.\n");
		ret = 1;
		pthread_t retdis;
		int ser = pthread_create(&retdis, NULL, RetDis, &new_fd);
		if (ser != 0)
			printf("pthread create RetDis error!\n");
		else
			printf("RetDis pthread created!/\n");
	}
}
close(new_fd);
close(fd);
```
### 3.2.2.控制端    
#### 3.2.2.1.UI设计    
##### 侧边栏打开     
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/62.png)    
##### 侧边栏关闭  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/63.png)    
##### 侧边栏设计    
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/64.png)    
##### 操作区设计    
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/65.png)    
#### 3.2.2.2.代码逻辑设计（仅关键代码） 
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/66.png)  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/67.png)  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/68.png)  
##### 异步建立Socket连接   
```
StreamSocket socket = new StreamSocket();
try
{
    if (adapter == null)
    {
        	await socket.ConnectAsync(hostName, PortNameForConnect.Text);
        NotifyUser("Connected", NotifyType.StatusMessage);
    }
    else
    {
        NotifyUser(
            "Connecting to: " + HostNameForConnect.Text +
            " using network adapter " + adapter.NetworkAdapterId,
            NotifyType.StatusMessage);
        await socket.ConnectAsync(
            hostName,
            PortNameForConnect.Text,
            SocketProtectionLevel.PlainSocket,
            adapter);
        NotifyUser(
            "Connected using network adapter " + adapter.NetworkAdapterId,
            NotifyType.StatusMessage);
    }
    SendBuffer("2CSaaaaa#");
    SendBuffer("2GL" + string.Format("{0,5:0}", sliderL.Value) + '#');
    SendBuffer("2GR" + string.Format("{0,5:0}", sliderR.Value) + '#');
    ReadBuffer();
}
catch (Exception exception)
{
    NotifyUser("Connect failed with error: " + exception.Message, NotifyType.ErrorMessage);
    ConnectSwitch.IsOn = false;
}
```
创建新的StreamSocket对象，异步建立连接，建立连接后发送停止、设置指令并开始读取数据。    
##### 异步接收数据   
```
DataReader reader = new DataReader(socket.InputStream); 
//实例化reader对象，并以StreamSocket的输入流为reader的来源
reader.InputStreamOptions = InputStreamOptions.Partial; 
 //采用异步方式
await reader.LoadAsync(24); 
 //获取一定大小的数据流
string response = reader.ReadString(reader.UnconsumedBufferLength);  
//获取字符串，指定为reader的未读取缓冲区的长度
RecvF.Text = "Front: " + response.Substring(1, 3);
RecvL.Text = "Left: " + response.Substring(5, 3);
RecvR.Text = "Right: " + response.Substring(9, 3);
ReadBuffer();//递归接收
```
##### 异步发送数据   
```
var packetsToSend = new List<IBuffer>();
packetsToSend.Add(
Windows.Security.Cryptography.CryptographicBuffer.ConvertStringToBinary(message, Windows.Security.Cryptography.BinaryStringEncoding.Utf8));
//将要发送的数据打包为写入接口使用的引用字节数组
foreach (IBuffer packet in packetsToSend)
{
    await socket.OutputStream.WriteAsync(packet);//异步写入输出流
}
```
##### 加速度计  
```
private async void ReadingChanged(object sender, AccelerometerReadingChangedEventArgs e)
{
    await Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
    {
        AccelerometerReading reading = e.Reading;
        txtXAxis.Text = string.Format("{0,5:0.0}", reading.AccelerationX);
        txtYAxis.Text = string.Format("{0,5:0.0}", reading.AccelerationY);
        txtZAxis.Text = string.Format("{0,5:0.0}", -reading.AccelerationZ);
    });
}
``` 
当加速度计读数变化时，异步地将读数显示到侧边栏的对应三个文本框内。显示格式为5位，精确到0.1。这样可以简单的滤除干扰，提高性能。  
当显示到文本框的读数发生变化时，将数值打包成命令发送到小车端。  
##### 模拟摇杆   
```
thumbX += e.HorizontalChange * 2;
thumbY += e.VerticalChange * 2;
RodX.Text = string.Format("{0,5:0.0}", Math.Atan(thumbX / thumbY)/1.6);/
RodY.Text = string.Format("{0,5:0.0}", 
-(thumbY / Math.Abs(thumbY)) * 
(Math.Sqrt(thumbX * thumbX + thumbY * thumbY) / 300));
//归一化输出动力值和角度值。
if (thumbX * thumbX + thumbY * thumbY >= 300 * 300)
{
    thumbX = thumbX / (Math.Sqrt(thumbX * thumbX + thumbY * thumbY) / 300);
    thumbY = thumbY / (Math.Sqrt(thumbX * thumbX + thumbY * thumbY) / 300);
	//确保不会将小圆圈拖出大圆圈的范围。
}
RootThumb.Margin = new Thickness(0, 0, -thumbX, -thumbY);//更新坐标
```
# 4.功能测试  
## 4.1.连接、测距回显功能测试   
控制端连接到小车后，位于控制区的三个文本框能够正确显示前、左、后方的距离。如图：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/69.png)   
## 4.2.迷宫寻路测试   
控制端连接到小车后，打开Maze开关，小车启动，用时15秒从起点抵达终点。  
抵达终点后，关闭Maze开关，控制台回显迷宫线程成功退出，小车顺利停下。
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/70.png)   
## 4.3.重力感应控制测试  
手机连接小车后，切换到重力感应控制模式。  
前后绕Y轴转动手机，Z轴数据变化，小车相应做线性动力运动，并于控制台返回相应数值，完全符合预期。如图   
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/71.png)  
左右绕Z轴转动手机，Y轴数据变化，控制台返回正确转向数值，完全符合预期。如图.   
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/72.png)   
自由转动手机，小车按照手机姿态进行行驶，完全符合预期。  
## 4.4.模拟摇杆控制测试  
控制端连接小车后，切换到模拟摇杆控制模式  
自由拖动小圆圈，小车按照模拟摇杆姿态做相应运动，并在控制台回显相应数值。如下图.  
放下小圆圈，小车停止运动。  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/73.png)   
## 4.5.按键控制测试
控制端连接到小车后，切换到按键控制模式  
按下（触摸）按键，小车相应做正确运动  
抬起（触摸）按键，小车停止运动  
 














