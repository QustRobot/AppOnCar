# 1.系统定义 
## 1.1设计实现的功能   
1.小车走迷宫  
2.方向盘控制  
3.语音控制  
4.路径控制  

## 1.2 可行性分析  

1.所谓迷宫，即存在死路的带有墙壁的复杂的道路的集合。一般迷宫的路径存在不可预知性，即迷宫的路径情况不可知。所以想要小车能从迷宫路口走到迷宫出口，必然  要对迷宫类型进行定义和对小车的行车逻辑进行相应的具体设计。所以我们定义小车所走的迷宫为不存在环路的迷宫，避免小车行走时出现绕环运行的情况。还有迷宫可以 存在死路，那便要求小车能从死胡同中转出来。故可以采用小车贴墙走的方案，因为当小车按贴墙走时，迷宫的死胡同对于小车便成了通路，这样小车便可走出迷宫。当然我们需要定义小车贴哪边的墙行走，我们选择了左边。   
当然，对于迷宫的算法，我们不只设计了以上一种算法，还有相似的居中走法，在进入死胡同时进行180°原地旋转，同时可以选择左路优先和右路优先。从原理上讲，这种算法和上种算法存在一般以上的相同思想，但是由于小车的电机和超声波传感器的不精准问题导致这种算法下的小车转弯的精确性更差，所以不得不舍弃。
最后，还有建立在第二种算法上的可记录型算法，即小车在行走迷宫的同时会记录自身在迷宫的转弯情况，即路径情况。通过对路径的记录和小车的实时行走的情况便可以判断出迷宫的死胡同的存在，并将其从路径记录中删除，同时让小车退回到下一条可行走的迷宫道路中，以此逻辑便可以让小车走出迷宫，并可以得到这个迷宫的正确路径。当小车再次行走这个迷宫时，便可以通过先前记录下迷宫路径记录直接找到正确路径，做出迷宫。   

2.采用Android平台，方向盘UI设定为两个同心圆，一大一小，大的为基盘，小的为操作盘。将两圆绘制与自定义控件中，通过手指对手机屏幕的触摸来改变小圆的方位，并通过小圆圆心和大圆圆心的连线和屏幕水平线的夹角便可以知道应当发送怎样的方位控制命令。服务器解析并执行这样的方位控制命令。  

3.采用Android平台，对于语音的识别可以采用百度语音，谷歌语音或讯飞语音。由于讯飞语音的离线识别下线了，先了解了百度语音，谷歌语音还未了解，故选择了百度语音。百度语音可以支持在线语音识别，离线语音识别，命令词唤醒等功能。由于小车WiFi并没有连接外网，故在编写识别代码时采用了离线识别api，但是由于百度语音离线识别技术有存在网络便会优先选择在线模式，故尽管代码采用离线识别api，只要联网，不管是否连入外网，都会尝试获取在线识别功能，所以识别速率会大幅降低。经测试可知，离线识别api可以在500-700ms内迅速返回识别结果，但一旦手机客户端连接到小车WiFi后识别速度在6s左右，且识别率大大下降。解决方法为将小车连入外网，便可以让客户端识别代码走在线模式，更加迅速的得到识别结果。最后将客户端识别结果转发给服务器，服务器解析并执行相应的控制命令，让小车按命名运行。
由于百度语音支持唤醒词的操作，并且通过测试，唤醒词的速率比在线识别的速率更快，因为唤醒词技术是一个本地技术，但是唤醒词的定义官方给了有一些限制，比如词长3-5汉字等，但最关键的是官方不允许一次定义过多自定义的唤醒词。一个唤醒词的bin文件中中包含最多3个。然而语音命令有前进，前进左转，左转等8个和停止1个。故采用唤醒词的方案被官方限制给抹杀了。  

