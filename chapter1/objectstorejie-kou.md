## OS/ObjectStore

```
class ObjectStore {
protected:
    string path;

public:
    CephContext* cct;
```

ObjectStore是对象存储的抽象类，向上提供了一些API，向下声明了一些接口函数等待具体实现。

CephContext\* cct存放了Ceph集群实体的地址，该对象包含了map/config/log/socket等集群操作相关的通用实体，具体内容查看章节common/CephContext。



