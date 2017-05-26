# ElasticSearch 优化
## 硬件
* Memory is the most import, there should be about half the available memory left to lucene use, because lucene relies a lot on the file system cache for performance.
* Disk, the best way is using SSD. But if you use spinning media, then the RAID 0 is the better choice.
* Network, better network
* CPU, the more cores is better than faster cpu

## 配置
* if your heap size is under 32gb and your jdk is before 6u23 release, please use -XX:UseCompressedOops
* file description bigger , 32k or even 64k
* use unicast instead of multicast for node discovery，and the data node have a static list of the master node
* Index Management, you can simply discard index 

## shard
* number_of_shards = index_size / max_shard_size. how to decide the max_shard_size, it should be get by your test, when you set one shard and write index to it, and when your query start to slow down and become too slow for you, then the max capacity on that hardware is reached. 

## Client Node(Gateway)
it is both CPU and Memory intensive
the task list is

* receive the index and query request, send it to relative data node, and scatter and gather operations

## Date Node
it is both CPU, Memory and I/O intensive

* index data

## Master Node
the task list is

* check the cluster health
* update routing table
* manage shards and replicate

## Tribe Node
The tribes feature allows a tribe node to act as a federated client across multiple clusters.

## ShardAllocator
cluster.routing.allocation.same_shard.host which will concern the primary and replicate whether is in the same physical machine, default is false, I think it should be true. But the better method is through parameter index.routing.allocation

***reference***

[Hardware Information](https://www.elastic.co/guide/en/elasticsearch/guide/current/hardware.html)

[on-elasticsearch-performance](https://blog.liip.ch/archive/2013/07/19/on-elasticsearch-performance.html)

[load-balancing-elasticsearch-clusters](https://qbox.io/blog/load-balancing-elasticsearch-clusters)

[running-elasticsearch-in-production](http://blogs.justenougharchitecture.com/running-elasticsearch-in-production/)

[modules node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)

[module gateway](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html)

[modules-tribe](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-tribe.html#modules-tribe)

[mastering-elasticsearch](http://udn.yyuap.com/doc/mastering-elasticsearch/chapter-4/43_README.html)




