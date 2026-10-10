# 组合与能力初探

DeviceManager:
    组合：
        sdbusplus::async::context&                                                  获得利用异步上下文能力
        entity_manager::EntityManagerInterface                                      获得从Entity Manager中获得inventory info的能力
        EventIntf::Events                                                           获得Events的产生事件的能力
        inventory_device_map_t                                                      获得InvenrotyIntf::Device管理的能力
        port_map_t                                                                  获得SerialPortIntf管理的能力
        device_map_t                                                                获得DeviceIntf::BaseDevice管理的能力
    能力：
        processConfigAdded()                                                        创建Config的能力
        processPortAdded()                                                          创建Port的能力
        processInventoryAdded()                                                     创建Inventory的能力
        processDeviceAdded()                                                        创建Device的能力
        processConfigRemoved()                                                      删除Config的能力                                                         

EntityManagerInterface:
    组合：
        sdbusplus::async::context&                                                  获得利用异步上下文的能力
        interface_list_t = std::vector\<std::string>                                获得存储interface的能力
        Callback_t = std::function\<sdbusplus::async::task\<>(const sdbusplus::object_path&, const std::string>  获得异步任务的能力  
    能力：
        handleInvenrotyGet()                                                        从Entity Manager中获得inventory info的能力

EventIntf = phosphor::modbus::events
EventIntf::Events:
    组合：
        sdbusplus::async::context&                                                  获得利用异步上下文的能力
        event_map_t = std::map\<std::string, sdbusplus::object_path>                获得event name和log event object path映射的能力
    能力：
        generateSensorReadingEvent()                                                产生读传感器事件的能力
        generateSensorFailureEvent()                                                产生传感器失败事件的能力
        generateControllerFailureEvent()                                            产生控制器失败事件的能力
        generatePowerFaultEvent()                                                   产生电源失效事件的能力
        generateFilterFailureEvent()                                                产生过滤器失败事件的能力
        generatePumpFailureEvent()                                                  产生泵失败事件的能力
        generateFanFailureEvent()                                                   产生风扇失败事件的能力
        generateLeakDetectEvent()                                                   产生漏夜检测事件的能力

InventoryIntf = phosphor::modbus::rtu::inventory
InventoryIntf::Device:
    组合：
        sdbusplus::async::context&                                                  获得利用异步上下文的能力
        SerialPortIntf&                                                             获得读取保持寄存器的能力
        ProbeCallback=std::function\<sdbusplus::async::task<>(bool success)>        获得异步任务的能力
        std::unique_ptr\<InventoryServer>                                           获得接口增删的通知的能力
        std::chrono::seconds                                                        获得失败重启定时的能力
        bool dormant = false                                                        获得设备是否dormant的能力
        std::chrono::steady_clock::time_point dormantSince                          获得dormant state开始的能力
        bool mismatchLogged = false                                                 获得某种日志是否被记录的能力
        std::vector\<RegisterSpan>                                                  获得拥有寄存器Span的能力
    能力：
        handleProbeFailed()                                                         获得处理探测失败的能力
        checkAndClearDormant()                                                      获得清理设备dormant状态的能力
        startProbing()                                                              以固定间隔开始探测设备的能力
        probeDevice()                                                               执行一次探测的尝试的能力
        addInventoryServer()                                                        通知外部的能力

RegisterSpan:
    组合：
        startOffset                                                                 结构中某一个register元素开始的偏移
        totalSize                                                                   结构中的所有register元素的大小
        std::vector\<size_t> registerIndices                                        结构中的register列表的索引


InventoryServerType = sdbusplus::async::server_t\<
    InventoryServer,
    sdbusplus::aserver::xyz::openbmc_project::inventory::item::details::Chassis,
    sdbusplus::aserver::xyz::openbmc_project::inventory::decorator::details::Asset,
    sdbusplus::aserver::xyz::openbmc_project::association::details::Definitions>    获得server_t的接口组合能力，组合Chassis，Asset，Definitions这三个接口的能力为InventoryServerType

InventoryServer:
    组合：
        ChassisIntf = sdbusplus::aserver::xyz::openbmc_project::inventory::item::details::Chassis\<InventoryServer, InventoryServerType>
                                                                                    获得具体的D-Bus Chassis接口的能力
        AssetIntf = sdbusplus::aserver::xyz::openbmc_project::inventory::decorator::details::Asset\<InventoryServer, InventoryServerType>
                                                                                    获得具体的D-Bus Asset接口的能力
        AssocIntf = sdbusplus::aserver::xyz::openbmc_project::association::details::Definitions\<InventoryServer, InventoryServerType>
                                                                                    获得具体的D-Bus Definitions的能力
    能力：
        emit_added()                                                                 获得接口emit添加通知的能力
        emit_removed()                                                               获得接口emit移除通知的能力

SerialPortIntf = phosphor::modbus::rtu::port::BasePort
SerialPortIntf:
    组合：
        std::string                                                                 获得名字
        int fd=-1                                                                   获得文件描述符的能力
        std::unique\<ModbusIntf>                                                    获得Modbus设置属性和读取保持寄存器的能力
        sdbusplus::async::mutex                                                     获得异步锁的能力
    能力：
        readHoldingRegisters()                                                      获得读取保持寄存器的能力

ModbusIntf = phosphor::modbus::rtu
ModbusIntf
    组合：
        sdbusplus::async::context&                                                  获得利用异步上下文的能力
        int fd                                                                      获得文件描述符的能力
        uint16_t rtsDelay                                                           获得rts延迟的能力
        uint32_t baudRate = 0                                                       获得波特率的能力
        Parity parity                                                               获得校验的能力
        sdbusplus::async::fdio                                                      获得异步文件io的能力
    能力：
        writeRequest()                                                              获得write Request的能力
        readResponse()                                                              获得read Response的能力
        setProperties()                                                             获得set Properties的能力
        readHoldingRegisters()                                                      获得read holding registers的能力

DeviceIntf=phosphor::modbus::rtu::device
DeviceIntf::BaseDevice:
    组合：

# 本软件最小框架

1. 在context中，往object path注册一个object manager.

2. 在DeviceManager中注册这个context.

3. 在context中申请service name.

4. 异步执行context.

**context**

一个用于handle异步dbus操作的run-loop context：

- sdbusplus::async::details::context封装了异步dbus操作的run-loop，特别是那些使用co-routines的操作。

- 主要通过`spawn`将co-routine提供给对象进行处理，然后运行对象的`run`方法，该方法会handle所有的异步操作。

- 这个context有两个线程：

    - 调用`run`的线程，通常来自main，称为`caller`

    - 一个工作线程`worker`，负责执行异步操作

- 为了避免不必要地阻塞工作线程，只有`caller`线程会调用`sd_bus_wait`，而所有`sd_bus_process`调用都在`worker`线程中执行。

- `caller`和`worker`线程之间有一个基于条件变量的握手机制来实现这种交互。

Run-loop（运行循环）是一种常见的事件驱动编程模式，通常指一个持续运行的循环，它会不断检查并分发待处理的事件（如 I/O 就绪、定时器到期、信号到达等），然后调用对应的回调或协程来响应这些事件。

组合：
    bus_t bus                                       获得DBus总线连接能力
    event_t event_loop                              获得基于RAII的sd-event事件循环的能力
    event_source_t dbus_source                      获得将bus注册到event_loop上并触发DBus消息回调的能力
    dbus_event_handle                               获得处理DBus事件的能力

    event_loop作为底层多路复用器，dbus_source将bus的fd挂载到event_loop上。当内核报告该fd可读时，event_loop调用dbus_event_handle，后者内部可能会触发sd_bus_process并产生新的异步任务（sender）。

    execution::run_loop loop                        获得单线程FIFO调度器，接收并执行sender产生的任务的能力
    async_scope pending_tasks                       获得跟踪所有异步任务的完成情况的能力
    async_scope internal_tasks                      获得跟踪DBus/event状态的内部任务的能力
    initial_stop/final_stop                         获得用任务控制信号的能力
    spawn_watcher_running                           获得监控watcher协程是否正在运行的能力

    所有异步操作都被包装成sender，通过execution::schedule提交到loop中排队。pending_tasks和internal_tasks分别记录两类任务的生命周期，确保在停止前所有任务完成。initial_stop用于取消常规任务，final_stop用于清理内部任务。

    worker_run()                                    获得执行sd_bus_process和驱动loop的能力
    caller_run()                                    获得sd_bus_wait和最终的停止协调能力
    std::mutex lock{};
    std::condition_variable caller_wait{};          获得握手能力
    details::wait_process_completion* staged = nullptr;     
    details::wait_process_completion* pending = nullptr;
    bool wait_process_stopped = false;              获得同步原语标记sd_bus_wait阶段是否完成的能力

    caller线程调用caller_run()，执行sd_bus_wait，完成后通过条件变量通知worker，worker线程运行worker_run()，循环执行sd_bus_process处理消息，并驱动loop执行sender任务。
    两个线程通过staged/pending指针和条件变量实现精确握手：worker处理完一批消息后通知caller进入下一轮wait，避免忙等待或死锁。

    check_stop_requested()                          获得检查是否需要停止整个上下文的能力
    spawn_watcher()                                 获得启动watcher协程，监控停止信号并触发清理的能力
    spawn_complete()                                获得所有pending任务完成后执行回调的能力
    wait_for_wait_process_stopped()                 获得等待worker线程完全停止sd_bus_wait循环的能力

    外部请求停止

# 端口有关功能

## PortFactory

静态工厂，简单工厂模式。

特征在于：

1. 所有成员函数都是static无需实例化即可调用。

2. create方法根据传入的配置对象config::PortFactoryConfig返回std::unique_ptr\<BasePort>，封装了对象创建的逻辑。

3. getInterface提供可支持的接口列表，getConfig负责从外部获取配置，两者共同服务于后续的创建过程。

# base_port

1. BasePort构造函数，接收异步上下文，配置，设备路径，并用config.name初始化BasePort中的name以及用config.name初始化mutex：

构造函数首先打开设备路径获得文件描述符，在获得文件描述符且文件描述符是正确的之后，尝试获取ModbusIntf的引用，如果有问题那么就会转向runtime错误处理，最后成功了之后发出info通知串口成功创建的消息。

2. readHoldingRegisters接收设备地址，寄存器偏移，波特率，校验位，寄存器向量组，然后返回一个异步任务。

读保持寄存器功能首先对mutex声明一个异步锁护卫，然后等待异步上锁。

如果ModbusIntf设置波特率和校验位属性没有成功，那么就输出没有成功设置串口属性的error

然后异步等待ModbusIntf使用读取保持寄存器功能根据设备地址，寄存器偏移，寄存器向两组读取内容。

如果异步等待的结果有问题，那么就输出error错误。

最终异步返回异步等待的结果到上层。

3. PortMode端口支持模式RS232和RS485以及未知

4. 定义静态constexpr 有效端口模式

5. 声明配置Config的结构，包括配置名字，端口模式，波特率和rts延迟，在结构体中声明析构函数以利用C++内存管理。

6. 模板函数updateBaseConfig，类型就是上面的两个模板函数，然后返回一个bool

具体来说updateBaseConfig就是把properties中的内容分到config中。

模板功能函数HasPropertiesMembers，是一个concept，需要T是属性，

模板功能函数HasConfigMemebers，是一个concept，需要T是config


# port_factory

1. PortType支持端口模式usb以及未知

2. 结构PortFactoryConfig继承自Config，Porttype最开始初始化为未知，析构函数设置为virtual

3. PortFactory是一个类，里面有三个公共的静态方法：

其一是getInterfaces的方法，返回一个向量组的字符串即一个USB Port配置接口

第二个是getConfig方法，接收异步上下文，对象路径以及接口名称，返回类型是端口工厂配置的指针的异步任务，其中如果接口名称如果与USB Port配置接口一致，那么就异步等待返回从USBPort的对象路径getConfig，如果不一致就会异步返回一个空指针。

第三个是接收异步上下文的引用，工厂配置的引用的create方法，返回一个类型为BasePort的唯一指针，在方法中如果配置中的端口类型是端口的usb那么就创造一个类型为USBPort的对象，要不然就返回一个nullptr

# usb_port

USBPort类继承自BasePort

1. UBSPortConfig继承自PortFactoryConfig

2. 静态方法getDevicePath接收端口工厂配置的引用，从端口工厂配置转换为USB端口配置，声明一个正则表达式匹配usb总线，声明一个搜索路径，

3. USBPort的构造函数接收一个异步上下文和一个对象路径，来用异步上下文，端口工厂配置以及设备路径来初始化BasePort

4. getConfig方法接收一个异步上下文和一个对象路径。

# main

声明一个异步上下文context，并在path /xyz/openbmc_project注册context（或者说在context中注册一个path？），然后用context初始化一个deviceManager，之后context申请一个serviceName即xyz.openbmc_project.ModbusRTU，最后启动context运行。

**context实际是什么？**

本质上是一个融合了D-Bus事件处理与C++20协程的执行引擎。

核心角色相当于：

- **D-Bus的事件循环** 通过 sd_event（systemd 事件库）监听 D-Bus 总线上的消息、信号、超时等，并触发相应的回调。

- **协程调度器** 利用 std::execution::run_loop 管理一组待执行的协程（通过 spawn 提交），并在工作线程或主线程中驱动它们。

- **总线连接的管理者** 持有一个 bus_t（即 sdbusplus::bus_t，代表一个 D-Bus 连接），提供 request_name 等服务注册功能。

- **多线程协调器** 内部有一个专用工作线程（worker_thread）来执行异步任务，并通过锁和条件变量与调用者线程同步。

**要完成context的这些角色需要什么能力？**

- D-Bus连接能力：建立连接、请求服务名、收/发消息、加object manager，加match/signal

- I/O事件能力：监听sd-bus文件描述符，处理可读、定时器、signal；不然消息来了都不知道没有消费者

- 异步任务调度能力：把task\<void>/sender放到某个executor上跑，支持co_wait，挂起，恢复，按scheduler继续跑

- 生命周期/作用域能力：跟踪所有用户任务，维护系统任务，停止时等它们收尾，避免析构时还有活任务

- 停止/取消能力：外部可请求停止，任务可传播stop token，分阶段停用户任务和系统任务

- 线程协调能力：D-Bus的sd_bus_wait可能阻塞，而协程处理不应被它拖死，需要把等fd和跑任务分开

**context中实际有什么**

1. D-Bus连接

- bus_t bus

- operator bus_t&()、get_bus()、request_name()

2. sd-event / fd 事件

- event_t event_loop{}：sd-event 封装，跑 io/timer/signal source

- event_source_t dbus_source：把 sd-bus 的 fd 挂到 event loop

- static int dbus_event_handle(...)：sd-event 回调，fd 就绪后处理 dbus 事件

- 相关 event 封装还支持 add_io，add_oneshot_timer，condition 等

3. 协程 / sender 调度

- execution::run_loop loop{}：std::execution（stdexec/P2300 风格）的 run-loop，提供 scheduler

- spawn(Snd&&)：execution::starts_on(loop.get_scheduler(), ...) 把 sender 投到该 loop；spawn(task、<void>&&) 是特化

- async_scope pending_tasks{}：所有用户 spawn 的任务作用域，等全部完成才能安全停

- async_scope internal_tasks{}：dbus/sd-event 维护类任务单独放，避免和用户任务耦合。task/promise 还会带 scheduler，stop token 传播

4. 线程模型

- std::thread worker_thread{}：跑 loop、执行协程、做 sd_bus_process 类工作

- worker_run()：worker 主体

- caller_run()，run()：caller 侧负责 sd_bus_wait 类阻塞等待，再通过握手让 worker 处理；这就是 sdbusplus async 的“双线程”模型——caller 等、worker 跑

5. 停止与收尾

- std::stop_source initial_stop{}：request_stop() / stop_requested()，主要用于用户任务/主循环。

- std::stop_source final_stop{}：内部任务最终停止，配合 internal_tasks 做彻底收尾。

- check_stop_requested()：spawn 前检查，避免停了还投任务。

- 析构若还有未结束任务会抛/terminate，保证不泄漏工作。

6. 跨线程握手

- std::mutex lock + std::condition_variable caller_wait：caller 与 worker 同步

- details::wait_process_completion* staged/pending + wait_process_stopped：封装“sd_bus_wait 完成”的通知，让 worker 知道可以 process

- spawn_watcher() / spawn_watcher_running：启动一个看门任务，监控完成/停止事件

- spawn_complete()：收尾时处理作用域完成

**context是怎么运行的？**

1. 构造context对象

- 获取底层D-Bus连接所使用的文件描述符fd（这个fd是sd_bus内部用于D-Bus守护进程通信的套接字）

- 注册到事件循环

    - 将文件描述符fd添加到事件循环的监控列表中

    - 监控事件类型：EPOLLIN可读事件，即当D-Bus套接字上有数据到达时触发

    - 注册回调函数dbus_event_handle，当事件发生时被调用

    - 传递this指针作为用户数据，以便回调中能访问到context对象

dbus_source是一个用RAII包装的对象，用于管理这个注册项的生命周期，当context析构时，dbus_sensor会自动注销该事件源。

dbus_event_handle：取出self->pending，并调用complete()通知Worker线程“D-Bus事件已经就绪，可以处理”，这个handler连接着Caller线程和Worker线程。

static_cast\<目标类型>(表达式)是将表达式转化为目标类型。

2. 调用run()

内部调用caller_run()：

**为什么不使用shared_ptr来获得sdbusplus::async::context ctx？**

ctx 是一个栈上分配的 sdbusplus::async::context 对象，生命周期由 main 函数严格管理。它被传递给 manager 和 DeviceManager 时，是通过引用（或隐式转换得到的 bus_t&）传递的，这些类并不拥有 ctx，只是借用它的服务。

**为什么不使用 shared_ptr？**

所有权明确：ctx 的生命周期完全由 main 函数控制——它在 main 开始时创建，在 main 返回时销毁。所有使用者（manager、DeviceManager）的生命周期都嵌套在 main 内部，必然在 ctx 销毁前结束，不存在悬垂引用风险。不需要共享所有权。

避免不必要的开销：shared_ptr 引入堆分配、引用计数原子操作等额外成本。对于这种单一线程、单一所有者的场景，栈分配是最轻量、最高效的方式。

接口限制：context 的拷贝构造函数被 delete（代码中 context(const context&) = delete），移动构造函数也可能被删除（context(context&&) = delete）。这意味着你不能轻易将它放入 shared_ptr（需要可移动或可复制构造）。即使通过 std::make_shared 在堆上构造，也会破坏现有代码的简单性。

语义匹配：ctx.run() 是阻塞调用，它会一直运行事件循环直到被停止。整个 main 函数是同步风格的异步编程——ctx 就是“主循环”本身，放在栈上最自然。

# libsystemd

## sd-bus

### add

#### systemd/sd-bus.h

**match**

```
消息订阅。
```

它们是消息过滤器（match rule）的安装函数。无论是客户端还是服务端，都可以通过 match 规则来接收特定的消息（最常见的是接收信号）。但在实际应用中，绝大多数情况是客户端使用它们来订阅服务端发出的信号。

sd_bus_add_match：客户端/服务端告诉总线守护进程：当有匹配该表达式的信号出现时，把它转发给我。

sd_bus_add_match_async：添加异步匹配

sd_bus_match_signal：是更方便的变体，自动生成匹配字符串，且只匹配信号。

sd_bus_match_signal_async：添加异步匹配

**object_manager**

```
DBus服务端子对象发现。
```

用于实现标准接口sd_bus_add_object_manager，这个接口定义了一个方法GetManagedObjects 和两个信号 InterfacesAdded / InterfacesRemoved

作用：

- 当服务端管理一组动态子对象（例如每个 USB 设备一个对象路径）时，客户端不可能事先知道所有路径。ObjectManager 提供了一种标准方式让客户端：

1. 调用GetManagedObjects 一次性获取当前所有子对象及其接口和属性。

2. 监听 InterfacesAdded / InterfacesRemoved 信号，实时跟踪变化。

- sd_bus_add_object_manager 自动实现了 GetManagedObjects 方法，你只需在子对象增减时手动调用 sd_bus_emit_interfaces_added / sd_bus_emit_interfaces_removed 发送信号。

**sd_bus_add_node_enumerator**

```
DBus服务端动态枚举某个对象路径下的子节点。
```

作用：

协助introspection和ObjectManager正确地展示服务端的对象结构，尤其是当子对象是通过fallback机制动态创建时。

#### systemd/sd-bus-vtable.h

```
DBus服务端对象注册。
```

**object**

sd_bus_add_object：直接在object path上挂一个通用的消息处理器，不区分接口和方法。

sd_bus_add_object_vtable：为固定的对象路径注册一个完整的接口描述，包括方法、属性、信号。

**fallback**

sd_bus_add_fallback：为某个path前缀注册一个通配处理器，所有以该前缀开头的子路径都落入这个回调。

sd_bus_add_fallback_vtable：为动态子对象树注册共享的接口定义，通过find回调将路径映射到具体实例。

**filter**

sd_bus_add_filter：安装一个全局过滤器，对所有进入的消息都先经过这个回调（常用于认证、日志等通用逻辑）。

### attach

```
DBus客户端与服务端处理异步消息
```

#### systemd/sd-bus.h

sd_bus_attach_event(bus,e,priority)：将bus连接到事件循环e上。

sd_bus_detach_event(bus)：将bus从当前关联的事件循环上分离。

sd_bus_get_event(bus)：返回当前与bus关联的sd_event指针；如果没有关联，返回NULL

*事件循环是application用于处理异步IO和定时器的机制，不是D-Bus守护进程的组成部分。*

*在D-Bus中的文件描述符是D-Bus连接的套接字，事件循环就是监听这个fd上的可读事件来知道有新消息到达。*

*D-Bus事件循环如何工作？*

D-Bus的协议层次：D-Bus守护进程 + 传输层——连接层——消息层——应用层（实际上就是客户端或者服务端）

传输层使用的是UNIX域套接字，因此事件循环就是：

1. sd_bus_open_system() 内部会创建一个 Unix 域套接字，连接到 /run/dbus/system_bus_socket

2. 这个套接字的文件描述符可以通过sd_bus_get_fd(bus) 获取

3. 事件循环就是监听这个套接字fd上的可读/可写事件。

4. 当有数据到达，事件循环调用sd_bus_process()从套接字中读取并解析D-Bus消息。

### call

#### systemd/sd-bus.h

```
DBus客户端发起方法调用的核心API，对应DBus协议中的Method Call和Method Return/Error交互模式。
```

*D‑Bus 方法调用的本质*

在 D‑Bus 协议中，一次方法调用分为两步：

    客户端发送一个 METHOD_CALL 消息，指定：
        目标服务（destination）
        对象路径（path）
        接口（interface）
        方法名（member）
        参数（序列化为 D‑Bus 类型）
    服务端处理完毕后回复一个 METHOD_RETURN（成功）或 METHOD_ERROR（失败）消息。

sdbus_call 系列函数就是对这个过程的封装，屏蔽了消息构造、序列化、发送、等待回复等细节。

sd_bus_call：已有构造好的消息m，阻塞等待回复

sd_bus_call_async：	已有构造好的消息 m，不阻塞，通过回调处理回复

sd_bus_call_method：直接指定目标/路径/接口/方法/参数，自动构造消息并阻塞等待.

sd_bus_call_methodv：同上，但参数以 va_list 传入

sd_bus_call_method_async：直接指定目标/路径/接口/方法/参数，不阻塞，回调处理回复

sd_bus_call_method_asyncv：	同上，参数以 va_list 传入

### can

sd_bus_can_send：用于处理文件描述符传递的兼容性问题。

### close

```
D-Bus连接的生命周期管理。
```

sd_bus_close：终止与守护建成的传输层连接，不阻塞，立即关闭。

sd_bus_flush：确保所有待发送消息已交付给守护进程，阻塞，等待写入完成。

sd_bus_default_flush_close：便捷清理默认总线，阻塞。

### creds

```
D-Bus安全与身份认证子系统
```

### default

```
DBus连接管理的核心入口，负责建立应用程序与DBus总线守护进程之间的通信通道。

位于传输层与协议层之间，主要解决如何找到并连接到正确的总线这一问题。
```

*两种总线类型：system和session*

**默认连接与初始化**

行为：返回一个单例的 sd_bus 指针，第一次调用时会自动打开连接，后续调用返回同一实例。

在 D‑Bus 体系中的角色：这是最推荐的入门方式，适用于大多数简单程序。无需手动管理连接生命周期，只需在退出前调用 sd_bus_default_flush_close() 清理即可。

内部机制：根据 $DBUS_SESSION_BUS_ADDRESS 或硬编码的路径找到总线套接字，然后调用 sd_bus_open_* 打开连接。

sd_bus_default：根据环境变量$DBUS_SESSION_BUS_ADDRESS 或默认会话总线

sd_bus_default_user(&bus)：显式获取会话总线

sd_bus_default_system(&bus)：显式获取系统总线

**显式打开连接**

行为：每次都创建一个新的独立连接，不与任何默认连接共享。

在 D‑Bus 体系中的角色：适用于需要多个独立总线连接的程序（例如，同时作为系统总线和会话总线的客户端，且需要隔离）。

with_description 变体：为连接设置一个人类可读的描述字符串，用于调试、审计或 systemd 日志中标识该连接。

sd_bus_open(&bus);                        // 打开默认总线（同 sd_bus_default）

sd_bus_open_user(&bus);                   // 打开会话总线

sd_bus_open_system(&bus);                 // 打开系统总线

sd_bus_open_user_with_description(&bus, desc);   // 带描述

sd_bus_open_system_with_description(&bus, desc); // 带描述

**特殊场景连接**

_machine：用于连接到由 systemd-machined 管理的容器内部的总线。例如，在一个 LXC 容器内运行的服务可以通过此函数连接到宿主机上的总线（或反之），实现跨容器通信。

_remote：通过 SSH 隧道连接到远程主机的系统总线。需要远程主机上运行 dbus-daemon 且 SSH 能转发 Unix 域套接字。

在 D‑Bus 体系中的角色：扩展了 D‑Bus 的连接范围，使其不仅限于本地进程间通信，还能跨越容器和机器边界，但本质上仍然是连接到目标机器上的标准 D‑Bus 守护进程。

sd_bus_open_user_machine(&bus, machine);   // 连接到指定容器（machined）的用户总线

sd_bus_open_system_machine(&bus, machine); // 连接到指定容器的系统总线

sd_bus_open_system_remote(&bus, host);     // 通过 SSH 连接到远程主机的系统总线

### emit

```
DBus服务端主动向客户端推送事件的接口，对应DBus协议中的Signal消息。
在DBus体系中，信号是一种单向、无回复的消息，用于通知订阅者某个事件的发生。
```

**通用信号发射**

sd_bus_emit_signal(bus, path, interface, member, types, ...);

sd_bus_emit_signalv(bus, path, interface, member, types, va_list);

sd_bus_emit_signal_to(bus, destination, path, interface, member, types, ...);

sd_bus_emit_signal_tov(bus, destination, path, interface, member, types, va_list);

作用：发送一个自定义信号，由你指定接口和方法名（member）。

_to 变体：可以指定目标客户端的唯一名字（如 :1.234），实现定向信号（非广播）。

v 变体：参数以 va_list 形式传入，适合包装函数。

在 D‑Bus 体系中的定位：这是最底层的信号发射函数，相当于直接构造一个 SIGNAL 消息并发送。适用于定义了自己的信号接口的服务。

**标准接口的便利信号**

**Properties 变更通知（org.freedesktop.DBus.Properties）**

sd_bus_emit_properties_changed(bus, path, interface, name, ...);

sd_bus_emit_properties_changed_strv(bus, path, interface, char **names);

实现了标准接口的PropertiesChanged信号，是D-Bus属性机制的标配。

**Interfaces 添加/移除（ObjectManager 的一部分）**

sd_bus_emit_interfaces_added(bus, path, interface, ...);

sd_bus_emit_interfaces_added_strv(bus, path, char **interfaces);

sd_bus_emit_interfaces_removed(bus, path, interface, ...);

sd_bus_emit_interfaces_removed_strv(bus, path, char **interfaces);

这些信号属于org.freedesktop.DBus.OBjectManager接口。当服务端动态管理对象树时（例如，插入或拔出 USB 设备），通过这两个信号通知客户端子树的变化。配合 sd_bus_add_object_manager 注册的 ObjectManager 接口，客户端可以通过 GetManagedObjects 获取全量快照，然后监听这两个信号增量更新。

**Object 添加/移除（简化版 ObjectManager 信号）**

sd_bus_emit_object_added(bus, path);

sd_bus_emit_object_removed(bus, path);

### enqueue

sd_bus_enqueue_for_read：sd-bus 库提供的一个消息调度控制函数，允许你将一条已经接收到的消息重新放回本地的读取队列，以便它被再次调度和处理。这在 D‑Bus 体系中并不对应协议层面的任何操作，而是 sd-bus 库为实现异步授权、延迟处理等高级模式而设计的扩展机制。

### error

```
标准的DBus错误定义以及错误处理
```

### get

**DBus连接前期配置阶段：用于指定如何建立与对端的通信通道**

```
在 D‑Bus 体系中，它们位于传输层地址解析与连接建立之前，决定了 sd_bus_open_* 系列函数实际连接的目标。
```

sd_bus_set_address：手动指定总线地址

sd_bus_set_exec：启动一个进程作为总线对端

sd_bus_get_address：查询当前配置或已经连接的总线地址。

**配置和查询sd_bus对象的属性与状态**

### is


### message


### negotiate


### new


### path


### pending


### process


### query


### reply

### request


### send


### set


### slot


### start


### track


### wait




# deviceManager

用异步上下文context初始化了DeviceManager内部的上下文引用

