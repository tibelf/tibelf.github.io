# MySQL Connection Memory
针对每一个 MySQL 的连接，会占用 MySQL 实例的内存空间，占用的内存空间大小与以下的这些参数相关。

Global 和 Session 的区别在于

* Global：代表的是默认值
* Session：代表的是可以针对每一个 client 定制化，只要其连接过来的账号有权限，执行 `set` 命令即可

## sort\_buffer_size
> 针对每一个 session 连接，执行 sort 操作的时候，需要分配这个参数指定大小的内存空间
> 
> Scope：Global，Session

在 linux 平台，推荐的阈值是 256KB ~ 2MB，因为再大的话会导致内存分配速度变慢。

## read\_buffer_size
> 针对每一个线程，对 MyISAM 引擎的表进行顺序读取操作的时候，需要分配这个参数指定大小的内存空间
> 
> Scope：Global，Session

read\_buffer_size 设置的值需要是 4KB 的倍数，否则系统也会自动向下/向上取整，取最接近的 4KB 的倍数。

针对其他的引擎，这个空间会在以下几种情况下被使用

* 来执行行的 order by 操作的时候，用来缓存临时文件中的索引
* 批量插入分区
* 缓存嵌套查询的结果

## read\_rnd\_buffer_size
> Scope：Global，Session

### 使用场景
* 读取 MyISAM 引擎表的内容缓存
* 其他引擎的多范围读取优化

每一个客户端会分配一个指定大小的内存空间


## join\_buffer_size
> 在执行没有索引情况下的 join，导致全表扫描的情况下才会用到这个内存空间
> 
> Scope：Global，Session

每一个 join_buffer_size 大小的内存空间是针对两张表进行全量 join 时候使用的，如果针对复杂的 join 涉及到多张表的话，可能就会分配多个 join_buffer_size 大小的内存空间。


## thread_stack
> 每一个线程的栈大小
> 
> Scope：Global

在64位系统中的默认大小是 256KB，这个栈的大小会限制复杂的 SQL 指令，存储过程的递归深度，还有就是其他的消耗内存的操作。


## binlog\_cache_size
> 存储在事务执行过程中二进制文件的变更内容
> 
> Scope：Global

如果当实例的存储引擎支持事务并且设置打开二进制文件，会为每一个 client 分配指定大小的内存空间；

## tmp\_table_size
> 内部内存临时表
> 
> Scope：Global，Session

实际的限制会从 tmp\_table\_size 和 max\_heap\_table_size 两个值中取最小值。