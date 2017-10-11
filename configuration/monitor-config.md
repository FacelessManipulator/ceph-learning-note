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





