## ObjectStore workflow简要分析

想要理解各部分代码的功能目的，首先需要对整个存储架构的workflow有一个大致的了解，这里以bluestore为objectstore的实例，简要描述了workflow的过程

从调用objectstore create生成storage实例以后，所有IO操作基本通过Transaction来触发。

ObjectStore暴露给上层的写数据接口为queue\_transactions，因为所有写操作都要通过Transaction处理。在queue\_transactions中，进程接受四个参数

* Squencer \*posr: 指定的顺序队列，不同队列中的日志可以并行处理，队列由ObjectStore的使用者创建与维护
* vector&lt;Transaction&gt;& tls: 传入的日志集合

* TrackedOpRef op:

* ThreadPool::TPHandle \*handle:

queue\_transactions会执行如下操作：

* 准备日志：首先会通过collect\_contexts收集日志集合中所有的回调事件实体\(onreadable, ondisk, onreadablesync\)，并将他们按类别归类到三个回调事件列表中

* 设置顺序队列：检查传入posr是否指向存在的实例，如果不存在，则创建新的队列实例并将posr实体的ref指向新创建的队列\(OpSequenecer\)

* 准备：调用\_txc\_create创建新的TransContext，然后进行初始化操作

  * 获得rockdb的日志实体
  * 将tls中的事件通过\_txc\_add\_transaction函数分别添加至txc中，添加过程中不同的操作码将进行不同的处理。
  * \_txc\_calc\_cost　计算txc IO的开销，采用了简单的IO cost与数据量正相关的算法

  * \_txc\_write\_nodes　将onode写入rocketdb的transaction中

  * 



