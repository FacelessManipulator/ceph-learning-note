## OSD

#### basic settings

```
[osd]
osd uuid = 0
osd journal size = 1024
osd data = /var/lib/ceph/osd/$cluster-$id
osd max write size = 90
osd max object size = 134217728 #(128MB)
osd client message size cap = 524288000 #(500MB)
osd class dir = $libdir/rados-classes
```

* uuid指osd daemon的id，与cluster id不同，一般不需要手动指定，ceph会自动生成
* journal size 至少得是\(存储设备运行速度\*filestore max sync interval \* 2\)，并且一般都有专用设备ssd挂载到journal目录用来存储
* osd data: 存放osd 数据的位置，一般包含osd 的keyring， cluster\_fsid, osd 的 fsid，type, KV/wal/db/原始数据存放的block等数据，可以通过将不同的设备挂载到相应的文件上来保证隔离大小和效率等
* max write size: 限定了每个写请求的最大size，单位为MB （为什么默认只有90MB，这不是一个写请求连个大点的object都写不完么？）

* osd client message size cap: 限制了客户端消息的最大长度，在object长度等都不修改的情况下是足够的

* osd class dir: 存放RADOS类插件的位置，可以通过修改这个来部署不同版本\(自己的\)的链接库

#### Filesystem

在build或Mount 文件系统时如果有其他的需求选项，可以通过下列设置添加

```
# osd mkfs options {fs-type} = 创建<fs-type>时的附加选项
osd mkfs options xfs= -f -d agcount=24
# osd mount options {fs-type} = 挂载<fs-type>时的附加选项
osd mount options xfs = rw, noation, inode64, logbufs=8
```

#### Journal

如果不经过优化，在使用FileStore作为对象存储时，journal就存储在默认地址下，也就是和osd data存储在同一文件系统的同一设备下，最好能通过把journal挂载到ssd来提升性能。如果是bluestore作为文件存储，由于bluestore采用了WAL机制，日志写在block.wal上，并不会在journal path中创建Journal文件。

Ceph的默认journal size为0,需要自己计算并指定，较大的journal size可以保证在journal刷入data的间隔时间内，日志不会溢出导致丢失。journal size可以通过下列公式算得:

```
osd journal size = {2 * (expected throughput * filestore max sync interval)}
```

比如用了7200转HDD硬盘，最大IO速度是100MB/s，用的千兆网络，则 expected throughput选择流量瓶颈处就是硬盘IO速度100MB/s，然后乘 filesotre的最大同步时间间隔，默认是5s，最后乘2，结果是500MB即524288000

```
osd journal = /var/lib/ceph/osd/$cluster-$id/journal
osd journal size = 523288000 # (default 0)
```

#### Scrubbing

为了保证对象的多份copies之间的完整性一致性，Ceph会以PG为单位scrubbing数据。对于每个PG，Ceph首先生成一份目录，根据目录来比较目录中每个object是否完整或者丢失。Light scrubbing\(日常的轻量级洗刷\)只会检查每个Object的size和attributes。Deep scrubbing\(每周的深度洗刷）会计算每个object的检验和来保证数据完整性。

```
osd max scrubs = 1 # 同时进行的洗刷操作数量
osd scrub begin hour = 0
osd scrub end hour = 24 #这两个选项定义了scrub可以进行的窗口时间，但并不是绝对的
# 如果scrub的实际间隔超过了下面定义的max interval，那无论是否在窗口时间，ceph都会开始scrub
osd scrub during revovery = true # 执行recovery时是否运行开始scrub,正在进行的scrub不受影响，可以用来减负
osd scrub thread timeout = 60 # 超时清除scrub线程的最大时间
osd scrub finalize thread timeout # 超时清除终结线程的最大时间
osd scrub load threshold = 0.5 # scrub最大负载，超过阈值时不会进行scrub，load通过getloadavg()算得
osd scrub min interval = 86400 # (1 hour)
osd scrub max interval = 604800 # (1 day)
osd scrub chunk min = 5 # 每次scrub操作时最小数量的object store chunk
osd scrub chunk max = 25 # 最大数量的object store chunk
osd scrub sleep = 0 # 每scrub完一个chunk后sleep的时间，可以用来减少负载
osd scrub interval randomize ratio = 0.5 # 用来某个PG计算下次scrub的时间，算法为
# osd_scrub_min_interval * random(1, 1+osd_scrub_interval_randomize_ratio)
osd deep scrub stride = 524288 # deep scrub时读的字节数
```



