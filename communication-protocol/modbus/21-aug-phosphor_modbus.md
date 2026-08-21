phosphor-modbus git commit

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

