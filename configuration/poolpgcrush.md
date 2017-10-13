# POOL/PG/CRUSH

忘了保存了！！简单还原版本:

* pg数量的设置通常建议每个osd上包含100左右pg，可以通过 100\*osd num/replicated num算出

```
osd pool default size = 3
osd pool default min size = 2 # degraded状态下允许的最小副本数

osd pool default pg num = 100 # 假设只有3个osd，(100*3)/3=100
osd pool default pgp num = 100 # 不知作用，暂时值同pg num
```

* weight的设置: 为保证负载均衡，需要为新加入的osd设置合适的weight。默认情况下是osd存储设备的size\(TB为单位\)

```
osd crush initial weight = ... # 不需要自己设置
# replicated pool 的默认ruleset，选择最小id的ruleset,一般是ruleset 0
osd pool default crush replicated ruleset = -1 (CEPH_DEFAULT_CRUSH_REPLCATED_RULESET)
```



