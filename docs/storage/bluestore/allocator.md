## OS/BLUESTORE/ALLOCATOR

Allocator.h中包含了_class Allocator _虚类作为后续实现的BitMapAllocator/StupidAllocator的基类。

```
class FreelistManager;

class Allocator {
public:
  virtual ~Allocator() {}

  virtual int reserve(uint64_t need) = 0;
  virtual void unreserve(uint64_t unused) = 0;
  virtual int64_t allocate(uint64_t want_size, uint64_t alloc_unit,
               uint64_t max_alloc_size, int64_t hint,
               AllocExtentVector *extents) = 0;

  int64_t allocate(uint64_t want_size, uint64_t alloc_unit,
           int64_t hint, AllocExtentVector *extents) {
    return allocate(want_size, alloc_unit, want_size, hint, extents);
  }

  virtual void release(uint64_t offset, uint64_t length) = 0;

  virtual void dump() = 0;

  virtual void init_add_free(uint64_t offset, uint64_t length) = 0;
  virtual void init_rm_free(uint64_t offset, uint64_t length) = 0;

  virtual uint64_t get_free() = 0;

  virtual void shutdown() = 0;
  static Allocator *create(CephContext* cct, string type, int64_t size, int64_t block_size);
};
```

* static Allocator \*create ，工厂模式中创建不同类型子类实例的接口，cct接受ceph实例，type接受将要实例化的allocator的类型，size与block size是初始化BitMapAllocator实例所需要的参数，前者表示设备总存储大小，后者表示设备中块的大小。
* allocate是申请内存的接口，