4.采用Android平台，首先在自定义控件中设置画布，当手指触摸到手机屏幕时，记录下触摸点，将触摸点坐标添加到路径数组中。然后绘制完成后再对路径数组中的触摸点进行路径分析，解析成一个包含路径信息的字符串，发送给服务器。最后服务器解析后按照解析后的路径信息执行响应的控制命令。  
绘制后的路径数组解析可以按照以下原则进行解析。首先将前两个点的连线和垂直线进行角度计算和方向判定，并计算出连线的距离，将这3个结果拼接成命令字符串。然后将第3点和第2点连线和第2点和第1点连线进行先前的计算操作。以此类推直至全部的路径节点都被计算分析。最终发送命令字符串到客户端。  
## 1.3 需求分析
1.小车走迷宫，需要小车的电机控制，控制小车前进后退左转右转。需要小车的超声波传感器的距离检测，来监控小车是否碰壁和是否存在可走路径。  
2.方向盘控制，需要方向盘的UI设计和逻辑输出，通过确定方向盘的变化来控制小车的运行状况。  
3.语音控制，需要语音识别接口，如百度语音识别。语音输入，百度语音识别，获取正确的识别结果，发送至服务器控制小车运行状态。  
4.路径控制，需要自定义路径绘制控件和路径解析逻辑，发送服务器，服务器再解析，控制小车行走。  

# 2 系统总体设计  
## 2.1 总体设计方案的确定  
首先要分为客户端程序和服务器程序，客户端程序为Android程序，使用Android Stduio开发软件。服务器程序为C语言程序，使用notepad++这个一般的文本编辑器来开发。分为服务器程序和单独的迷宫程序。  
## 2.2 软硬件功能划分  
硬件功能：小车电机控制，超声波传感器控制  
软件功能：Android客户端设计，即迷宫界面设计，方向盘界面和逻辑设计，语音界面设计，路径界面和逻辑设计。服务端命令解析系统。  
## 2.3 硬件体系架构设计  
采用wiringPi树莓派开发库，便捷开发小车电机控制和超声波传感器控制。  
## 2.4 软件体系架构设计  
Android客户端：只有一个Activity其中包含一个ViewPager，ViewPager中包含4个自定义Fragment界面，每个Fragment界面代表一个功能界面。  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/42.png)  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/43.png)  
服务端命令解析系统：建立Socket服务器，读取来至Android客户端的命令数据，初步解析后分发给对应功能类型的处理函数处理。  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/44.png)    
命令系统： 
```
typedef enum comdType{//命令类型     
    	MAZE=1,		//迷宫  
	KEYOPT,		//方向盘  
	SOUNDOPT,	//语音  
	PATHOPT		//路径  
}cType;  

typedef enum comdSubType{//命令子类型    
    	CONF=1,		//参数,迷宫配置参数或路径链表  
	START,		//开始  
	END,		//结束  
        OPT			//操作项0-8  
				//0:停止 1:前进 2:前进左转 3:左转 4:后退左转 ...  
}csType;    

typedef struct Comd{//命令结构体  
     	cType ct;  
	csType cst;  
	union{  
	  char* conf;  
          int opt;  
	    }parm;  
}comd;  
```


例如:  
	"1:1:0.8,30,5,10,10,7.0\n"   
	"1:2\n"  
	"1:3\n"  
对应含义:  
	"maze:conf:0.8,30,5,10,10,7.0\n"  
	"maze:start\n"  
	"maze:end\n"  

处理函数：
```
dowork(buf,len);//处理命令数据  
void doMazeWork(comd* cd);//迷宫处理函数  
void doKeyoptWork(comd* cd);//方向盘处理函数  
void doSoundoptWork(comd* cd);//语音处理函数  
void doPathoptWork(comd* cd);//路径处理函数
```

# 3 系统详细设计 

## 3.1 硬件详细设计  

