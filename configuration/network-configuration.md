## Ceph cluster Network configuration reference

由于client/mon/osd三者之间都是直接通信，因此网络配置对Ceph集群至关重要。

官方推荐在Ceph集群上配置两个网络:

* Public Network: client/MON/MDS/OSD的直连网络，所有client到osd的数据传递都将通过此网络，类似于公网
* Cluster Network: OSD之间的直连网络，数据的scrub, replicate, recovery等都通过此内部网络

这种配置保证了集群的效率以及安全性，具备防范一定Dos攻击的能力\(当public network阻塞时，通信于内部网络的状态更新和数据一致性保持等功能仍能正常运行\)

### selinux/iptables

绑定端口之前一定得先配置安全组策略，mon服务默认监听6789端口，osd服务端口一般为6800:7300，mds服务通常也在6800:7300之间，因此首先运行

```
iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT
iptables -A INPUT -i {iface} -m multiport -p tcp -s {ip-address}/{netmask} --dport 6800:7300 -j ACCEPT
```

### Config File

网络配置写在Ceph config file的**\[global\]**部分，一旦配置了cluster network，osd就会把互相之间的心跳检测/数据备份/恢复的流量放在cluster network上

```
[global]
        # ... elided configuration
        public network = {public-network/netmask}
        cluster network = {cluster-network/netmask}
```

Ceph配置中通常需要显式地指定各个服务的ip以及mon服务的port，对于osd可以不指定ip地址，Ceph 的Configuration模块会自动指定ip地址

```
[mon.a]
        host = {hostname}
        mon addr = {ip-address}:6789

[osd.0]
        host = {hostname}        
        ＃ 可以不指定
        public addr = {host-public-ip-address}
        cluster addr = {host-cluster-ip-address}
```

* public network: 在\[global\]中指定，用来显示指定public network的地址/掩码，如果不设置则将由ceph自动选择
* public addr: 在\[$type.$id\]中指定，用来局部重载public network配置
* cluster network 与 cluster addr 用法类似



Ref:

\[1\] [http://docs.ceph.com/docs/master/rados/configuration/network-config-ref/](http://docs.ceph.com/docs/master/rados/configuration/network-config-ref/ "Ref: Ceph documents")

