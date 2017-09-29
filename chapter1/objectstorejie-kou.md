## OS/ObjectStore

ObjectStore是对象存储的抽象类，向上提供了一些API，向下声明了一些接口函数等待具体实现。目前ObjectStore的具体实现有:

* FileStore: 比较稳定和常用的存储引擎，基于文件系统，具体内容查看章节Storage/FileStore
* BlueStore: 社区自己实现的专门支持RADOS的文件系统
* KStore: 在本地KV存储基础上实现的存储系统
* MemStore: 在内存中存储对象的文件系统

### 基本概念

#### Object

* object的metadata，用来记录对象data以外的属性，以KV形式存储，有xattr和omap两种实现
* ObjectStore是基于事务和日志的操作，来实现操作的原子性。具体的实现在ObjectStore的内部类**Transaction**中。

```
class ObjectStore {
protected:
    string path;

public:
    CephContext* cct;
    struct Sequencer_impl : public RefCountedObject;
    struct Sequencer;
    struct CollectionImpl : public RefCountedObject;
    class Transaction;
    ...
}
```

**string path**为对象存放地址。

**CephContext\* cct **为Ceph集群实体的指针，该对象包含了map/config/log/socket等集群操作相关的通用实体，具体内容查看章节common/CephContext。

**struct Sequencer\_impl : public RefCountedObject **序列器的简单实现，基于

