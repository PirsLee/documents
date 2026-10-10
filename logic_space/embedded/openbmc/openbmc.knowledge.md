# D-Bus的概念与逻辑

总线 Bus

消息 Message

对象路径 Object Path

接口 Interface

方法/信号/属性 Method/Signal/Property

总线名称 Bus Name

连接 Connection

事件循环 Eent Loop

绑定库 Binding Library

# OpenBMC管理概念与逻辑

把服务器的硬件状态抽象成D-Bus对象，用Redfish/IPMI/Web/SSH等标准接口读写这些对象，由一组松散耦合的phosphor服务做状态机、域值、日志、电源、固件等策略，最终通过Linux驱动操作I2C/GPIO/PMBus/CPLD等硬件。

服务就是要么实现DBus定义的接口，要么就是在DBus通信。

**怎么使用自定义D-Bus接口？**

1. 向总线注册接口

2. 服务端在总线上注册对象路径并实现接口

3. 客户端通过服务名称和对象路径调用接口中的方法或信号

**怎么在服务端实现自定义接口？**

1. 连接总线并占用well-known name

2. 向总线注册路径和接口及方法、属性、信号等

3. 激活接口

4. 实现行为

5. 进行事件循环

**OpenBMC怎么把内核抽象出来的硬件文件抽象成D-Bus对象的？**

实现D-Bus接口。

**OpenBMC怎么利用内核暴露出来的硬件文件的？**

1. entity-manager的原理。（接近于main之类的怎么运转的。）

```
获取well known name：
    xyz.openbmc_project.EntityManager

初始化01：
    成员变量                                    初始化方式                                                              含义
    systemBus                                   systemBus(systemBus)                                                    将传入的shared_ptr复制给成员变量，使该对象拥有对D-Bus连接的共享所有权
    objServer                                   objServer(sdbusplus::asio::object_server(systemBus, true))              临时构造一个object_server对象，然后将其移动或拷贝到成员objServer，其负责管理D-Bus对象路径及接口
    configuration                               configuration(configurationDirectories, schemaDirectory)                用配置文件目录和模式目录初始化配置管理器，它将扫描并加载JSON配置文件
    lastJson                                    lastJson(nlohmann::json::object())                                      初始化为一个空的JSON对象，用于记录上一次的完整配置快照
    systemConfiguration                         systemConfiguration(lohmann::json:object())                             初始化为空JSON对象，用于存储系统级配置
    dbus_interface                              dbus_interface(io, objServer, schemaDirectory)                          构造一个D-Bus接口辅助对象，它在io上注册方法处理器，并使用objServer暴露接口，利用shcemaDirectory验证配置
    powerStatus                                 powerStatus(*systemBus)                                                 构造电源状态监控对象，它通过D-Bus连接监听主机电源状态变化
    propertiesChangedTImer                      propertiesChangedTimer(io)                                              构造一个定时器，用于延迟执行属性变更回调，避免频繁触发

初始化02：
    在/xyz/openbmc_project/inventorty下添加一个manager，用于管理这个路径下的子树对象。

初始化03：
    在/xyz/openbmc_project/EntityManager路径上添加一个接口，接口名为xyz.openbmc_project.EntityManager，获得一个指向该接口的智能指针entityIface

初始化04：
    在接口中注册一个D-Bus方法Rescan，当远程调用这个方法时，会执行propertiesChangedCallback()，重新扫描配置文件并更新D-Bus属性。

初始化05：
    调用静态方法tryIfaceInitialize尝试正式激活该接口。如果失败，静默忽略。

初始化06：
    从configuration冲获取probeInterfaces，并据此初始化过滤器。过滤器用于决定哪些设备或接口应该被EntityManager监控和处理。

异步任务排队：
    boost::asio::post提交一个em.propertiesChangedCallback()任务进行排队，但是并不执行

使用配置缓存：
    em.handleCUrrentCOnfigurationJson()

异步运行：
    io.run()

```

结论是：
    
    EntityManager负责管理/xyz/openbmc_project/inventory路径下的对象子树，并将配置发布到xyz.openbmc_project.Configuration接口下。EM根据configurations文件来发布配置，是生产者。

2. FRU-Device

```

```

# 具体服务概念与逻辑

1. 实现接口

2. 事件循环

