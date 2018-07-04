# 用户手册

### 应用背景

随着互联网的快速发展和智能设备的大量普及，人们对于基于位置服务的需求日益增长，尤其是在室内的复杂环境中。GPS是目前应用最广泛的定位技术，但是在室内，受到建筑物的影响，信号强度大大衰减、多径效应使得GPS无法在室内实现较为精确的定位。

目前WLAN技术普遍应用在了家庭、旅馆、机场等各类大型及小型建筑物内，相关的研究如雨后春笋，开发了许多能实现较高精度定位的算法，使得WI-FI室内定位成为了定位领域中最引人注目的定位技术之一。

而随着传感器技术的进步和发展，PDR航位推算（Pedestrian Dead Reckoning）的精确性不断提高，使得它也成为了室内定位的可用选项之一。

本项目尝试使用WI-FI位置指纹法、PDR航位推算法进行室内定位，并对位置定位的算法进行改进以获得更好的室内定位效果，主要包括以下几点：
* 利用Android手机采集实际环境中的WIFI接收信号强度数据，并根据设定Reference Point采集到的信号强度值（RSS）生成指纹库；
* 在Android手机上设计并实现了一个基于位置指纹的WI-FI室内定位系统，并在真实的WI-FI环境实现相关算法及其最优的参数选择；
* 分别使用K近邻、基于贝叶斯的概率估计算法完成基于RSS差值的定位并在学校内实测效果；
* 利用Android手机传感器采集加速度信息，利用PDR航位推算算法实现室内定位并实现地图的可视化。

### 使用源代码
你需要添加百度地图相关的so包才能使用此程序。相关的包添加到jniLibs文件夹下面即可。本程序依赖于地图包进行可视化绘制。

### 功能简介

本软件基于 Anrdoid 和 Android 手机集成传感器开发，进行了 WIFI 和 PDR 室内定位的实验。本手册用于简单说明软件使用方法和用户界面构成。

#### WIFI定位功能

![wifi1](/media/wifi_positioning_1.png) ![wifi1](/media/wifi_positioning_2.png) ![choose_algorithm](/media/algorithm_pick.png)
- 添加结束接入点和采样点之后，采样点会被绘制到地图上，随着wifi强度信息的积累，定位的位置会随着用户的走动不断变化。
- 使用WIFI 定位功能之前，用户可以选择使用的算法。系统实现了几种基本算法：KNN(Euclidean \ Cosine) \ Bayes。用户可以使用不同的算法进行试验。

#### WIFI定位离线步骤
![add_ap](/media/adding_access_point.png) ![add_rp](/media/adding_reference_point.png) ![delete_rp](/media/deleting_reference_point.png)

- 使用WIFI 定位功能之前，用户需要采集当地的wifi指纹。
- 点击`+接入点`按钮，系统自动添加强度足够的SSID到数据库中，这些信号源参与定位比较。
- 点击`+参考点`按钮，进入添加参考点界面。用户可以拖动界面上的红色手柄调节参考点的地理位置。软件通过基站和GPS等自动给出估计的位置。
- 长按对应的参考点和接入点可以删除相对应的数据。

#### PDR定位功能

![pdr](/media/pdr.png)
- 用户将手机对准正前方稳定握持，用户点击开始之后，算法自动开始记录手机加速度数据。
- 用户的轨迹会被一根线记录在地图上。
- 通过拖动手柄调节起始位置。
- 通过拖动滑动条调节传感器磁偏角初始误差。
- 通过调节下面的滑动条调节步长和实际距离之间的比例关系。


### 软件架构

软件主要有四大部分构成，本地数据库、Android传感器和WIFI服务、算法、用户界面，各个部分的依赖关系如图所示。
![01.系统架构设计](/assets/系统架构.svg)

#### 类清单

```tree
.类名和目录                                   功能说明
├── CoreAlgorithm.java                      核心算法部分程序，用于WIFI定位
├── SharedConstants.java                    Android程序常数公共类
├── Utilities.java                          工具函数类
├── WifiPositioningApplication.java         Application顶层类
├── WifiService.java                        Wifi服务类
├── model                                   数据模型类集合
│   ├── AccessPoint.java                    AP数据存储
│   ├── LocationWithDistance.java           数据包类型，组合了距离和位置
│   ├── LocationWithNearbyPlaces.java       数据包类型，组合了最近位置和一串LWD
│   ├── Project.java                        数据包类型，组合了所有的AP和RP信息
│   ├── ReferencePoint.java                 数据包类型，组合了所有的RSSI信息和位置信息
│   ├── WifiNetwork.java                    数据包类型，组合了某一个AP的强度和BSSID、SSID信息
│   └── WifiData.java                       包含了一列WN类型的类
└── view
    ├── AddReferencePoint.java              添加参考点
    ├── AlgorithmDebugActivity.java         算法调试界面
    ├── PDRActivity.java                    PDR调试界面
    ├── PositioningActivity.java            WIFI定位界面
    ├── PrefActivity.java                   算法选择界面
    ├── ProjectDetailActivity.java          主界面，包括所有界面的入口和大多数功能
    ├── SearchWifiAPActivity.java           添加接入点
    └── viewfrags                           大量组成界面的工具类
        ├── APSection.java                  
        ├── NRAdapter.java                  
        ├── PointViewHolder.java            
        ├── PrefsFragment.java              
        ├── RPSection.java                  
        ├── RecyclerItemClickListener.java  
        ├── ReferenceReadingsAdapter.java   
        ├── SectionHeader.java              
        └── WifiResults.java                

3 directories, 28 files
```

