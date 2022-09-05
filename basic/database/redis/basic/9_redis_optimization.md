## bigkey 问题

> 如果一个 key 对应的 value 所占用的内存比较大，那这个 key 就可以看作是 bigkey。

`bigkey` 除了会消耗更多的内存空间，对性能也会有比较大的影响，应尽量避免写入 bigkey。

可以**使用 Redis 自带的 `--bigkeys` 参数来查找bigkey**。

## 大量 key 集中过期问题

两种常见的方法：

1. 给 key **设置随机过期时间**。
2. 开启 `lazy-free`（惰性删除/延迟释放） 。lazy-free 特性是 Redis 4.0 开始引入的，指的是让 Redis 采用异步方式延迟释放 key 使用的内存，将该操作交给单独的子线程处理，避免阻塞主线程。