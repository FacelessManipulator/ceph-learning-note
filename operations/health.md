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

##### OSD\_OUT\_OF\_ORDER\_FULL / OSD\_FULL

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



