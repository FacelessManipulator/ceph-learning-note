## OS/BlueStore/BlockDevice

BlockDevice.h中包含了_class \_BlockDevice_ \_虚类作为后续实现KernelDevice/NVMEDevice/PMEMDevice的基类。

* KernelDevice指利用Linux kernel自带的模块来访问存储设备
* NVMEDevice 指使用非易失性内存主机控制接口规范（Non-Volatile Memory Host Controller Interface Specification）来访问非易失性设备
* PMEMDevice指持久性记忆设备，ceph通过调用libpmem来操作存储



