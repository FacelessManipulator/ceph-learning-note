## OS/BLUESTORE/STUPIDALLOCATOR

StupidAllocator是Allocator的stupid实现，

```
class StupidAllocator : public Allocator {
private:
  CephContext* cct;
  std::mutex lock;

  int64_t num_free;     ///< total bytes in freelist
  int64_t num_reserved; ///< reserved bytes

  typedef mempool::bluestore_alloc::pool_allocator<
    pair<const uint64_t,uint64_t>> allocator;
  std::vector<btree_interval_set<uint64_t,allocator>> free;  ///< leading-edge copy

  uint64_t last_alloc;

  unsigned _choose_bin(uint64_t len);
  void _insert_free(uint64_t offset, uint64_t len);

  uint64_t _aligned_len(
    btree_interval_set<uint64_t,allocator>::iterator p,
    uint64_t alloc_unit);
  ...
  
};
```



