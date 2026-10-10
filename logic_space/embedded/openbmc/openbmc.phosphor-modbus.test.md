# 使用phosphor-modbus中的test server进行测试时的配置

成功于202608，再次参考于202609

## EntityManager的测试参考配置

root@evb-ast2600:/etc/entity-manager/configurations# cat qemu-modbus.json 
{
	"Name": "QemuTestModbusBoard",
	"Type": "Board",
	"Probe": "TRUE",
	"Exposes": [
		{
			"Name": "QemuPort",
			"Type": "USBPort",
			"DeviceAddress": "1e6a1000",
			"DeviceInterface": 0,
			"DeviceLocation": "1",
			"Port": 0,
			"Mode": "RS485",
			"BaudRate": 115200,
			"RTSDelay": 0
		},
		{
			"Name": "MockDevice",
			"Type": "DeltaBBUBC100AE000PowerMonitorModule",
			"Address": "0x01",
			"SerialPort": "QemuPort",
			"PollRate": 10
		}
	]
}

## phosphor-modbus的测试参考配置

root@evb-ast2600:/usr/share/phosphor-modbus/profiles# cat MockPowerDevice.json 
{
    "DeviceType": "PowerMonitorModule",
    "DeviceModel": "DeltaBBUBC100AE000",
    "Parity": "Even",
    "BaudRate": 115200,
    "ProbeRegister": {
        "Offset": "0x8",
        "Size": 8,
        "ExpectedValue": "AB"
    },
    "SensorRegisters": []
}

## busctl 

root@evb-ast2600:/usr/share/phosphor-modbus/profiles# busctl tree xyz.openbmc_project.EntityManager
`- /xyz
  `- /xyz/openbmc_project
    |- /xyz/openbmc_project/EntityManager
    `- /xyz/openbmc_project/inventory
      `- /xyz/openbmc_project/inventory/system
        `- /xyz/openbmc_project/inventory/system/board
          `- /xyz/openbmc_project/inventory/system/board/QemuTestModbusBoard
            |- /xyz/openbmc_project/inventory/system/board/QemuTestModbusBoard/MockDevice
            `- /xyz/openbmc_project/inventory/system/board/QemuTestModbusBoard/QemuPort

root@evb-ast2600:/usr/share/phosphor-modbus/profiles# busctl tree xyz.openbmc_project.ModbusRTU    
`- /xyz
  `- /xyz/openbmc_project
    |- /xyz/openbmc_project/inventory
    |- /xyz/openbmc_project/metric
    |- /xyz/openbmc_project/sensors
    `- /xyz/openbmc_project/software
