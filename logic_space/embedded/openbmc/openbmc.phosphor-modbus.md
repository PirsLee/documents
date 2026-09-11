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