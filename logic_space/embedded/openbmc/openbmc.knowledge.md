# 一些基本条目



# 一些基础结构


# 一些固定规则


# OpenBMC的核心

## D-Bus

OpenBMC中的每个服务都是D-Bus上的一个“对象”。

## Systemd

OpenBMC中的每个服务都是通过systemd机制成为一个daemon的。

绝大多数的daemon结构都是：连接D-Bus——发布/订阅接口——响应事件。

## phosphor-dbus-interfaces

## phosphor-state-manager

状态机+systemd target编排

## dbus-sensors

硬件——D-Bus的桥梁

## phosphor-hwmon

硬件——D-Bus的桥梁

## entity-manager

## phosphor-modbus

## phosphor-pid-control

D-Bus消费者+执行器控制