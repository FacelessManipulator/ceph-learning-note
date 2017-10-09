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

  void try_aio_wake() {
    if (num_running == 1) {

      // we might have some pending IOs submitted after the check
      // as there is no lock protection for aio_submit.
      // Hence we might have false conditional trigger.
      // aio_wait has to handle that hence do not care here.
      std::lock_guard<std::mutex> l(lock);
      cond.notify_all();
      --num_running;
      assert(num_running >= 0);
    } else {
      --num_running;
    }
  }
};
```

IO实体是对aio.h中的异步IO的进一步包装

