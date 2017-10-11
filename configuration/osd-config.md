## OSD

#### basic settings

```
[osd]
osd uuid = 0
osd journal size = 1024
osd data = /var/lib/ceph/osd/$cluster-$id
osd max write size = 90
osd max object size = 134217728 #(12MB)
osd client message size cap = 52428000 #(49MB)
osd class dir = $libdir/rados-classes

```

* uuid指osd daemon的id，与cluster id不同，一般不需要手动指定，ceph会自动生成
* journal size 至少得是\(存储设备运行速度\*filestore max sync interval \* 2\)，并且一般都有专用设备ssd挂载到journal目录用来存储
* osd data: 存放osd 数据的位置，一般包含osd 的keyring， cluster\_fsid, osd 的 fsid，type, KV/wal/db/原始数据存放的block等数据，可以通过将不同的设备挂载到相应的文件上来保证隔离大小和效率等
* 