### 1.初始化wiringPi和小车控制端口 
```
#define Trig  28//前超声波  
#define Echo  29   
#define Trig1 24//左超声波  
#define Echo1 25  
#define Trig2 21//右超声波  
#define Echo2 22  

wiringPiSetup();  
pinMode(Echo, INPUT);  
pinMode(Trig, OUTPUT);  
pinMode(Echo1, INPUT);  
pinMode(Trig1, OUTPUT);  
pinMode(Echo2, INPUT);  
pinMode(Trig2, OUTPUT);/*WiringPi GPIO*/  
pinMode(1, OUTPUT);  //IN1 控制左边轮胎前进  
pinMode(4, OUTPUT);  //IN2 控制左边轮胎后退  
pinMode(5, OUTPUT);  //IN3 控制右边轮胎前进  
pinMode(6, OUTPUT);  //IN4 控制右边轮胎后退  
softPwmCreate(1, 1, 100);  
softPwmCreate(4, 1, 100);  
softPwmCreate(5, 1, 100);  
softPwmCreate(6, 1, 100); 
```

### 2.编写相应控制代码  
```
float disMeasure(int trig,int echo)//超声波测距
{
	//struct timeval {
	//	time_t tv_sec;	//64位系统下的time_t类型即long类型长度为8个字节
	//	suseconds_t tv_usec;
	//;
    struct timeval tv1,tv2;
    long start, stop;
    float dis;
    digitalWrite(trig, LOW);
    delayMicroseconds(2);
    digitalWrite(trig, HIGH);
    delayMicroseconds(10); //发出超声波脉冲
    digitalWrite(trig, LOW);
    while(!(digitalRead(echo) == 1));
    gettimeofday(&tv1, NULL); //获取当前时间
    while(!(digitalRead(echo) == 0));
    gettimeofday(&tv2, NULL); //获取当前时间
    start = tv1.tv_sec * 1000000 + tv1.tv_usec; //微秒级的时间 
    stop = tv2.tv_sec * 1000000 + tv2.tv_usec;
    dis = (float)(stop - start) / 1000000 * 34000 / 2; //求出距离
    return dis;
}


void brake(int time) //刹车，停车 time ms
{
    softPwmWrite(1, 0);
    softPwmWrite(4, 0);
    softPwmWrite(5, 0);
    softPwmWrite(6, 0);
    delay(time);//执行时间，可以调整
}


void run() // 前进
{
	softPwmWrite(1, 100*CONF.SPEEDSCALE); //左轮前进
    softPwmWrite(4, 0);
	softPwmWrite(5, 100*CONF.SPEEDSCALE); //右轮前进
    softPwmWrite(6, 0);
}

//length	cm
//curSpeed	cm/s 为正值
void runLength(float length,float curSpeed){
	float tempSpeed=curSpeed;
	float endTime=0;
	float frontDis=0;
	
	endTime=length*1000/tempSpeed+millis();//millis() wiringPiSetup开始后的ms

	while(1){
		run();
		frontDis=disMeasure(Trig,Echo);
		delay(50);
		if(millis()>=endTime||frontDis<CONF.DIS_COLLIDE_FRONT){
			brake(10);
			break;
		}
			
	}
}


void left(int time) //左转（左轮后退，右轮前进）
{
    softPwmWrite(1, 0);
    softPwmWrite(4, 100*CONF.SPEEDSCALE);//左轮后退
    softPwmWrite(5, 100*CONF.SPEEDSCALE);//右轮前进
    softPwmWrite(6, 0); 
    delay(time);
}

void leftAngle(unsigned char angle){
	int angleToTime = (int)CONF.ANGLEPERMT*angle;
	brake(200);
	left(angleToTime);
	brake(10);
}

void goLeft(float leftDis){
	//runLength(leftDis/2,SPEED);
	//leftAngle(90);
	//runLength(leftDis,SPEED);
	runLength(leftDis,SPEED);
	leftAngle(90);
	runLength(leftDis*1.4,SPEED);
}

void right(int time) //右转(右轮后退，左轮前进)
{
    softPwmWrite(1, 100*CONF.SPEEDSCALE);//左轮前进
    softPwmWrite(4, 0); 
    softPwmWrite(5, 0);
    softPwmWrite(6, 100*CONF.SPEEDSCALE);
    delay(time); //执行时间，可以调整
}

void rightAngle(unsigned char angle){
	int angleToTime = (int)CONF.ANGLEPERMT*angle;
	brake(200);
	right(angleToTime);
	brake(10);
}


void goLeftWall(float leftDis){//控制小车贴左墙行走
	int angleToTime=0;
	unsigned char leftFlag=leftDis>CONF.DIS_COLLIDE_MIN;
	unsigned char leftFlag2=leftDis<CONF.DIS_COLLIDE_MAX;
	if(!leftFlag){//距离左墙过近，右偏
		angleToTime = (int)CONF.ANGLEPERMT*3;
		//brake(100);
		PLOG(">>>>>>>>>GO Right\n");
		softPwmWrite(1, 100*CONF.SPEEDSCALE);//左轮前进
		softPwmWrite(4, 0); 
		softPwmWrite(5, 0);
		softPwmWrite(6, 100*CONF.SPEEDSCALE);//右轮后退
		delay(angleToTime);
	}
	if(!leftFlag2){//距离左墙过远，左偏
		angleToTime = (int)CONF.ANGLEPERMT*3;
		//brake(100);
		PLOG("<<<<<<<<<GO Left\n");
		softPwmWrite(1, 0);
		softPwmWrite(4, 100*CONF.SPEEDSCALE);//左轮后退
		softPwmWrite(5, 100*CONF.SPEEDSCALE);//右轮前进
		softPwmWrite(6, 0);
		delay(angleToTime);
	}
}

```

