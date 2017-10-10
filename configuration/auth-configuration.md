## Cephx

Ceph的认证方式目前是cephx，他的原理在architecture中有阐述，主要是对称加密协议，因此加解密效率高开销小，根本没有关闭的必要。以下配置没必要改。

```
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```



