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

一个或多个osd主机或者网络挂了

```

```



