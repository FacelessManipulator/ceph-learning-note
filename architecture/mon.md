# Monitor's architecture

MON节点在cluster中的定位在总体架构中已有阐述，这章主要描述MON内部架构。

mon提供的服务有:

* 维持一份cluster map 的副本，并负责提供此副本给client
* 记录日志
* 认证

![](http://docs.ceph.com/docs/master/_images/ditaa-ae8fc6ae5b4014f064a0bed424507a7a247cd113.png)