### 算法简介

#### 基于RSSI指纹的WI-FI室内定位技术

基于WI-FI的室内定位技术可以有多种实现方式，基本上分为：基于无线信号三边的三角定位、位置指纹定位和融合定位。三边信号定位由于其理论模型的不实际性，难以应对现实中的多路径效应及波动，使得实际定位效果难以达到室内定位的要求精度。融合定位算法则难以做到普适使用，不同地方的数据源、不同手机的差异性都会造成结果的巨大差距。本项目采用基于接收信号强度（RSS）的位置指纹法实现的WI-FI室内定位，能够考虑室内复杂环境下信号的时变性而定位成本低，方法实现灵活而精度能够达到预期目标。

信号的RSS或者接受功率取决于接收器的位置，是大多数无线通信设备正常运行中用以感知链路质量、实现切换、适应传输速率等功能的必需参数。RSS不受信号带宽的影响，假设有一个固定的信号发射源，在离它不同距离的位置上的平均RSS的衰减和距离的对数成正比。对于一个可以接收来自多个发射源信号的移动设备，我们可以通过使用来自多个发射源或者接收器的RSS组成的RSS向量，作为和位置相联系的指纹。 注意到RSS本身就是在一段时间内计算或测得的，因此只采集一个RSS样本是不合理的。在WIFI网络中，AP常常要发送一个beacon帧，约100ms发送一次，RSS通常使用这个beacon帧来测量的。它是未加密的，所以即使是一个封闭的网络，也能用来进行定位。

对于特定的位置，都有唯一的可测物理量映射，这种用于标识位置的可测物理量被称为位置指纹。在本系统中，采用WIFI接收信号强度（RSS）来作为位置指纹。一般来说，此定位方法一共分为两个阶段，离线数据采集和在线定位。离线数据采集阶段主要是在预先设定的参考点（RP）上采集多个无线接入点（AP）的接收信号强度样本，并存入位置指纹数据库中。 在线定位阶段，客户端实时采集AP信号强度信息，通过定位算法和指纹数据库中的数据进行匹配，估计用户所在的位置，后文的定位算法都基于此。

##### 基于KNN的简单定位算法

K近邻（KNN）算法通过测量不同特征值之间的距离进行分类，我们这里以参考点获得的指纹向量与RSS向量的欧氏距离定义两个位置的相似度。在指纹库中的M个指纹中，分别计算在信号空间中与RSS观测值的欧氏距离，然后将它所对应的位置坐标作为移动设备的位置。对所有位置计算出的坐标求均值，我们能够得到计算出的所在位置的结果。

K近邻算法的最终效果并不足够好，通过进一步的观察，我们发现实际上和RSS向量欧氏距离越近的点，对最终结果的所占权重也应该越大，于是我们应用了加权K近邻（WKNN）算法对其进行了进一步优化。以欧氏距离的倒数为权重。考虑过权重之后，最终实现的定位效果要比单纯的KNN算法好。

##### 基于贝叶斯模型估计的定位算法

最早的基于WiFi位置指纹的概率性定位算法是Youssef et al. (2003).提出的，基本的思路是，如果简单地使用一个RSS样本的统计量（比如RSS的均值）可能会带来误差，因为实际的RSS值应该是一个分布。因此，我们可以使用联合概率分布（有多个AP，所以是联合概率分布）来作为指纹。通过采集RSS样本获取联合概率分布并不是一个简单的事情，因为来自各个AP的RSS之间的相互关系不明显。他们假设这是独立的（这种假设是合理的），假设RSS服从高斯分布，然后简单地使用RSS的边缘分布的乘积作为联合分布。由贝叶斯公式，我们可以根据似然来确定后验概率的大小，从而获得相应最可能成为当前位置的点。

使用贝叶斯方法计算完各个模型的概率之后，使用这些概率进行最终结果的计算。模型的可能性和位置进行加权即得到估算的位置。

##### 基于向量余弦相似度的定位算法

余弦相似度衡量的是2个向量间的夹角大小，通过夹角的余弦值表示结果。可以容易的想到，两个向量夹角余弦值越接近于1，两个向量也就越接近。基于此，我们可以利用余弦相似度来衡量RSS向量和指纹向量之间的差距，并最终进行定位。我们对每个RSS向量求余弦相似度，获得一个相同维度的向量。为了使得每个RSS值都能够对最终结果有所贡献，对此向量的每个分量求得均值后，我们得到了最终计算的定位结果。实测表明，效果能够达到预期要求。

