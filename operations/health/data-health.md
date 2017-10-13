# Pools / Placement Group Health

##### PG\_AVAILABILTY

数据可用性降低，意味着cluster无法提供某些object的读写，通常是因为 peering, stale, incomplete, lack of active，可以通过ceph health detail查看详情，或者直接通过ceph tell &lt;pgid&gt; query 查看PG状态

##### PG\_DEGRADED / PG\_DEGRADED\_FULL

一些PG的数据冗余性降低，这意昧着replicated pool没有足够的拷贝，查错同PG\_AVAILABLILTY

##### PG\_DAMAGED / OSD\_SCRUB\_ERRORS

scrubbing过程中发现了数据不一致问题

##### TOO\_FEW\_PGS / TO\_MANY\_PGS / MANY\_OBJECTS\_PER\_PG

总的PG数量要么低于 mon\_pg\_warn\_min\_per\_osd 要么高于mon\_max\_pg\_per\_osd，也会导致每个pg的objects数量过多，前者可以通过增加每个池的pg数或者建立新的 池解决，后者可以通过添加新osd解决

##### POOL\_APP\_NOT\_ENABLED





