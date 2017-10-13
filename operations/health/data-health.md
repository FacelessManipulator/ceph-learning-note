# Pools / Placement Group Health

##### PG\_AVAILABILTY

数据可用性降低，意味着cluster无法提供某些object的读写，通常是因为 peering, stale, incomplete, lack of active，可以通过ceph health detail查看详情，或者直接通过ceph tell &lt;pgid&gt; query 查看PG状态

##### PG\_DEGRADED

一些PG的数据冗余性降低，这意昧着replicated pool没有足够的拷贝，

