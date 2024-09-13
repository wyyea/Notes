# 概念
+ node: 终端/主机
+ channel: 信道
+ Net Device：网卡，连接终端和信道
+ Topology helper: 构建 channel, net device 并完成连接
# 脚本
## 1 基本步骤 (first.cc)
+ 使用 NodeContainer 创建若干终端
+ 创建 Topology helper 并设置网卡和信道的属性
+ 使用 Topology helper 在 NodeContainer 上安装网卡和信道
+ 使用 Stack helper 在 NodeContainer 上安装协议栈 (注意在多个 NodeContainer 拥有同一个节点时只需安装一次)
+ 使用 Ipv4InterfaceContainer 为各 Node 分配 IP 地址
+ 使用 ApplicationHelper 为各 Node 安装应用层并设置属性
+ 使用 `Ipv4GlobalRoutingHelper::PopulateRoutingTables()` 安装全局寻路功能
+ 使用 `Simulator::Run(), Simulator::Destroy()` 开始模拟，在模拟结束后释放资源
+ 结束有两种可能，到达使用 `Simulator::Stop(Seconds(...))` 预先设置的结束时间(一般用于持续性应用的终止)，或事件队列为空
## 基本网络创建
### point to point (p2p)
```c++
PointToPointHelper pointToPoint;
pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

NetDeviceContainer p2pDevices;
p2pDevices = pointToPoint.Install(p2pNodes);
```
### csma
```c++
CsmaHelper csma;
csma.SetChannelAttribute("DataRate", StringValue("100Mbps"));
csma.SetChannelAttribute("Delay", TimeValue(NanoSeconds(6560)));

NetDeviceContainer csmaDevices;
csmaDevices = csma.Install(csmaNodes);
```

### wifi
```c++
// 创建物理层
YansWifiChannelHelper channel = YansWifiChannelHelper::Default();
YansWifiPhyHelper phy = YansWifiPhyHelper::Default();
phy.SetChannel(channel.Create());

// 创建mac层
WifiMacHelper mac;
Ssid ssid = Ssid("ns-3-ssid");

WifiHelper wifi;

// 为Sta节点设置mac层
NetDeviceContainer staDevices
mac.SetType("ns3::StaWifiMac",
            "Ssid", SsidValue(ssid),
            "ActiveProbing", BooleanValue(false));//Sta不自动探测AP，只被动接收AP信号

NetDeviceContainer staDevices;
staDevices = wifi.Install(phy, mac, wifiStaNodes);

// 为AP节点设置mac层
mac.SetType("ns3::ApWifiMac",
            "Ssid", SsidValue(ssid));

NetDevliiceContainer apDevices;
apDevices = wifi.Install(phy, mac, wifiApNode);

// 设置可移动性
MobilityHelper mobility;
mobility.SetPositionAllocator(...);
mobility.SetMobilityModel(...);
mobility.Install(...);
```

