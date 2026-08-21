phosphor-modbus git commit

# Chapter 01 : Code :: Domain model

commit 1 : Add dependencies

/subprojects/phosphor-logging.wrap
/subprojects/phosphor-dbus-interface.wrap
/subprojects/sdbusplus.wrap
meson.build : default dependency list
.clang-format : c++ code style
.clang-tidy : c++ code static analysis
/common
/rtu
meson.option : Meson Build Option Mechanism
tests/meson.build

commit 2 : modbus_rtu_lib : define read holding register

modbus_message.hpp/.cpp
Construct a message : implement Modbus’s ADU size, byte stream, push to a msg, pop from a msg, msg length, unique at one time, address/functioncode semantics field mapping, add CRC at msg tail, validate whole msg integrity, protocol consistency test, msg parse, generate CRC, search msg segment.

modbus_exception.hpp
implement : ModbusExceptionCode, ModbusException, ModbusCRCException, ModbusBadResponseException

modbus_commands.hpp/.cpp
implement ReadHoldingRegisters : ReadHoldingRegistersRequest / ReadHoldingRegistersResponse

test_modbus_commands.cpp

commit 3 : modbus_rtu_lib : implement library APIs

rtu/modbus/modbus.hpp modbus.cpp

**co_await/sdbusplus::async::task<bool>**

modbus RTU communication is a classical I/O Intensive Operation. Co-process can make _writeRequest_, _readResponse_ non-blocking suspend, waiting for hardware ready to resume execution.

**termios**

Modbus RTU physical layer depend RS-485 com port, this corresponds to physical layer configuration. 

**enum class Parity**

Modbus RTU common check mode (even, odd, none), and these enum value exactly correspond to termios

**baudRateMap**

Modbus support common baudrate mapping to Linux termios. Adapter for protocol stack and os.

# Chapter 02 : Commit History tell us business at least for phosphor-modbus

**rtu/port : add serial port interface**

/rtu/port/bas_port.hpp .cpp
/rtu/port/port_factory.hpp .cpp
/rtu/port/usb_port.hpp .cpp

**common : add entity manager interface**

/common/entity_manager_interface.hpp .cpp

**rtu/port : getCongig shall return unique_ptr** 

(without this change the values from USBPort are dropped when casting from USBPort to PortFactoryConfig)

**rtu : add mock modbus test device - no change after this commit**

**rtu : implement modbus rtu inventory source service**

/rtu/device_manager.hpp .cpp
/rtu/inventory/modbus_inventory.hpp .cpp
/rtu/xyz.openbmc_project.ModbusRTU.service

[phosphor]

Implement phosphor-modbus-rtu inventory source service based on https://gerrit.openbmc.org/c/openbmc/docs/+/77318

Entity Manager (D-Bus 配置)
   ↓
EntityManagerInterface 监听配置变化
   ↓
DeviceManager::handleInventoryGet 扫描已有配置
   ↓
inventory::Device 状态机: Probing → Active / Dormant / Restart
   ↓ (probe 成功)
InventoryServer 导出 D-Bus Inventory 对象

**common : use std::flat_map for entity_manager**

**rtu : implement modbus snesor read**

/rtu/device/base_config.hpp .cpp
/rtu/device/base_device.hpp .cpp
/rtu/device/device_factory.hpp .cpp
/rtu/device/reservoir_pump_unit.hpp .cpp

Read the modbus device config from the Entity Manager configuration and create the sensor interfaces for related sensor register config.

**add firmware inventory support**

/rtu/firmware/device_firmware.hpp .cpp

Add support to display the firmware version using the Firmware version register from EM configuration. The version will be exposed as xyz.openbmc_project.Software.Version interface on the Dbus.

**rtu: add read status register and event support**

/common/events.hpp .cpp

Add support to read the status registers and generate the appropriate events.

**add sensor related interfaces**

Add threshold, availability, operational status, association interfaces for sensor.

**rtu : sensor : Add support for all defined sensors**

Expands support to allow all sensors as defined in the D-Bus interface specification for Sensor value.

**rtu : sensor : change associations to all_sensors**

Without this, the sensors does not show up under the correct associated chassis in redfish.

**add meson option & 1 sec default for rtu timeout**

/rtu/modbus_rtu_config.hpp.in

Add meson option for rtu timeout with a default value of 1 second. This timeout will get triggered if modbus device fails to respond.

**rtu : device : add PowerMOnitorModuel device type**

/rtu/device/power_monitor_module.hpp .cpp

Add a new PowerMonitorModule device type to support Panasonic BJBPM102A0001, Artesyn 7000433970000, and Delta ECD70000020 power monitor module devices over Modbus RTU.

# Chapter 03 : How to add phosphor-modbus or other phosphor-* service to openbmc?



# Chapter 04 : How to test RS485 between AST2600 and a mcu?

1. check ttyS* at MIMO addr

dmesg | grep tty

UART 5 at 1E78:4000 --> ttyS4
VUART 1 at 1E78:7000 --> ttyS13
UART 3 at 1E78:E000 --> ttyS2
UART 4 at 1E78:F000 --> ttyS3
UART 10 at 1E79:0400 --> ttyS9
UART 11 at 1E79:0500 --> ttyS10

# Chapter 05 : How to use phosphor-modbus or other phosphor-* service to openbmc?


# Chapter 06 : How a daemon runs?

## phosphor-modbus


# Chapter 07 : How a hardware becomes D-Bus object?

## entity manager

detection daemon

fru-device : scans all available I2C buses for IPMI FRU EEPROM devices, parse IPMI FRU data and publish to D-Bus in the name of _xyz.openbmc_project.FruDevice_

# Chapter 07 : phosphor-dbus-interface


