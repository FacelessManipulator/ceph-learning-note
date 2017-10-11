## Interaction between mon and osd

#### OSD之间的心跳检测

每个osd都会每6s检测一下相邻其他osd的心跳，如果邻居OSD超过20s未显示心跳，则就将邻居osd标记为down，并汇报给monitor，mon经过仲裁后就会更新ceph的cluster map。

```
# 每6秒进行心跳检测
osd heartbeat interval = 6
# 超过20s邻居未返应出心跳就标记down
osd hearbeat grace = 20
```