```
服务                                                                    实现接口                                                            openbmc中的实现
Phosphor Inventory Manager                                              Inventory                                                           phosphor-inventory-manager
Phosphor Software Manager                                               Software                                                            phosphor-code-mgmt
Phosphor Download Manager                                               Software                                                            phosphor-code-mgmt
Telemetry                                                               Telemetry                                                           telemetry
...
```

## phosphor-modbus

### main

```
在/xyz/openbmc_project路径下注册管理器：
    sdbusplus::server::manager_t manager{ctx, path}

初始化DeviceManager01：
    成员变量                                                                初始化方式                                              含义
    sdbusplus::async::context& ctx                                          ctx(ctx)                                                DeviceManager对象获得context的一个引用
    entity_manager::EntityManagerInterface entityManager                   
    EventIntf::Events events                                                events(ctx)

初始化DeviceManager02：
    向异步上下文提交entityManager。handleInventoryGet()任务。

申请well known name

启动异步事件循环
```

### 一些认识

**EntityManagerInterface**

1. EntityManagerInterface的Constructor中spawn了两个异步任务handleInventoryAdded和handleInventoryRemoved，DeviceManager的Constructor中也spawn了一个异步任务entyityManager.handleInventoryGet()，这些spawn任务的执行顺序是什么？

在事件循环队列中，执行按照FIFO顺序执行，每个任务都可能co_wait其他异步操作，然后重新排队。

2. handleInventoryGet()做了什么？

判断有没有addCallback？

声明用sdbus++工具生成的接口代码InventoryIntf，目的是为了获取InventoryIntf::namespace_path

启动一个D-Bus异步代理entityManager以用于后续的异步调用。

调用GetManagedObjects并遍历ManagedOBjectType结果。

针对每一个[objectPath, interfaceConfig]，按照遍历interfaceName:interfaceNames在interfaceConfig中存在的接口名调用addedCallback(objectPath, interfaceName)

3. addedCallback()是devicemanager中的processConfidAdded(sdbusplus::object_path&, std::string&)

coreturn co_await processPortAdded(objectPath, interfaceName)

coreturn co_await processInventoryAdded(objectPath, interfaceName)

在这里面这两个任务是互斥的，这又是为啥？

**PortFactorty**

1. PortFactory::getInterfaces()返回的是什么？

目前返回的是接口代码化之后的USBPort的接口，即"xyz.openbmc_project.Configuration.USBPort"

**DeviceFactory**

1. DeviceFactory::getInterfaces()输出 => vector\<xyz.openbmc_project.Configuration + 所有的profile的名称>


### 一个不成功的例子

对象路径如下：

root@evb-ast2600:~# busctl tree xyz.openbmc_project.EntityManager
`- /xyz
  `- /xyz/openbmc_project
    |- /xyz/openbmc_project/EntityManager
    `- /xyz/openbmc_project/inventory
      `- /xyz/openbmc_project/inventory/system
        `- /xyz/openbmc_project/inventory/system/board
          `- /xyz/openbmc_project/inventory/system/board/QemuTestBoard
            |- /xyz/openbmc_project/inventory/system/board/QemuTestBoard/QemuMockServer
            `- /xyz/openbmc_project/inventory/system/board/QemuTestBoard/QemuUSBPort

接口、方法、属性和信号如下：

root@evb-ast2600:~# busctl introspect xyz.openbmc_project.EntityManager /xyz/openbmc_project/inventory/system/board/QemuTestBoard/QemuMockServer
NAME                                             TYPE      SIGNATURE RESULT/VALUE     FLAGS       
org.freedesktop.DBus.Introspectable              interface -         -                -           
.Introspect                                      method    -         s                -           
org.freedesktop.DBus.Peer                        interface -         -                -           
.GetMachineId                                    method    -         s                -           
.Ping                                            method    -         -                -           
org.freedesktop.DBus.Properties                  interface -         -                -           
.Get                                             method    ss        v                -           
.GetAll                                          method    s         a{sv}            -           
.Set                                             method    ssv       -                -           
.PropertiesChanged                               signal    sa{sv}as  -                -           
xyz.openbmc_project.Configuration.QemuMockServer interface -         -                -           
.Address                                         property  t         1                emits-change
.Name                                            property  s         "QemuMockServer" emits-change
.PollRate                                        property  t         10               emits-change
.SerialPort                                      property  s         "QemuUSBPort"    emits-change
.Type                                            property  s         "QemuMockServer" emits-change

root@evb-ast2600:/usr/share/phosphor-modbus/profiles# cat QemuMockServer.json   
{
	"DeviceType" : "PowerMonitorModule",
	"DeviceModel" : "Artesyn7000555240000",
	"Parity" : "Even",
	"BaudRate" : 115200,
	"ProbeRegister" : {
		"Offset" : "0x8",
		"Size" : 8,
		"ExpectedValue" : "AB"
	},
	"SensorRegisters" : []
}




# D-Bus接口

定义OpenBMC的能力。

**Inventory接口中的包含是啥意思？**


**接口是怎么通过sdbusplus变成代码的？**


**变成的代码又是怎么样被使用的？**

# sdbusplus库

封装libsystemd，提供与D-Bus交互的能力。

提供sdbus++工具从YAML接口文件生成C++绑定代码描述。

# systemd

让服务以守护进程运行并进行管理。

# 事件循环

消息通信的主要方式。

## context

### DBus与Event Loop相关

```
bus_t bus;
event_source_t dbus_source;
event_t event_loop{};
bool name_requested = false;

