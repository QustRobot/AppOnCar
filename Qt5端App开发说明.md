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
  
3.Android NDK：  
https://developer.android.com/ndk/downloads/  
下载解压即可。  
环境变量设置：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/8.png)  
以上安装完成，在QT中填入路径即可：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/9.png)  
最后创建AVD，方便电脑虚拟机运行安卓程序：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/10.png)  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/11.png)  
如果创建不成功，检查Android SDK 是否都正确下载安装。

3.2 应用程序设计与调试  
1．迷宫程序设计  
```
while(1)  
    {  
	     	//获取当前三个方向的距离和两个光传感器的信号  
	    disf = disMeasure(1);  
            disl = disMeasure(2);  
            disr = disMeasure(3);    
	    SR = digitalRead(RIGHT);    
            SL = digitalRead(LEFT);  
             if(SL==HIGH && SR==HIGH && (disf<25||disf>2000))    
//测得前方障碍的距离小于30cm时做出如下响应  
            {  
              if((disr<20||disr>2000)&&(disl<20||disl >2000)) //两侧都有障碍  
              {  
		 		brake(1);  		
                                left(10);       //180度掉头 需要实际调整！！！  
              }  
              else if(disr<disl)//只有右侧有障碍物体  
              {  
				brake(1);  
                left(5);        //左转  
				delay(50);  
              }  
              else              //左边有障碍  
              {  
				brake(1);  
                right(5);       //右转  
				delay(50);  
              }  
            }  

		 if(SL==LOW && SR==LOW) //光传感器有信号，近距离两侧都有障碍  
		{  
				if(disr<disl){back(2);left(4);}  //向距离大的一侧转弯调整  
					else {back(2);right(4);}  
		}  
		else if(SL==LOW || SR==LOW)//近距离一侧有障碍  
		{  
			if(disr<disl){back(2);left(2);}      //向距离大的一侧转弯调整  
					else {back(2);right(2);}  
		}   
		else//没有障碍  
		{  
			run();  
			delay(100);  
		}  
	} 
```
2.小车服务端关键程序  
（1）main()函数  
在main函数中创建两个线程，一个手动控制线程，一个TCP通信线程。在手动控制线程中，设置while循环，动作状态变量shouDongActive的值映射为小车的动作状态。 
```
int main()  
{  
    wiringPiSetup();  
    /*WiringPi GPIO*/  
    pinMode (1, OUTPUT);  
    pinMode (4, OUTPUT);  
    pinMode (5, OUTPUT);  
    pinMode (6, OUTPUT);  
    softPwmCreate(1,1,500);  
    softPwmCreate(4,1,500);  
    softPwmCreate(5,1,500);  
    softPwmCreate(6,1,500);  
   //创建线程：  
    ret=pthread_create(&id1,NULL,(void *) shouDongThread,NULL);//←手动控制线程  
    if(ret!=0)  
    {  
        printf("Create pthread error\n");  
        exit(1);  
    }  
    ret=pthread_create(&id2,NULL,(void *) socketThread,NULL);//←Tcp线程  
    if(ret!=0)  
    {  
        printf("Create pthread error\n");  
        exit(1);  
    }  
    pthread_join(id1,NULL);  
    pthread_join(id2,NULL);  
}  
```
（2）TCP通信线程  
在TCP线程中，先建立连接，接收到的字符存入buf中，buf的内容映射为小车动作状态变量shouDongActive的值，和创建、结束迷宫进程。
```
void  socketThread()  
{  
    struct sockaddr_in server_sockaddr,client_sockaddr;  
    int sin_size=1,recvbytes;  
    int sockfd;//client_fd;  

    if((sockfd=socket(AF_INET,SOCK_STREAM,0))==-1)  
    {  
        perror("socket");  
        exit(1);  
    }  
    printf("socket success! sockfd=%d\n",sockfd);  
  
    int val = 1;  
    int ret = setsockopt(sockfd,SOL_SOCKET,SO_REUSEADDR,(void *)&val,sizeof(int));  
    if(ret == -1)  
    {  
        printf("setsockopt");  
    }  

    server_sockaddr.sin_family=AF_INET;  
    server_sockaddr.sin_port=htons(SERVPORT);  
    server_sockaddr.sin_addr.s_addr=INADDR_ANY;  
    bzero(&(server_sockaddr.sin_zero),8);  

    if(bind(sockfd,(struct sockaddr *)&server_sockaddr,sizeof(struct sockaddr))==-1)  
    {  
        perror("bind");  
        exit(1);  
    }  
    printf("bind success!\n");  
    if(listen(sockfd,BACKLOG)==-1)  
    {  
        perror("listen");  
        exit(1);  
    }  
    printf("listen success\n");  
    if((client_fd=accept(sockfd,(struct sockaddr *)&client_sockaddr,&sin_size))==-1)  
    {  
        perror("accept");  
        exit(1);  
    }  
    printf("accept success\n");  

    while(1)  
    {  
        if((recvbytes=recv(client_fd,buf,1,0))>0)  
        {  
            printf("Receive:%s\n",buf);  
            switch(buf[0])  
            {  
            case '0':  
                system("sudo killall maze");//接收到0，结束迷宫程序  
                shouDongActive = 0;      //停止手动程序  
                break;  
            case '1':  
                system("sudo killall maze");//接收到1，结束迷宫程序  
                shouDongActive = 1; //车的动作状态为前进  
                break;  
            case '2':   
                system("sudo killall maze");//接收到2，结束迷宫程序  
                shouDongActive = 2;//车的动作状态为后退  
                break;  
            case '3':  
                system("sudo killall maze");//接收到3，结束迷宫程序  
                shouDongActive = 3;//车的动作状态为左转  
                break;  
            case '4':  
                system("sudo killall maze");//接收到4，结束迷宫程序  
                shouDongActive = 4;//车的动作状态为右转  
                break;  
            case '5':  
                system("sudo killall maze");//接收到5，结束迷宫程序  
                system("sudo ./maze&"); //后台运作迷宫程序  
                shouDongActive = 0; //停止手动控制  
                printf("5!");  
                break;  
  
            case '9':  
                system("sudo killall maze");//接收到9，结束迷宫程序  
                close(client_fd);//关闭端口号  
                close(sockfd);  
                shouDongActive = 0; //停止手动控制  
                system("sudo ./server3&");//重新运行服务端程序  
                exit(0);  
                break;  
            default:  
                break;  
            }  
        }  

        else if(recvbytes<=0)  
        {  
            printf("Connection closed!\n");  
            close(client_fd);  
            client_fd=accept(sockfd,(struct sockaddr *)&client_sockaddr,&sin_size);  
        }  
    }  
    printf("thread over\n");  
} 
```
（3）手动控制线程 
```
void shouDongThread()  
{  
	  brake(1);  
    printf("shouDongThread start!!\n");  
    while(1)  
    {  
			switch(shouDongActive)  
			{  
			case 0:brake(1);break; //动作状态为0，车停止  
			case 1:run();break;    //动作状态为1，车前进  
			case 2:back();break;   //动作状态为2，车后退  
			case 3:left(1);break;  //动作状态为3，车左转  
			case 4:right(1);break; //动作状态为4，车右转  
			default:break;  
			}  
    }  
}  
```

4 功能测试  
4.1测试手机遥控功能  
（1）先编译小车服务端程序server3.c和迷宫程序maze.c；  
（2）运行服务端程序server3，显示如下，说明两个线程创建成功，等待与手机连接。  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/12.png)  
（3）手机连接小车WiFi：QUST-ROBOT-XXX，密码12345678；打开手机“智能车遥控器”，输入IP和端口号，显示界面如下：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/13.png)  
（4）点击Connect按钮，显示accept success,表示和手机连接成功。
按住“向上”按钮，小车前进，松开小车停止。小车接收到的信息如下：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/14.png)  

4.2测试走迷宫功能  
（1）迷宫如下图所示  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/15.png)  
（2）将小车放在起点位置，点击遥控“走迷宫”按钮，小车用时13秒到达终点，中途未碰到障碍物，小车路线为：  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/16.png)  
（3）小车到达终点，点击“stop”按钮，小车停止，测试结束，实现了预期的功能。  
![image](https://github.com/QustRobot/AppOnCar/blob/master/images/17.png)  
