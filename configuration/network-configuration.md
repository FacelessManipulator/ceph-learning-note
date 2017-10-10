### Ceph cluster Network configuration reference

由于client/mon/osd三者之间都是直接通信，因此网络配置对Ceph集群至关重要。

官方推荐在Ceph集群上配置两个网络:

* Public Network: client/MON/MDS/OSD的直连网络，所有client到osd的数据传递都将通过此网络，类似于公网
* Cluster Network: OSD之间的直连网络，数据的scrub, replicate, recovery等都通过此内部网络

这种配置保证了集群的效率以及安全性，具备防范一定Dos攻击的能力\(当public network阻塞时，通信于内部网络的状态更新和数据一致性保持等功能仍能正常运行\)

Ref:

\[1\] [http://docs.ceph.com/docs/master/rados/configuration/network-config-ref/](http://docs.ceph.com/docs/master/rados/configuration/network-config-ref/ "Ref: Ceph documents")