##### RSSI算法实现遇到的困难

- 信号强度时变性太强，非常不稳定，经过测试，典型的强度波动为10dBm，两次检测出来的信号强度像是两个不同地方的结果。随着距离的增加，信号的波动越来越剧烈，在低信号强度的情况下，典型波动达到20dBm，甚至偶尔不能检测到信号。这些因素给提高算法精度造成困难。
- Android 端性能限制，采样频率极低，导致定位延迟很高，难以进行调试。
- 经过测试，信号强度和手机空间朝向有关，不得不在指纹中添加方向信息，导致参考点数量大量增加。
- 指纹库无法快速建立。按照一个点采样10次，一次采样3秒间隔，每个点四个大概方向计算，采样$400m^2$内所有的点，消耗的时间在几个小时左右。

![signal_direction_variation](/assets/signal_direction_variation.png) ![typical_time_sequence](/assets/typical_time_sequence.png)

##### RSSI算法可能的改进方向
- 降低低强度信号的计算权重，提高高质量信号的权重。
- 计算上，可以通过排除距离过大参考点来实现加速。
- 通过其他定位方法缩小指纹匹配范围和接入点范围，比如通过PDR确定大概的位置范围。

#### PDR方法

航位推算技术最早应用于航海，其基本原理为利用航向传感器和速度传感器获取单位时间内载体的转角和位移，从而对由前一时刻载体的位置和航向推算出当前时刻载体的位置和航向。此算法的定位精度，取决于初始位置和姿态信息的精确性和推算过程中速度和航向信息求解的精度。在本项目中，初始位置设置为手动调试，而航向信息、加速度等可以通过安卓手机的传感器获得。航位推算可以作为其他定位方法的辅助，比如在GPS信号丢失时参与位置估计。

我们通过加速度传感器峰值的个数进行了步频探测，在计算出相应的步频之后，利用论文中得出的步频和步长之间的特征训练模型得到了最终的步长结果。使用一个低通滤波器能够较好的实现内部对磁信号的滤波。通过融合磁传感器和加速度传感器最终确定当前航向角。我们利用可视化工具，使其能够精确地描绘出我们的行走路线，实现PDR算法的测试。

Android 提供的开发包给开发带来了很大便利，提供了包括自动的硬铁偏移校准，内置的动作识别算法，以及内置的传感器融合航向角方法。Android 没有开放传感器校准API，其内部的实现不断进行着传感器矫正工作。

传感器融合本身涉及到一套复杂的算法，在Android平台上可能已经通过软件甚至硬件实现。AHRS有很多种实现方法，其中比较著名的包括[Madgwick方法](http://www.x-io.co.uk/res/doc/madgwick_internal_report.pdf)。

- 典型手持设备Z轴加速度时间序列

![typical_acceleration_plot](/assets/typical_acceleration_plot.png)

- 典型手持设备加速度绝对值时间序列

![acc_abs](/assets/acc_abs.png)

- 步长估计公式

$$
Steplength = \left\{
\begin{aligned}
0.4375\quad 0 \le F \lt 1.35 & \ \\
0.45F - 0.17\quad 1.35 \le F \lt 2.45 & \ \\
0.9325\quad 2.45 \le F \lt +\infty
\end{aligned}
\right. meters
$$

- 简单加速度计计步原理
  - 通过加速度计的绝对值进行处理，对加速度计信号进行滑动平均以及计算其局部变化率。低于阈值的信号不进行考虑。
  - 对滤波的信号找到局部极大值点，并且要求时间上间隔足够远，并且在最近时间内曾经有一个极大值。

##### PDR方法误差来源

经过测试，步长的误差并不是主要的误差来源，即使采用固定的步长，导致的误差也不是很大。航向角误差是手持计步方法最大的问题。室内的地磁异常比室外大得多，磁场方向受房屋结构，铁磁体对手机传感器影响严重，这是航向角误差的一个主要来源。另一个来源在于手机廉价传感器的漂移和用户手持方向随时间变动、以及地磁偏角和地理北的角度差，这些导致了方向的系统误差，甚至是时变的系统误差，导致巨大的位置偏差。因此PDR方法无法实现长时间精确定位，需要通过融合方法配合WIFI、地磁、地图区域判断等定位方法进行矫正。

如图为电院四号楼走廊东侧出现肉眼可见的地磁异常。进入走廊中间部分之后出现大约10度的方向偏差。

![mag_abnormal](/assets/mag_abnormal.png)

如图为在楼内一次行走获得的磁场强度绝对值时间序列，磁场受到建筑结构的影响是巨大的，在任何不空旷的室内都会遭到剧烈干扰。这个问题不可能通过改进现有算法解决，只能另辟蹊径。

![mag_interference](/assets/mag_interference.png)

##### PDR的改进方法

- 使用其他定位方法进行定期校准
- 采用带有路径限制的地图进行自动校准
- 提供步态检测功能，提供更精确的步长，上下楼等行为的检测
- 使用多传感器融合提高精度