## 3.2 软件详细设计  
服务端Socket逻辑：(server.c)  
/*创建一个socket 指定IPv4协议族 TCP协议*/ 

    sfd = Socket(AF_INET, SOCK_STREAM, 0);
    bzero(&serv_addr, sizeof(serv_addr));           //将整个结构体清零
    serv_addr.sin_family = AF_INET;                 //选择协议族为IPv4
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);  //监听本地所有IP地址
    serv_addr.sin_port = htons(SERV_PORT);          //绑定端口号    

	/*绑定服务器地址结构*/
    ret=Bind(sfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
	
	/*设定链接上限,注意此处不阻塞*/
    ret=Listen(sfd, 2);                                
    printf("wait for client connect ...\n");

	/*获取客户端地址结构大小*/ 
    clie_addr_len = sizeof(clie_addr_len);
	
	/*参数1是sfd; 参2传出参数, 参3传入传出参数, 全部是client端的参数*/
	cfd = Accept(sfd, (struct sockaddr *)&clie_addr, &clie_addr_len);
	printf("cfd = ----%d\n", cfd);
	inet_ntop(AF_INET, &clie_addr.sin_addr.s_addr, clie_IP, sizeof(clie_IP));
	printf("client IP: %s  port:%d\n", clie_IP, ntohs(clie_addr.sin_port));
    while (1) {
		/*读取客户端发送数据*/
		len = Read(cfd, buf, sizeof(buf));
		if(len==0)
			count++;
		else if(len>0){
			//PLOG("len:%d\n",len);
			Write(STDOUT_FILENO, buf, len);

			/*处理客户端数据*/
			dowork(buf,len);
		}
		if(count>0xff||len<0)//继续监听客户端连接
		{
			count=0;
			Close(cfd);
			cfd = Accept(sfd, (struct sockaddr *)&clie_addr, &clie_addr_len);
			printf("cfd = ----%d\n", cfd);
			inet_ntop(AF_INET, &clie_addr.sin_addr.s_addr, clie_IP, sizeof(clie_IP));
			printf("client IP: %s  port:%d\n", clie_IP, ntohs(clie_addr.sin_port));
		}
    }
	 /*关闭链接*/
    Close(sfd);
	Close(cfd);


客户端Socket逻辑：  
(MainActivity.java)  
	// 初始化线程池
	mThreadPool = Executors.newCachedThreadPool();
		
	@Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();

        if (id == R.id.Connnect) {
			// 利用线程池直接开启一个线程 & 执行该线程
            mThreadPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        // 创建Socket对象 & 指定服务端的IP 及 端口号
                        socket = new Socket("192.168.12.1", 6666);
                        // 判断客户端和服务器是否连接成功
                        System.out.println(socket.isConnected());
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
            return true;
        }else if(id==R.id.Disconnect){
            try {
                // 断开 客户端发送到服务器 的连接，即关闭输出流对象OutputStream
                //outputStream.close();
                // 断开 服务器发送到客户端 的连接，即关闭输入流读取器对象BufferedReader
                //br.close();
                // 最终关闭整个Socket连接
                socket.close();
                // 判断客户端和服务器是否已经断开连接
                System.out.println(socket.isConnected());
            } catch (IOException e) {
                e.printStackTrace();
            }
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
	
```
 void sendData(final String str){
        mThreadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    if(socket==null)
                        return;
                    // 步骤1：从Socket 获得输出流对象OutputStream
                    // 该对象作用：发送数据
                    outputStream = socket.getOutputStream();

                    // 步骤2：写入需要发送的数据到输出流对象中
                    outputStream.write(str.getBytes("utf-8"));
                    // 特别注意：数据的结尾加上换行符才可让服务器端的readline()停止阻塞

                    // 步骤3：发送数据到服务端
                    outputStream.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        });
}
```

![image](https://github.com/QustRobot/AppOnCar/blob/master/images/45.png)  

### 3.2.1迷宫控制  
迷宫程序：  
```
fileUtil.h/c:配置文件定义和实现  
struct Conf{  
	float SPEEDSCALE; //最高速的比例0-1  
	unsigned char DIS_LEFT;	//当前超声波探测器测量的距离大于DIS_LEFT，表左方有路,单位CM  
	unsigned char DIS_COLLIDE_MIN;	//左贴墙最小距离  
	unsigned char DIS_COLLIDE_MAX;	//左贴墙最大距离  
	unsigned char DIS_COLLIDE_FRONT;//前进规避碰撞距离  
	float ANGLEPERMT; //偏转每度所需毫秒值  
};  
 
int readConfFile(struct Conf* CONF);//读取配置文件   
int writeConfFile(struct Conf* CONF);//写入配置文件   
void printConf(struct Conf CONF);//打印配置信息  
int praseConfStr(char* str,struct Conf* CONF);//解析配置字符串  
```
maze.c:控制小车在迷宫中行走的主逻辑，主体逻辑如下  
```
while(1){
		leftDis=disMeasure(Trig1,Echo1);//超声波测距，测出左边距离
		PLOG("leftDis = %0.2f cm\n", leftDis);
		if(leftDis>=CONF.DIS_LEFT){//左边有路
			PLOG("#############goLeft#############\n");
			goLeft(saveLeftDis);//进入左道
		}else
			saveLeftDis=leftDis;
		goLeftWall(leftDis);//在行走中判断小车与左墙的距离，并调整距离
				
		frontDis=disMeasure(Trig,Echo);//超声波测距，测出前方距离
		frontPath=frontDis>=CONF.DIS_COLLIDE_FRONT;
		PLOG("frontDis = %0.2f cm :frontPath=%d\n", frontDis,frontPath);
		if(frontPath){				
			PLOG("#############run#############\n");
			run();//前进
			delay(100);
			continue;
		}
		else{
			rightAngle(90+5);//进入右道
		}
	}
```
carOpt.h:小车的控制操作，见硬件详细设计    
Debug.h:调试用的头文件，一个宏定义，当编译时添加-DPDEBUG参数是便可以将PLOG打印功能使能，反之PLOG不会打印任何信息。   
#ifdef PDEBUG    
#define PLOG(fmt,args...) printf(fmt,##args)    
#else     
#define PLOG(fmt,args...) /*do nothing*/   
#endif   
Makefile:编译脚本   

服务器迷宫功能：    
命令格式   
"1:1:0.8,30,5,10,10,7.0\n"  
"1:2\n"  
"1:3\n"   
命令意义  
"maze:conf:0.8,30,5,10,10,7.0\n"//命令类型:迷宫;配置参数为0.8,30...  
"maze:start\n"//命令类型:迷宫;开始    
"maze:end\n"//命令类型:迷宫;结束    
命令操作    
当服务器收到"1:1:0.8,30,5,10,10,7.0\n"命令时，将配置参数保存。收到"1:2\n"命令时开启迷宫程序子进程。收到"1:3\n"命令时结束迷宫程序子进程。详见  server.c/doMazeWork函数    

Android客户端迷宫功能界面：   
界面：fragment_maze.xml，通过拖拽UI单元，复制，后简单设定便可获得以下界面效果。（按钮为第三方库中的控件）  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/46.png)   

逻辑：通过设置3个按钮的点击监听事件，当点击配置按钮时，发送配置编辑框中的内容即配置命令字符串到服务器中；当点击开始按钮时，发送开始命令字符串；当点击结束按钮时，发送结束命令字符串。详见com.carclienta.FragmentMaze.java文件。  
 
 ### 3.2.2方向盘控制  
服务器方向盘功能：  
命令格式  
"2:4:1\n"  
命令意义  
"keyopt:opt:1\n"//命令类型:方向盘;前进  
// 0:停止 1:前进 2:前进左转 3:左转 4:后退左转 ...  
命令操作  
当服务器收到"2:4:1\n"命令时,开始方向盘操作并控制小车运动。详见server.c/doKeyoptWork函数  

Android客户端迷宫功能界面：  
界面：fragment_keyopt.xml，使用自定义控件GamePad  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/47.png)   
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/48.png)   
逻辑：自定义控件GamePad中有方位监听接口，实现这个接口，在接口实现中发送相应的诸如"2:4:1\n"的命令字符串到服务器中。详见com.carclienta.FragmentKeyOpt.java.自定义控件GamePad原理详见1.2可行性分析。代码详见com.carclienta.GamePad.java和com.carclienta.Rocker.java  