+ [!] 注意要设置模拟结束时间，否则 AP 会一直发送信号，无法自动停止
### 队列
+ 流量控制层 (traffic control layer) 和网卡层 (netdevice layer) 都可以使用不同的队列方式
+ ns3 默认的队列方式和大小不会随 bw, delay 变化，可使用 BQL
+ 可用的队列类型，修改队列类型和大小，使用 BQL 详见 [ns3-tutorial 7.4.1/7.4.2](https://www.nsnam.org/docs/tutorial/html/building-topologies.html#available-queueing-models-in-ns3)

## 2 日志 (Logging)
### 2.1 等级
![[ns_log_level.png]]

+ 一般使用 `LOG_INFO, LOG_FUNCTION`
+ 使用 `LOG_LEVEL_XX` 表示该等级且以上所有等级，如 `LOG_LEVEL_INFO` 包括 4 个等级
### 2.2 输出
+ 一个 `.cc` 文件中使用 `NS_LOG_COMPONENT_DEFINE("xxx")` 定义一个 xxx 的日志模块
+ 使用 `NS_LOG_XX("...")` 表示在本文件的日志模块中输出 XX 等级的日志，`NS_LOG_UNCOND` 表示无条件输出
### 2.3 开关
+ **代码**：打开指定模块指定等级的日志输出

```c++
LogComponentEnable("UdpEchoClientApplication", LOG_LEVEL_INFO);
``` 

+ **命令行**：打开指定模块指定等级的日志输出，冒号用于分隔环境变量中多个值

```bash
export 'NS_LOG=UdpEchoClientApplication=level_all|prefix_func: UdpEchoServerApplication=level_all|prefix_func'
```

+ [*] 可使用以下代码输出所有模块所有等级的日志，便于调试时排查问题

```bash
export 'NS_LOG=*=level_all|prefix_func|prefix_time'
./ns3 run scratch/myfirst > log.out 2>&1
```

## 3 命令行控制
### 3.1 打开命令行控制
```c++
CommandLine cmd;
cmd.Parse(argc, argv);
```

使用 `./ns3 run "scratch/myfirst --PrintHelp"` 可打开命令行帮助界面

```
myfirst [General Arguments]
General Arguments:
  --PrintGlobals:              Print the list of globals.
  --PrintGroups:               Print the list of groups.
  --PrintGroup=[group]:        Print all TypeIds of group.
  --PrintTypeIds:              Print all TypeIds.
  --PrintAttributes=[typeid]:  Print all attributes of typeid.
  --PrintVersion:              Print the ns-3 version.
  --PrintHelp:                 Print this help message.
```
### 3.2 属性值查询/修改
+ 查询特定类的所有属性值 (可用 PrintGroups 显示 groups, 用 pringGroup 显示 typeid, 再用 typeid 查询属性值)

```bash
./ns3 run "scratch/myfirst --PrintAttributes=ns3::PointToPointNetDevice"
```

+ 修改特定类的<mark style="background: #FF5582A6;">默认</mark>属性值

```bash
./ns3 run "scratch/myfirst
  --ns3::PointToPointNetDevice::DataRate=5Mbps
  --ns3::PointToPointChannel::Delay=2ms
  --ns3::UdpEchoClient::MaxPackets=2"
```

### 3.3 命令行注入参数 (Ho<mark style="background: #FF5582A6;"></mark>oking)
- [*] 该功能十分适合交互/调试

```c++
CommandLine cmd;
cmd.AddValue("nPackets", "Number of packets to echo", nPackets);
cmd.Parse(argc, argv);
...
echoClient.SetAttribute("MaxPackets", UintegerValue(nPackets));
```

在命令行帮助界面会显示

```
[Program Options] [General Arguments]

Program Options:
  --nPackets:  Number of packets to echo [1]
...
```

此时使用如下命令即可注入参数

```bash
./ns3 run "scratch/myfirst --nPackets=2"
```

## 4 包的追踪系统
### 概念
+ 分为 tracing source 和 tracing sink, 可使用不同的 tracing sink 达到不同的追踪样式（还可自定义 sink）
### tracing sink
+ ASCII 追踪, 输出到文件 `myfirst.tr`, 格式详见 [ns3-tutorial: 6.3.1.1](https://www.nsnam.org/docs/tutorial/html/tweaking.html#parsing-ascii-traces)

```c++
AsciiTraceHelper ascii;
pointToPoint.EnableAsciiAll(ascii.CreateFileStream("myfirst.tr"));
```

+ PCAP (packet capture) 追踪，输出到若干 `.pcap` 文件，可用 tcpdump 或 Wireshark 进行解析

```c++
// 不需要添加后缀为myfirst.pcap，对所有节点的所有设备进行追踪
pointToPoint.EnablePcapAll("myfirst");

// 对特定节点的0号设备进行追踪，使用 promiscuous mode，即任何收到的包(无论是否被接收)都记录
csma.EnablePcap("second", csmaNodes.Get(nCsma)->GetId(), 0, true); 

// 对特定设备进行追踪
csma.EnablePcap("second", csmaDevices.Get(1), true);
```

- [ ] 自定义 tracing sink 并连接到 tracing source

# 备注
+ ns3 可由用户灵活配置，构建的模型可能在现实中不能存在，ns3 并未添加一些与现实相关的内在限制