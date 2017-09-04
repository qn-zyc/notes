
# 发现系统热点
* 使用 guava 里的 ConcurrentLinkedHashMap, 保留一定数量的key, 删除非热点key. 双链表记录最近访问的节点, 当被访问时将节点移到链表头, 超出一定大小时删除链表尾部的节点.链表的操作可以是异步的.
* 使用 redis, 设置过期时间, 做个累加器.
* 使用 `ngx.shared_dict`, 和 redis 类似.


## 统计单位时间

### 动态滑动窗口

窗口长度为2秒, 窗口被分为20个格子, 那么每个格子表示100ms内的统计. 窗口可以使用环形数组来实现.

统计这个窗口的平均值时考虑到削平毛刺qbs, 可以参考[这个链接](http://en.wikipedia.org/wiki/Moving_average?spm=5176.100239.blogcont4225.13.9G0FcZ#Exponential_moving_average)的一些公式.


# 参考
* [限流系统如何发现系统的热点](https://yq.aliyun.com/articles/4225)