### 3.2.3语音控制  
服务器语音功能：  
命令格式  
"3:2\n"  
"3:4:1\n"  
"3:3\n"  
命令意义  
"soundopt:start\n"//命令类型:语音;开始  
"soundopt:opt:1\n"//命令类型:语音;前进  
// 0:停止 1:前进 2:前进左转 3:左转 4:后退左转 ...  
"soundopt:end\n"//命令类型:语音;结束  
命令操作  
当服务器收到"3:2\n"命令时表示服务器现在处理的命令为语音控制命令且为开始态；当服务器收到诸如"3:4:1\n"命令时，服务器便控制小车做出相应的运动；当服务器收到诸如"3:3\n"命令时，结束当前命令类别的操作。详见server.c/doSoundoptWork函数  


Android客户端迷宫功能界面：  
界面：fragment_soundopt.xml通过拖拽UI单元，后简单设定便可获得以下界面效果。（按钮为第三方库中的控件）  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/49.png)  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/50.png)  
逻辑：首先集成百度语音识别，步骤如下。然后在handler处理信息函数根据识别状态来循环开启语音识别，这样便可以做到不间断的语音命令控制。并识别成功是发送相应命令字符串。详见com.carclienta.FragmentSoundOpt.java  
```
	// 集成步骤 1.1 新建一个回调类，识别引擎会回调这个类告知重要状态和识别结果
	listener = new SimpleRecogListener(handler);
	// 集成步骤 1.2 初始化：new一个IRecogListener示例 & new 一个 MyRecognizer 示例
	myRecognizer = new MyRecognizer(getActivity(),listener);
	if (enableOffline) {
		// 集成步骤 1.3（可选）加载离线资源。offlineParams是固定值，复制到您的代码里即可
		offlineParams = OfflineRecogParams.fetchOfflineParams();
		myRecognizer.loadOfflineEngine(offlineParams);
	}
	// 集成步骤2.1 开始识别
	myRecognizer.start(offlineParams);
	// 集成步骤2.2 结束识别		
	myRecognizer.stop();
				
	// 集成步骤3 释放资源
	// 如果之前调用过myRecognizer.loadOfflineEngine()， release()里会自动调用释放离线资源
	myRecognizer.release();
		
	//android 6.0 以上需要动态申请权限,详见FragmentSoundOpt.java/initPermission函数
	//AndroidManifest.xml菜单文件配置-权限
	<uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	//AndroidManifest.xml菜单文件配置-百度语音应用信息
	<application>
        <meta-data
            android:name="com.baidu.speech.APP_ID"
            android:value="xxx" />
        <meta-data
            android:name="com.baidu.speech.API_KEY"
            android:value="xxx" />
        <meta-data
            android:name="com.baidu.speech.SECRET_KEY"
            android:value="xxx" />
</application>
```