```

**bus_t bus是怎么连接的？怎么代表连接的？连接有啥？**

怎么连接的？ => 调用new_bus()，底层调用sd_bus_open_system()：与D-Bus守护进程建立Unix Domain Socket连接 - 完成D-Bus认证握手 - 发送Hello方法调用，获取本连接的唯一名称。

连接代表什么？ => 

    bus_t对象代表一条活跃的D-Bus连接，内部持有sd_bus*指针。对象中具有：

        - 唯一名称

        - 请求名称

        - 文件描述符：用于与D-Bus守护进程通信的socket fd，可通过sd_bus_get_fd()获取

        - 消息队列：发送和接收D-Bus消息

        - 匹配规则：通过add_match()注册的信号过滤规则

**event_source_t dbus_source是sd-event事件循环源，那么这样的事件循环是怎么循环的？都需要处理哪些对象？**

event_source的本质：

    dbus_source = event_loop.add_io(bus.get_fd(), EPOLLIN, dbus_event_handle, this);

    将D-Bus连接的socket fd注册到sd-event事件循环中，监听EPOLLIN可读事件。当D-Bus socket上有数据可读时，sd-event会调用dbus_event_handler回调

sd-event事件循环怎么循环？ => 

    sd-event是基于linux epoll的通用事件循环实现。循环核心是sd_event_run()，等待直到有事件可处理，然后分派注册的handler

    一次迭代的流程为：

        sd_event_run()
            ├── sd_event_wait()      // 阻塞在 epoll_wait，等待 fd 事件或超时
            └── sd_event_dispatch()  // 按优先级依次调用就绪事件源的 handler

sd-event管理多种事件源：

    事件源类型                                          创建函数                                        用途
    I/O                                                 sd_event_add_io()                               监听fd的可读/可写事件
    定时器                                              sd_event_add_time()                              定时触发
    信号                                                sd_event_add_signal()                           接收Unix信号
    子进程                                              *_child()                                       监视子进程状态变化
    inotify                                             *_inotify()                                     监视文件系统变化
    defer                                               *_defer                                         延迟执行，在循环末尾i触发
    内存压力                                                *_memory_pressure                               监视内存压力事件

在context中主要处理I/O事件。


### 异步执行相关

```
execution::run_loop loop{};
std::thread worker_thread{};
async_scope pending_tasks{};
async_scope internal_tasks{};
std::stop_source initial_stop{};
std::stop_source final_stop{};
```


### 线程同步相关

### context涉及概念

DBus连接与事件源及事件循环

异步与协程

线程同步

**顶层概念关系？**


**线程与进程被搞迷糊了？**


**线程是基础执行单元，线程是怎么产生的？线程是怎么调度执行的？有多少线程？异步任务算什么？算线程复用执行的？**

**异步与协程**

1. 协程写异步逻辑，需要写什么东西？也就是说满足异步的充分条件是什么？通过spawn提交什么东西到事件循环中的什么地方？

协程是一种可以被挂起恢复的函数，是线程的执行流。

异步操作是非阻塞I/O，发起操作立即返回，等待事件响应。

异步操作在线程中执行。

也就是说通过协程实现异步操作。

协程只是异步操作的一种实现模式。

异步操作关键是让程序在等待I/O时不阻塞线程。

协程强调的是挂起/恢复能力，是一种语法/运行时特性。

异步任务强调的是非阻塞和并发执行，是一种编程模型。



2. sender怎么成为异步操作的？sender怎么被绑定到worker的事件循环中的？

3. 协程在worker线程中执行的具体细节是什么？遇到co_await挂起具体是怎么挂起的？挂起到哪里？

4. D-Bus操作通过bus_t发起，不阻塞worker，不阻塞指的是什么？

5. caller线程用sd_bus_wait等D-Bus fd的时候在做什么？

6. sd-event在fd可读时触发dbus_event_handle中的sd-event是关于D-Bus的事件模型？这个事件模型是怎么被用的？

7. worker调用sd_bus_process处理消息，这个函数是怎么被调用的？

8. 挂起的协程被唤醒，是通过什么唤醒的？事件循环怎么恢复它？

9. 协程等待的时候需要占用CPU吗？不占用CPU怎么知道消息来了？异步有哪些机制保证异步操作？

10. async_scope管理任务生命周期，具体是怎么管理的？stop_source控制停止，具体是怎么停止的？

# 抽象


# Linux内核

**怎么将硬件转变为用户空间的硬件文件？**

**使用I2C设备的充分条件？FRU设备？**

I2C设备必须要要满足的条件：

- 物理连接：SCL时钟线和SDA数据线

- 电气特性：逻辑电平通常3.3或5

- 协议条件：起始条件和停止条件

---

FRU Device是指那些带有FRU信息存储区的可换件，BMC可读出厂商、型号、序列号、资产信息表。

绝大多数FRU信息底层就是一颗*I2C/SMBus EEPROM*，总之信息底层收拢于FRU存储区概念。

也就是说FRU Device Service是探测这些EEPROM获取资产信息。

**物理接口与通信协议类型？**

一、芯片级/板级串行总线

|名称	|类型	|信号线数	|主要特点	|典型应用|
|---|---|---|---|---|
|I²C	|协议+接口|	2（SCL+SDA）|	同步、半双工、多主、有地址、ACK	|EEPROM、传感器、RTC、GPIO扩展|
|SMBus	|协议+接口|	2（SMBCLK+SMBDAT）|	I²C 子集，加超时和固定速率|	系统管理（FRU、电源、电池、SPD）
|SPI	|协议+接口|	4（MOSI+MISO+SCK+CS）|	同步、全双工、无地址（片选）、速度快|	ADC/DAC、显示屏、SD卡、Flash
|UART	|协议+接口|	2（TX+RX）|	异步、全双工、无时钟、需波特率一致	|GPS、蓝牙模块、串口调试
|1-Wire	|协议+接口|	1（DQ）|	半双工、单线供电+数据、地址识别	|温度传感器（DS18B20）、加密芯片
|MDIO	|协议+接口|	2（MDC+MDIO）|	类似简化I²C，管理以太网PHY|	PHY寄存器配置
|JTAG	|协议+接口|	5（TCK+TMS+TDI+TDO+TRST）|	边界扫描、调试/编程|	CPU/FPGA调试、Flash烧录
|SWD	|协议+接口|	2（SWDIO+SWCLK）|	ARM 调试接口替代JTAG	|Cortex-M调试
|I³C	|协议+接口|	2（SCL+SDA）|	I²C增强版，更高速度、动态地址、带中断	|传感器集线器、下一代I²C替代


二、外设 / 存储专用接口

| 名称 | 类型 | 信号线数 | 主要特点 | 典型应用 |
|------|------|---------|---------|---------|
| SDIO | 协议+接口 | 6~9（CMD+CLK+DAT0~DAT3等） | 支持SPI和SD模式，用于存储和WiFi模块 | SD卡、WiFi/蓝牙模块 |
| eMMC | 接口 | 8~13（CLK+CMD+DAT0~DAT7等） | 嵌入式MMC，并行/差分 | 手机/平板内置存储 |
| UFS | 协议+接口 | 差分对（2对TX/RX） | 高速串行，全双工，类似PCIe | 旗舰手机/SSD |
| NAND Flash接口（ONFI/Toggle） | 协议+接口 | 8/16位并行+控制线 | 并行访问，速度快，需ECC | SSD、U盘、嵌入式闪存 |
| DDR内存接口（DDR4/5/LPDDR） | 协议+接口 | 并行差分（数十条） | 高速同步，双倍数据率 | 电脑/手机内存 |
| HDMI | 接口 | 19针（含TMDS差分对+DDC+CEC） | 音视频传输，支持HDCP | 显示器、电视、投影 |
| DisplayPort | 接口 | 20针（含差分对+AUX+HPD） | 数字视频，可传输音频/数据 | 电脑显示器、笔记本 |
| MIPI CSI/DSI | 协议+接口 | 差分对（1~4 lane）+时钟 | 摄像头/显示串行接口 | 手机摄像头、屏幕排线 |


三、工业 / 自动化 / 仪器仪表

| 名称 | 类型 | 信号线数 | 主要特点 | 典型应用 |
|------|------|---------|---------|---------|
| RS-232 | 接口 | 3（TX+RX+GND） | 点对点异步，±12V电平，短距离 | 串口鼠标、工控设备调试 |
| RS-422 | 接口 | 4（A+B+Y+Z） | 差分平衡，多点，长距离（1200m） | 工业相机、编码器 |
| RS-485 | 接口 | 2（A+B） | 差分，半双工，多点（最多256节点） | Modbus RTU、楼宇自控 |
| Modbus | 协议 | — | 应用层，可跑在RS-232/485/TCP上 | PLC、传感器、变频器 |
| CAN / CAN FD | 协议+接口 | 2（CAN_H+CAN_L） | 差分，多主，优先级仲裁，高可靠 | 汽车、机器人、医疗 |
| LIN | 协议+接口 | 1（LIN总线） | 单线，低成本，低速（≤20kbps） | 车窗、雨刷、车门控制 |
| Profibus | 协议+接口 | 2（RS-485） | 西门子工业现场总线，令牌传递 | 工厂自动化 |
| Profinet | 协议+接口 | 以太网（RJ45） | 实时以太网，基于TCP/IP | 分布式I/O、运动控制 |
| EtherCAT | 协议+接口 | 以太网（RJ45） | 极低延迟，从站处理数据帧 | 伺服驱动器、数控机床 |
| USB | 协议+接口 | 4（VBUS+D+D-+GND） | 通用串行，热插拔，差分 | 键盘、鼠标、U盘、充电 |


四、网络 / 高速数据接口

| 名称 | 类型 | 信号线数 | 主要特点 | 典型应用 |
|------|------|---------|---------|---------|
| Ethernet（IEEE 802.3） | 协议+接口 | 8（RJ45 4对差分） | 局域网标准，速率10M~400G | 互联网、企业网络 |
| PCI Express (PCIe) | 协议+接口 | 差分对（x1/x4/x8/x16） | 高速串行，点对点，可扩展 | 显卡、NVMe SSD、网卡 |
| SATA | 协议+接口 | 7（差分对+地） | 串行ATA，热插拔 | 机械硬盘、SSD |
| SAS | 协议+接口 | 7（差分对+地） | 串行SCSI，双端口，高可靠性 | 服务器硬盘、磁盘阵列 |
| Thunderbolt | 协议+接口 | USB Type-C（复用） | 融合PCIe+DP，40Gbps | 外置显卡、高速存储 |
| Fibre Channel | 协议+接口 | 光纤/铜缆 | 高性能存储网络，2~128G | SAN存储区域网络 |
| InfiniBand | 协议+接口 | 差分对（4x/8x/12x） | 超低延迟，高带宽（HDR 200G） | HPC集群、AI训练 |


五、无线协议（无固定物理接口，但依赖射频天线）

| 名称 | 频段 | 典型速率 | 主要特点 | 典型应用 |
|------|------|---------|---------|---------|
| Wi-Fi (802.11 a/b/g/n/ac/ax/be) | 2.4/5/6 GHz | 最高数Gbps | 局域网覆盖广，兼容性好 | 家庭/办公上网 |
| Bluetooth / BLE | 2.4 GHz | 1~50 Mbps | 低功耗，短距离 | 耳机、鼠标、IoT |
| Zigbee / Thread / Matter | 2.4 GHz | 250 kbps | 低功耗mesh网络 | 智能家居传感器 |
| LoRa / NB-IoT | Sub-GHz / LTE | 几十kbps | 远距离，低功耗，窄带 | 物联网、水表抄表 |
| NFC | 13.56 MHz | 106~424 kbps | 近场（<10cm），被动通信 | 移动支付、门禁 |
| GPS / GNSS | 1.5~1.6 GHz | — | 卫星定位 | 导航、授时 |