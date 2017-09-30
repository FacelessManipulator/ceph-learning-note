## OS/ObjectStore

ObjectStore是对象存储的抽象类，向上提供了一些API，向下声明了一些接口函数等待具体实现。目前ObjectStore的具体实现有:

* FileStore: 比较稳定和常用的存储引擎，基于文件系统，具体内容查看章节Storage/FileStore
* BlueStore: 社区自己实现的专门支持RADOS的文件系统
* KStore: 在本地KV存储基础上实现的存储系统
* MemStore: 在内存中存储对象的文件系统

### 基本概念

#### Object

* 所有ObjectStore中的object都是命名对象，并以名字为全局唯一区分。数据结构为hobject\_t, ghobject\_t,存储在命名集合**\(coll\_t**\)之中。
* ObjectStore支持对object的创建/修改/删除以及在同一个集合中枚举的操作。其中枚举操作是以**hash\(object\_name\)**排序

* 每个object包含四个独立部分: 二进制数据，xattr，omap 头，omap 实体。其中后三项是以KV的形式存储Object 的元数据。

* ObjectStore需要实现对object的随机读写以及部分访问，对系数数据的存储是一个很好的特性，但不强制要求。

* 受限于操作系统，object的大小是有上限的，一般是100M左右

* xattr只存储少量KV数据\(&lt;64KB\)，用来保证对最近object xattr的访问是低消耗的

* omap\_header是一块全读全写的数据块。

* omap\_entries与xattr类似，但是存储在不同的位置，并且允许存储更多的数据量\(MBs\)。它的实现必须支持高效的范围查询

#### Collections

* Collections是一组object,也以名字区分\(coll\_t\)，支持有序枚举。同时，collection也有自己的xattr

#### 日志与事务

* ObjectStore是基于事务\(Transaction\)和日志的操作，来实现操作的原子性。具体的实现在ObjectStore的内部类**Transaction**中。
* 一个事务通常包含一组基本操作
* 每个事务都支持若干个回调函数，回调函数有三种类型: on\_applied\_sync, on\_applied, on\_commit.
* **on applied sync 与 on applied**在请求结果返回时被调用，on applied sync由ObjectStore主线程调用，因此要求低消耗，无锁。on applied则由专门的finisher线程执行
* **on commit **也由finnisher线程执行，不同的是它是在数据落盘以后被回调 

### 代码分析

#### 成员变量

```
class ObjectStore {
protected:
    string path;

public:
    CephContext* cct;
    struct Sequencer_impl : public RefCountedObject;
    struct Sequencer;
    struct CollectionImpl : public RefCountedObject;
    ...
}
```

**string path**为对象存放地址。

**CephContext\* cct **为Ceph集群实体的指针，该对象包含了map/config/log/socket等集群操作相关的通用实体，具体内容查看章节common/CephContext。

**struct Sequencer\_impl : public RefCountedObject **序列器的简单实现，基于