注：SimpleRecogListener MyRecognizer类为自定义类，在com.carclienta包下和core库中。  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/51.png)   

### 3.2.4路径控制  
服务器路径功能：  
命令格式  
"4:1:1=20,3=90,1=30,7=90,1=60"  
"4:2"  
"4:3"  
命令意义  

"pathopt:conf:1=20,3=90,1=30,7=90,1=60"  
//路径链表 1=20,3=90,1=30,7=90,1=60  
//1:前进		1=距离  
//3,5:左右转	5,7=角度  
//前进20cm，左转90°，前进30cm...  
"pathopt:start"//命令类型:路径控制;开始  
"pathopt:end"//命令类型:路径控制;结束  
命令操作
当服务器收到诸如"4:1:1=20,3=90,1=30,7=90,1=60"命令时，将路径链表解析到路径结构体中，当收到"4:2"命令后，开启子线程，并从路径结构体中获取路径节点，根据路径节点的信息控制小车运动状态。当收到"4:3"命令后，关闭子线程，结束小车的路径控制。详见server.c/doPathoptWork函数。以下为路径结构体结构：（详见Path.h/c） 
```
#define PATHMAX 100 //路径节点默认最多数目

typedef enum Direction{//方向
	RUN=1,
	LEFT=3,
	BACK=5,
	RIGHT=7
}Dir;

typedef struct Node{//路径节点
	Dir dir;
	int val;
	struct Node* next;
	struct Node* pre;
}node;

typedef struct Path{//路径
	node* head;
	node* tail;
	int num;
}path;
```


