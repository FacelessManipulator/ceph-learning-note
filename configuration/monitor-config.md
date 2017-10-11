### Monitor config

#### 初始配置

```
＃　等效于分别配置三个mon的网络参数
[mon]
        mon host = hostname1,hostname2,hostname3
        mon addr = 10.0.0.10:6789,10.0.0.11:6789,10.0.0.12:6789
        mon initial member = a,b,c

[mon.a]
        host = hostname1
        mon addr = 10.0.0.10:6789
```

* mon initial member: 指定初始化quorum的mon，用来快速建立集群
* mon host和mon addr用来指定monitor节点主机名和地址

#### data

mon的data需要通过leveldb存储数据到本地文件系统中，由于mon需要频繁的把内存中的monmap刷入硬盘，可能会影响osd的效率，因此最好不要将mon部署到osd节点上。

0.58以前旧版本的mon将直接通过文件系统存储data，新版本为了保证data ACID\(原子性/一致性/隔离性/持续性\)，通过kv store\(leveldb\)来存储data。

* mon配置中有一大堆mon warn on &lt;condition&gt;开头的，是用来在条件符合时报出HEALTH\_WARN的，default都是True，不需要更改
* mon health to clog　系列setting是定时向cluter的Log中写入cluster的健康状态，如果有定制化监控需求，可以通过修改mon health to clog interval来自定义监控间隔
* mon data: 指定mon data文件存放位置，一般除非默认文件夹有权限问题，否则不需要修改

```
mon data = /var/lib/ceph/mon/$cluster-$id

```



