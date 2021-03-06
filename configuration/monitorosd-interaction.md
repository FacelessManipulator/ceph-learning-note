## Interaction between MONs and OSDs

#### OSD之间的心跳检测

每个osd都会每6s检测一下相邻其他osd的心跳，如果邻居OSD超过20s未显示心跳，则就将邻居osd标记为down，并汇报给monitor，mon经过仲裁后就会更新ceph的cluster map。

```
# 每6秒进行心跳检测
osd heartbeat interval = 6
# 超过20s邻居未返应出心跳就标记down
osd hearbeat grace = 20
```

#### Report

有的时候由于一个机架的交换机出现问题，会导致整个机架的osd都不通，为了减少虚假报警\(那个机架的osd会认为其他机架的osd不通并汇报\)，ceph默认在不同层级结构之间只需要两个osd汇报就能作出决定。通过修改下面设置就能改变不同子集报错的最小数量以及子集所在的level

```
 mon osd min down reporters = 2
 mon osd reporter subtree level = rack
```

如果一个osd在尝试连接Ceph config文件中定义的任意一个osd准备peering时失败，那他就会每30s连接mon来获取最新cluster map，如果感觉cluster map变化周期长想减小public network压力，也可以通过修改下面的配置来更改时间间隔:

```
# osd 连接mon的周期
osd mon heartbeat interval = 30
# osd之间互ping的周期
osd heartbeat interval = 6
```

OSD需要不定时向MON进行汇报，否则在一段时间以后MON会将OSD标记为down。一般当osd检测到状态变化时，比如pg状态变化，并且距离上次汇报超过了最小回报时间后，就会向mon汇报，当到达最大汇报间隔时间后，无论是否有变化发生，osd也会向mon进行汇报。

```
 # Mon认为osd down的超时时间
 mon osd report timeout = 900
 # osd向mon汇报的最小间隔
 osd mon report interval min = 5 
 # osd向mon汇报的最大间隔
 osd mon report interval max = 600
```



