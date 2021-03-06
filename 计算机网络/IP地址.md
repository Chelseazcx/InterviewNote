# IP地址

## 简介

两台计算机通过TCP/IP协议通讯的过程如下所示

![image](https://github.com/orangehaswing/InterviewNote/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/resource/201204251235454745.png?raw=true)

传输层及其以下的机制由内核提供，应用层由用户进程提供,应用程序对通讯数据的含义进行解释，而传输层及其以下处理通讯的细节，将数据从一台计算机通过一定的路径发送到另一台计算机。应用层数据通过协议栈发到网络上时，每层协议都要加上一个数据首部（header），称为封装（Encapsulation），如下图所示:

![image](https://github.com/orangehaswing/InterviewNote/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/resource/201204251235468989.png?raw=true)

 

不同的协议层对数据包有不同的称谓，在传输层叫做段（segment），在网络层叫做数据报（datagram），在链路层叫做帧（frame）。数据封装成帧后发到传输介质上，到达目的主机后每层协议再剥掉相应的首部，最后将应用层数据交给应用程序处理。

上图对应两台计算机在同一网段中的情况，如果两台计算机在不同的网段中，那么数据从一台计算机到另一台计算机传输过程中要经过一个或多个路由器，如下图所示:

![image](https://github.com/orangehaswing/InterviewNote/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/resource/201204251235479088.png?raw=true)

 

IP地址是网际协议地址（internet protocol address,IP地址）的简称。用于internet上主机的唯一标识。

它的地址由类别、网络地址和主机地址共3个部分组成。类别区分地址的使用方式，网络地址用于区分不同的网络，主机地址用于在一个网络中区分主机。

IP地址：={（网络号），（主机号）}

IP地址分成5类：A类（Class A），B类（Class B），C类（Class C），D类（Class D），E类（Class E）。

其中A、B和C类地址是基本的internet地址，是用户使用的地址，D类地址用于多目标广播的广播地址，E类地址为保留地址。

- A类：1.x.y.z-126.x.y.z（其中，127.0.0.1不作IP地址，用于网络内部使用）
- B类：128.x.y.z-191.x.y.z
- C类：192.x.y.z-223.x.y.z
- D类：224.x.y.z-239.255.255.255（其中，244.0.0.0不用，224.0.0.1分配给永久性IP主机组，包括网关）

一个A类网络可容纳的地址数量最大，一个B类网络的地址数量是65536，一个C类网络的地址数量是256。D类地址用作多播地址，E类地址保留未用。

特殊的IP地址

- 网络地址：主机地址为全“0”的IP地址不分配给任何主机，而是作为网络本身的标识。例：主机202.198.151.136所在网络的网络地址为202.198.151.0；
- 直接广播地址：主机地址为全“1”的IP地址不分配给任何主机，用作广播地址，对应分组传递给该网络中的所有结点（能否执行广播，则依赖于支撑的物理网络是否具有广播的功能）。
- 有限广播地址：32位为全“1”的IP地址（255.255.255.255）称为有限广播地址，通常由无盘工作站启动时使用，希望从网络IP地址服务器处获得一个IP地址。
- 主机本身地址：32位为全“0”的IP地址（0.0.0.0）称为主机本身地址。
- 回送地址：127.0.0.1称为回送地址，常用于本机上软件测试和本机上网络应用程序之间的通信地址。

## 子网掩码

子网掩码的作用就是和IP地址与运算后得出网络地址，子网掩码也是32bit，并且是一串1后跟随一串0组成，其中1表示在IP地址中的网络号对应的位数，而0表示在IP地址中主机对应的位数。

### 标准子网掩码

​	A类网络（1 - 126） 缺省子网掩码：255·0·0·0

255·0·0·0 换算成二进制为 11111111·00000000·00000000·00000000

​	可以清楚地看出前8位是网络地址，后24位是主机地址，也就是说，如果用的是标准子网掩码，看第一段地址即可看出是不是同一网络的。如 21.0.0.0.1和21.240.230.1，第一段为21属于A类，如果用的是默认的子网掩码，那这两个地址就是一个网段的。

​	B类网络（128 - 191） 缺省子网掩码：255·255·0·0

​	C类网络（192 - 223） 缺省子网掩码：255·255·255·0

 B类、C类分析同上。

### 特殊的子网掩码

​	标准子网掩码出现的都是255和0的组合，在实际的应用中还有下面的子网掩码

- 255·128·0·0

- 255·192·0·0

  ……

- 255·255·192·0

- 255·255·240·0

  ……

- 255·255·255·248

- 255·255·255·252

A 10.0.0.0-10.255.255.255
B 172.16.0.0-172.31.255.255
C 192.168.0.0-192.168.255.255

A类网络缺省子网掩码：255.0.0.0

B类网络缺省子网掩码：255.255.0.0

C类网络缺省子网掩码：255.255.255.0

192·168·0·1和192·168·0·200如果是默认掩码255.255.255.0两个地址就是一个网络的，如果掩码变为255.255.255.192这样各地址就不属于一个网络了









