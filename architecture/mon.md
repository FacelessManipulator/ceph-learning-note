# Monitor's architecture

MON节点在cluster中的定位在总体架构中已有阐述，这章主要描述MON内部架构。

mon提供的服务有:

* 维持一份cluster map 的副本，并负责提供此副本给client
* 记录日志
* 认证

当cluster map的改变\(挂了一个osd\)被当前mon检测到时，mon就会把changes写入同一个Paxos实例中，然后由Paxos将Changes写入kv store来保证强一致性。

![](http://docs.ceph.com/docs/master/_images/ditaa-ae8fc6ae5b4014f064a0bed424507a7a247cd113.png)

#### Quorum

#### Cluster Map

#### Consistency

* mon通过monitor map\(monmap\)来发现其他monitor node，目的是为了时时通过paxos动态更新monmap，而其他daemon则通过config file来寻找Mon节点
* monmap的修改必须通过分布式算法Paxos以确保quorum中保有相同版本的monmap

* 


