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

#### Oprations

默认情况下, osd使用两个线程来处理ops，ops在queue中排队等待处理,通常每个线程有15s超时时间和30s投诉时间

```
osd op threads = 2 # osd op处理线程数
osd disk threads = 1 # 处理后台disk IO密集型op的线程，例如scrbbing 和 snap trimming
osd op thread timeout = 15 # 我目前并不清楚这是op等待超时时间还是线程执行op超时时间
osd op complaint time = 30 # 我目前不并清楚complaint time有什么用
```

每个op都带有priority属性，不同类型的有限队列会有不同的优先级处理方式:

* 原始的Original PrioritizedQueue\(prio\) 使用令牌桶系统，当高优先级队列中有足够的token时，会先从高优先级队列出队op，如果高优先级队列token不足，就将低优先级队列中的op排入高优先级队列中。算法见[https://baike.baidu.com/item/%E4%BB%A4%E7%89%8C%E6%A1%B6%E7%AE%97%E6%B3%95](https://baike.baidu.com/item/令牌桶算法)
* WeightedPriorityQueue\(wpq\) 为每个队列赋予weight，根据Weight从所有优先级的队列中出队op，以防止某个低优先级op starving，在OSDs之间负载不均时WPQ很有帮助
* 最新的基于mClock的队列\(mclock\_opclass\)，把request归入5种类型:

  * client op: 用户发起的op
  * osd subop: 主OSD发起的op
  * snap trim: 裁剪相关op
  * pg recovery: 回复相关op
  * pg scrub: scrub相关op

  每个op 队列都有三种属性:

  * reservation: 分配的最小IOPS\(io per second\)，用来保证无论集群负载多大，该队列也不会starving
  * limitation: 分配的最大IOPS，用来保证单一队列不会用尽集群的IO
  * weight: 多余资源分配时的优先级

* mclock\_opclass的改进版，mclock\_client，除了根据请求类别区分队列，同时也会考虑不同用户。所以保证不同类型request优先级时也保证了不同用户之间的优先级

```
osd op queue = prio # osd op 所使用的queue的类型，有prio, wpq, mclokc_opclass, mclock_client
osd recovery op priority = 3
osd client op priority = 63
osd scrub priority = 5
osd snap trim priority = 5
```

#### MClock配置

有一些配置可以影响到mclock queue的运行，如果osd queue使用了mclock\_\*，则要特别注意以下因素:

* 发送给OSD的请求是根据PG id分片的，每个分片都有自己的mclock queue，这些队列并不会交互或者共享信息，较少的队列能够增加mclock算法的影响力，但是会有其他微妙的影响

```
osd_op_num_shards = 0
osd_op_num_shards_hdd = 5
and osd_op_num_shards_ssd = 8
```

* 请求是从多个queue进入到operation sequencer\(操作序列器，执行阶段使用\)中的，mclock决定自己队列中的那个操作进入序列器，对于seq中op的数量就需要权衡，一方面seq中充足的op可以让线程不需要IO等待\(异步\)就可以执行下个op，但是过多的op进入seq也会消弱mclock的影响

```
bluestore_throttle_deferred_bytes = 134217728
bluestore_throttle_cost_per_io = 0
bluestore_throttle_cost_per_io_hdd = 670000
bluestore_throttle_cost_per_io_ssd = 4000
```

* 还有其他因素:

```
osd push per object cost = 1000 # push op的cost
osd recovery max chunk = 8Mib # recovery op最大操作chunk
osd op queue mclock client op res = 1000.0
osd op queue mclock client op wgt = 500.0
osd op queue mclock cilent op lim = 1000.0
...#各种类型队列的res wgt 和 lim
```

默认情况下，osd会从普通队列中将优先操作发到严格队列，当cut off为low的时候，所有重复的op也会被发送过去，当cut off为high的时候，只会把重复的ack和更高级的包发送过去。当OSD因为重复的包负载过大时，可以将cut off设置为high

```
osd op queue cut off = low
```

#### Backfilling

当向集群中添加或移除一个OSD时，SCRUSH算法就会通过移动PG来重平衡。重平衡的操作就是backfiling，他的相关设置有

```
osd max backfills = 1 # 一个osd上最多能执行的backfilling数量
osd backfill scan min = 64
osd backfill scan max = 512 #每次backfill扫描时最多/最少object数量
osd backfill retry interval = 10.0 # 重试backfill请求的时间间隔
```

### BlueStore

每个osd的BlueStore通常管理一至三个物理存储设备，在只有一个存储设备的情况下，这个设备一般被分为两个分区:

* 一个较小的XFS文件系统的分区，用来存储OSD相关的metadata的信息，比如keyring, cluster id, id等
* 一个较大的raw的分区，由BlueStore直接管理，用来存储data，通常在$data目录有个block的符号链接指向设备

如果有多个存储设备，则可以分出一些存储服务:

* 用来存WAL\(write-ahead-log\)信息和journal的设备，在$data中的符号链接为block.wal
* 用来存KV-db数据的存储设备，BlueStore会在DB device上存放尽量多的数据来提高效率

##### Cache

Bluestore使用Cache缓存数据以提高效率，Cache会缓存三种数据，KV metadata / BlueStore metadata / BlueStore data

```
# 不同设备的cache的size大小
bluestore_cache_size = 0
bluestore_cache_size_hdd = 1*1024*1024*1024 #(1GB)
bluestore_cache_size_ssd = 3*1024*1024*1024 #(3GB)
# 不同类型数据占cache的比例
bluestore_cache_meta_ratio = 0.01
bluestore_cache_kv_ratio = 0.99
# bluestore data的ratio使用剩余部分，不需要特别设置
```

##### Checksums

在Deep scrub的时候为了比较Object的data和metadata，需要CheckSum。metadata的checksum由RocksDB负责，使用src32c算法计算，data的checksum由Bluestore直接负责，可以选择使用crc32c, xxhash32 或者xxhash64算法。

由于为了存储大量小object的checksum需要消耗较大空间，所以在client指定的情况下\(如rbd\)，bluestore会checksum一个大的block。checksum算法可以由一下方式指定:

```
# command line
ceph osd pool set <pool-name> csum_type <algorithm>
# global setting in config file
bluestore_csum_type = crc32c # none, crc32c, crc32c_16, crc32c_8, xxhash32, xxhash64
```

##### inline compression

bluestore支持在线压缩工具，如 snappy, zlib, lz4，Bluestore包含四种压缩判别模式:

* none: 永不压缩
* passive: 默认不压缩，除非op含有compressible选项
* aggressive: 默认压缩，除非op含有incompressible选项
* force: 强制压缩

```
# commandline
ceph osd pool set <pool-name> compression_algorithm <algorithm>
ceph osd pool set <pool-name> compression_mode <mode>
ceph osd pool set <pool-name> compression_required_ratio <ratio>
ceph osd pool set <pool-name> compression_min_blob_size <size>
ceph osd pool set <pool-name> compression_max_blob_size <size>
# global config in config file
bluestore compression algorithm = snappy
bluestore compression mode = none # 默认如果需要压缩，那么得通过命令行对某个pool进行指定
bluestore compression required ratio = 0.875 # 对压缩后压缩率的要求，如果压缩收益太低，则无论模式如何，都不会压缩
bluestore compression min blob size = 0 #比这个size小的不需要压缩
bluestore compression min blob size hdd = 128k 
bluestore compression min blob size ssd = 8k
bluestore compression max blob size = 0 # 比这个size大的需要分割后再压缩
bluestore compression max blob size hdd = 512k
bluestore compression max blob size ssd = 64k
```

##### SPDK 使用

spdk是inte开发的用来支持NVME标准的SSD设备读写的工具箱，如果想要在bluestore存储中使用NVMe设备，需要设置"spdk:"的前缀:

```
# get the serrial number with following command
$ lspci -vvv -d 8086:0953 | grep "Device Serial Number"
# then set in config file
bluestore block path = spdk:...
```

如果想把WAL和DB也分别部署在不同的NVMe设备上，则需要分别指定path和size:

```
bluestore_block_db_path = ""
bluestore_block_db_size = 0
bluestore_block_wal_path = ""
bluestore_block_wal_size = 0
```

否则ceph默认会将符号链接指向kernel based DB/WAL IO.

#### Filestore config

在目前使用的稳定版本\(10.x\)，ceph默认使用filestore作为ObjectStore。FileStore基于系统原生的文件系统

##### XATTRs

关于XATTR的介绍可以参考博客: [http://blog.csdn.net/ganggexiongqi/article/details/7661024](http://blog.csdn.net/ganggexiongqi/article/details/7661024)

XATTRs在文件系统中通常用来存储文件的扩展信息，比如访问控制等，Ceph利用文件系统的XATTR来存储Ceph所要使用的XATTR

```
filestore max inline xattr size = 0
filestore max inline xattr size xfs = 65536
filestore max inline xattr size btrfs = 2048
filestore max inline xattr size other = 512
filestore max inline xattrs = 0
filestore max inline xattrs xfs = 10
filestore max inline xattrs btrfs = 10
filestore max inline xattrs other = 2
```



