# phosphor-dbu-interfaces

## devtool build phosphor-dbus-interfaces出现如bug所示的错误

已经修正的bug在[phosphor-dbus-interfaces: dependency issue with do_write_config task #4012](https://github.com/openbmc/openbmc/issues/4012)

如何[add a new yaml interface definition](https://lists.ozlabs.org/pipermail/openbmc/2021-January/024454.html)

---

为什么最新的代码中已经修改了bug却还是出现依赖错误呢？

原来原因出现在workspace中的append文件夹下的配方文件，devtool模式下do_unpack和do_patch本来就不该存在，那么do_write_config也不应该依赖他们，因此在devtool生成的bbappend文件中添加，覆盖掉源文件中的依赖。

```
# 在 externalsrc 模式下，do_unpack 和 do_patch 已被删除，因此清除 do_write_config 对它们的依赖
do_write_config[depends] = ""
```