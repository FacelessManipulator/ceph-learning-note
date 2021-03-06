# Health check

以MON=3 OSD=3 MDS=3的teset cluster为例，模拟各种health状态，健康状态如下:

```
$ ceph -s
  cluster:
    id:     b4c2e1a3-bec4-4c87-bc7b-abb7a17e8581
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c
    mgr: x(active)
    mds: cephfs_a-1/1/1 up  {0=b=up:active}, 2 up:standby
    osd: 3 osds: 3 up, 3 in

  data:
    pools:   3 pools, 24 pgs
    objects: 21 objects, 2.19K
    usage:   3.20G used, 27.0G / 30.2G avail
    pgs:     24 active+clean
```

### OSD health

##### OSD\_DOWN

一个或多个osd主机或者网络挂了，检查服务和网络来解决问题

```
$ systemctl stop ceph-osd@2
$ ceph -s
  cluster:
      id:     b4c2e1a3-bec4-4c87-bc7b-abb7a17e8581
      health: HEALTH_WARN
              1 osds down
              Reduced data availability: 3 pgs inactive, 17 pgs peering
              Degraded data redundancy: 17 pgs unclean

    services:
      mon: 3 daemons, quorum a,b,c
      mgr: x(active)
      mds: cephfs_a-1/1/1 up  {0=b=up:active}, 2 up:standby
      osd: 3 osds: 2 up, 3 in

    data:
      pools:   3 pools, 24 pgs
      objects: 21 objects, 2.19K
      usage:   3.20G used, 27.0G / 30.2G avail
      pgs:     70.833% pgs not active
               17 peering
               5  active+clean
               2  stale+active+clean
```

##### 

##### OSD\_&lt;CRUSH TYPE&gt;\_DOWN

Crush上一个子树挂了，比如{ OSD\_HOST\_DOWN, OSD\_ROOT\_DOWN }，通常是一个机柜断电或者网络出了问题。

##### OSD\_ORPHAN

Crush map引用了一个不存在的osd，通过ceph osd crush rm osd.&lt;id&gt;从cursh中移除osd可以解决

##### OSD\_OUT\_OF\_ORDER\_FULL / OSD\_FULL / OSD\_BACKFILLFULL / OSD\_NEARFULL

osd的使用空间到达阈值以至于不能继续某些操作，如果不是负载不均衡，则通过添加i新的存储节点或者扩大full的阈值，通过ceph df查看目前空间使用情况

```
$ ceph df
    GLOBAL:
        SIZE      AVAIL     RAW USED     %RAW USED 
        30.2G     27.0G        3.20G         10.60 
    POOLS:
        NAME                  ID     USED      %USED     MAX AVAIL     OBJECTS 
        cephfs_data_a         1          0         0         8.89G           0 
        cephfs_metadata_a     2      2.19K         0         8.89G          21 
        test                  3          0         0         8.89G           0 
$ ceph osd set-backfillfull-ratio <ratio>
$ ceph osd set-nearfull-ratio <ratio>
$ ceph osd set-full-ratio <ratio>
```

##### OSDMAP\_FLAGS

cluster的flags，这是admin手动设置的:

* full - cluster 满了，不再提供写入服务
* pauserd, pausewr - 暂停写入或读出 \# 注意
* noup - 禁止OSD启动
* nodown - 忽视OSD failure的report，这样MON就不会把OSD mark为down
* noin - 被mark out的OSD重启以后也不会被mark in
* noout - 就算心跳检测超时了osd也不会被mark out
* nobackfill, bireciver, norebalance - 关闭一些功能
* noscrub, nodeep\_scurb - 同上
* -notieragent - 同上

```
ceph osd set <flag>
ceph osd unset <flag>
# example
ceph-client $ ./writeoj.py test hc HelloWorld!
    Writing to Cluster ID: b4c2e1a3-bec4-4c87-bc7b-abb7a17e8581
ceph-client $ ceph osd set full
ceph-client $ ./writeobj.py test hc HelloWorld2!
    #这儿write请求等了很久也没返回错误码，可能是timeout很长，unset以后成功写入
ceph-client $ ceph osd unset full
    #unset以后上一个请求立刻返回0成功
```

##### OSD\_FLAGS

也可以为每个osd单独设置flag, 有noup/nodown/noin/noout

```
ceph osd add-<flag> <osd-id>
ceph osd rm-<flag> <osd-id>
```

##### OLD\_CRUSH\_\*

* OLD\_CRUSH\_TUNABLES: crush map使用了低于mon\_crush\_min\_required\_version版本的settings

* OLD\_CRUSH\_STRAW\_CALC\_VERSION: cursh使用了过时的straw weight算法

##### CACHE\_POOL\_NO\_HIT\_SET

有的Cache pool未设置hitset，导致一些旧的object无法被刷入存储并移除cache

```
ceph osd pool set <poolname> hit_set_type <type>
ceph osd pool set <poolname> hit_set_period <period-in-seconds>
ceph osd pool set <poolname> hit_set_count <number-of-hitsets>
ceph osd pool set <poolname> hit_set_fpp <target-false-positive-rate>
```

##### POOL\_FULL

有的pool存储已达阈值，不再允许写入，可以通过提高阈值或删点东西解决:

```
ceph osd pool set-quota <poolname> max_objects <num-objects>
ceph osd pool set-quota <poolname> max_bytes <num-bytes>
```



