## Cephx

Ceph的认证方式目前是cephx，他的原理在architecture中有阐述，主要是对称加密协议，因此加解密效率高开销小，根本没有关闭的必要。以下配置没必要改。

```
[global]
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```

client的keyring默认为/etc/ceph/$cluster.$name.keyring，如果环境中需要配置多个ceph集群或者有其他特殊的文件管理需求，可以在\[global\]或者各个daemon下配置keyring的路径。

daemon的keyring默认路径为$data/keyring，比如osd.0的默认路径为/var/lib/ceph/osd/ceph-0/keyring，这些keyring不建议重新指定。

### Signature

为了保持向下兼容，让不同版本的daemon能够运行在同一集群中，cephx默认

```

# 所有包需要签名
cephx require signatures = false
# 集群内daemon之间的交互包需要签名
cephx cluster require signatures = false
# client与cluster之间的交互需要签名
cephx service require signatures = false
# 只有双方版本支持的情况下才需要签名
cephx sign messages = true

```



