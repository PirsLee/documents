# 1 D-Bus

What is D-Bus?

A set of IPC Mechanism, including a protocol, a daemon, and a series of libs/tools.

The way to development in D-Bus

01 : interface definition.

02 : object implementation.

03 : at runtime, dbus object will expose its implemented interfaces.

# 2 D-Bus in OpenBMC

Infra in openbmc

a protocol : phosphor-dbus-interface

a lib : sdbusplus

a tool : sdbus++
