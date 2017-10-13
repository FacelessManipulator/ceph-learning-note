# POOL/PG/CRUSH

忘了保存了！！简单还原版本:

* pg数量的设置通常建议每个osd上包含100左右pg，可以通过 100\*osd num/replicated num算出

```
osd pool default size = 3
osd pool default min size = 2 # degraded状态下允许的最小副本数

osd pool default pg num = 100 # 假设只有3个osd，(100*3)/3=100
osd pool default pgp num = 100 # 不知作用，暂时值同pg num
```



