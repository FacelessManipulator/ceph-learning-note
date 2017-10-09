## OS/BlueStore/bluestore\_types

bluestore\_types中定义了大部分bluestore的通用类以及数据结构。

```
struct bluestore_bdev_label_t {
  uuid_d osd_uuid;     ///< osd uuid
  uint64_t size;       ///< device size
  utime_t btime;       ///< birth time
  string description;  ///< device description

  void encode(bufferlist& bl) const;
  void decode(bufferlist::iterator& p);
  void dump(Formatter *f) const;
  static void generate_test_instances(list<bluestore_bdev_label_t*>& o);
};
```

* struct bluestore\_bdev\_label\_t 使用来存储块设备的基本信息
* uuid : osd的uuid信息，size: 设备大小，btime: osd创建时间，description: 设备描述



