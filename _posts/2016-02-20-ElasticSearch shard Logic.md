#Elasticsearch Shard Logic
首先我们需要对 shard 有一个概念了解
> ES 通过对 index 进行分片，从而实现了对 index 的分布式存储；在创建一个索引前，你需要事先指定这个索引的 shard 个数，一旦确定了，那么 shard 个数值是无法进行修改的。shard 也分 primary 和 replicate，我们这里讲的是 primary。

##IndicesClusterStateService
这个类负责了监听集群的索引状态变化，比如新建索引，更新索引等操作，都会触发其方法 `clusterChanged()` 调用，
###void clusterChanged(final ClusterChangedEvent event)
由于这个方法内部有一些事务性的操作，为了避免并发请求对其造成的影响，对这个方法内部的代码块，设置了同步锁。在 `clusterChanged()` 内部调用了 `applyNewOrUpdatedShards()`，创建或者更新 shard
###void applyNewOrUpdatedShards(final ClusterChangedEvent event)
遍历 routingNode 中的每一个 shardRouting，内部还有一些其他逻辑，这边就不详细说，我们直接跳到最下面一行，判断这个 shardRouting 是不是处于初始化状态，默认新建的所有 shard 的最先状态是 initializing，之后才是 started。如果是初始化的 shard，会调用 `applyInitializingShard()` 方法进行 shard 初始化
###void applyInitializingShard(final ClusterState state, final IndexMetaData indexMetaData, final ShardRouting shardRouting)
对于每一个索引，都有一个对应的 indexService 对象,所以对索引的 shard 的分配算法，是针对于一个索引的多个 shard 落到同一台机器上，如果这台机器上配置了多个数据存储目录（ES 配置中的 path.data 参数），那么会保证这些 shard 会均的分布到这台机器上的多个数据存储目录。这个方法会调用 `indexService.createShard()` 来进行 shard 的创建。
##IndexService
###synchronized IndexShard createShard(ShardRouting routing)
这个方法加了同步锁，但是仅仅是对象级别的，如果是不同的 indexService 对象，那么这个方法是可以被同时调用的。这里才真正进入了今天的主题，关于 shard logic 介绍，我们来详细说说逻辑，我们先假设 shard 的目录还未创建

1. 遍历当前 index 在这台机器上的所有 shard ，获取每一个 shard 的path，计数 path 下的不同 shard 个数，将结果保存在一个 map 对象中，关系是 Path -> Integer。
2. 计算当前 index 所在机器的硬盘剩余空间量，假设未来的 shard 会占用当前剩余空间的 5%，然后和历史平均 shard 大小进行比较，两者取最大作为单 shard 大小
3. 遍历当前机器的datapath路径（这个是在 ES 配置中的 path.data 指定的），然后计算当前这个路径的所在分区的剩余可用空间（A），由于第一步已经计算了每一个 datapath 下的 shard 个数，所以只需要这个 shard 个数乘以单 shard 大小，就是已经被预先划分磁盘容量（U），A-U 就是这个分区剩余的空间，我们肯定是希望将新的 shard 分配到磁盘容量相比较下最为宽裕的分区上，所以需要遍历所有的datapath，比较获得最优的datapath，来作为新 shard 的分配路径

***Tips***

这个 shard 分配算法属于通用算法，对于有些用户直接在 ES 启动的时候，将未来要创建的 index 一次性创建好，shard 的分布会极度不均匀，所以还是建议要创建 index 的时候再创建，而不是事先创建。
