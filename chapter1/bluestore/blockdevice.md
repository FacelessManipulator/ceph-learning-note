## OS/BlueStore/BlockDevice

BlockDevice.h中包含了_class \_BlockDevice_ \_虚类作为后续实现KernelDevice/NVMEDevice/PMEMDevice的基类。

* KernelDevice指利用Linux kernel自带的模块来访问存储设备
* NVMEDevice 指使用非易失性内存主机控制接口规范（Non-Volatile Memory Host Controller Interface Specification）来访问非易失性设备，ceph通过spdk\(Storage Performance Development Kit\)套件提供的高性能接口来操作NVME设备
* PMEMDevice指持久性记忆设备，ceph通过调用libpmem来操作存储PMEM设备

```
struct IOContext {
private:
  std::mutex lock;
  std::condition_variable cond;

public:
  CephContext* cct;
  void *priv;
#ifdef HAVE_SPDK
  void *nvme_task_first = nullptr;
  void *nvme_task_last = nullptr;
  std::atomic_int total_nseg = {0};
#endif


  std::list<aio_t> pending_aios;    ///< not yet submitted
  std::list<aio_t> running_aios;    ///< submitting or submitted
  std::atomic_int num_pending = {0};
  std::atomic_int num_running = {0};

  explicit IOContext(CephContext* cct, void *p)
    : cct(cct), priv(p)
    {}

  // no copying
  IOContext(const IOContext& other) = delete;
  IOContext &operator=(const IOContext& other) = delete;

  bool has_pending_aios() {
    return num_pending.load();
  }

  void aio_wait();

  void try_aio_wake();
};
```

IO实体是对aio.h中的异步IO的进一步包装

* lock与cond用来协调　try\_aio\_wake　和　aio\_wait之间的同步，aio\_wait中cond.wait阻塞进程，try\_aio\_wake中cond.notify\_all通过运行中任务的计数来唤醒cond阻塞
* cond作为



