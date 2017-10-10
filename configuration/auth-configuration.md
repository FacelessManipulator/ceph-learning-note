## Cephx

Ceph的认证方式目前是cephx，他的原理在architecture中有阐述，主要是对称加密协议，因此加解密效率高开销小，根本没有关闭的必要。以下配置没必要改。

```
[global]
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```

各daemon的keyring默认为/etc/ceph/$cluster.$name.keyring，如果环境中需要配置多个ceph集群或者有其他特殊的文件管理需求，可以在\[global\]或者各个daemon下配置keyring的路径