Android客户端迷宫功能界面：    
界面：fragment_pathopt.xml,主要为一个ImageView和android.view.ext.SatelliteMenu（第3方自定义控件）。效果如下：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/52.png)
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/53.png)   

逻辑：设定（第3方自定义控件）SatelliteMenu卫星菜单的点击监听，有画布清理，路径字符串发送，开始命令命令发送，结束命令字符串发送。路径绘制原理详见1.2可行性分析。代码详见com.carclienta.FragmentSoundOpt.java和com.carclienta.Path.java,以下为Path.java的大体情况： 
```
public class Path {
    private double scale;//每像素代表的距离 cm/dp
    public Path(double scale) //构造函数
	//解析路径，将触摸点数组，转换为路径命令字符串
    public String prasePath(List<FragmentPathOpt.point> lp)
	//获取连线1和连线2的夹角
    private int getDegrees(float firstX, float firstY, float secondX, float secondY, float thridX, float thridY)
	//获取连线2相对于连线1的方位
    private int getDir(float firstX, float firstY, float secondX, float secondY, float thridX, float thridY)
	//获取连线2的长度
    private double getDistance(float firstX, float firstY, float secondX, float secondY)
}
```  

# 4 功能测试  
由于小车以放回实验室，故无法做小车的功能测试截图，所以以下为Android客户端测试结果。  
## 1.迷宫测试  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/54.png)
## 2.方向盘测试  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/55.png)
## 3.语音测试  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/56.png)
## 4.路径测试  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/57.png)












