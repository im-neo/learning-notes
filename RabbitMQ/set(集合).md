## set(集合)

### SADD、SMEMBERS、SMEMBERS

```shell
127.0.0.1:6379> SADD myset hello			# 向指定集合中添加元素
(integer) 1
127.0.0.1:6379> SADD myset neo
(integer) 1
127.0.0.1:6379> SADD myset world
(integer) 1
127.0.0.1:6379> SMEMBERS myset				# 查看集合的所有元素
1) "neo"
2) "world"
3) "hello"
127.0.0.1:6379> SISMEMBER myset hello		# 判断元素是否存在指定集合中
(integer) 1
127.0.0.1:6379> SISMEMBER myset other
(integer) 0
```

### SCARD

```shell
127.0.0.1:6379> SCARD myset			# 获取指定集合中元素的数量
(integer) 3
```

### SREM

```shell
127.0.0.1:6379> SREM myset neo		# 移除指定集合中的指定元素
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "hello"
```

### SRANDMEMBER

```shell
127.0.0.1:6379> SADD myset hello1
(integer) 1
127.0.0.1:6379> SADD myset hello2
(integer) 1
127.0.0.1:6379> SADD myset hello3
(integer) 1
127.0.0.1:6379> SADD myset hello4
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "hello3"
2) "hello2"
3) "hello4"
4) "hello1"
127.0.0.1:6379> SRANDMEMBER myset		# 随机获取指定集合中的一个元素
"hello2"
127.0.0.1:6379> SRANDMEMBER myset
"hello4"
127.0.0.1:6379> SRANDMEMBER myset 2		# 随机获取指定集合中的制定个数的元素
1) "hello2"
2) "hello3"
```

### SPOP

```shell
127.0.0.1:6379> SMEMBERS myset
1) "hello3"
2) "hello2"
3) "hello4"
4) "hello1"
127.0.0.1:6379> SPOP myset			# 随机删除一个元素
"hello3"
127.0.0.1:6379> SPOP myset 2		# 随机删除指定个数的元素
"hello2"
"hello4"
127.0.0.1:6379> SMEMBERS myset
1) "hello4"
```

### SMOVE

```shell
127.0.0.1:6379> SADD myset hello
(integer) 1
127.0.0.1:6379> SADD myset hello1
(integer) 1
127.0.0.1:6379> SADD myset hello2
(integer) 1
127.0.0.1:6379> SADD yourset your
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "hello"
2) "hello2"
3) "hello1"
127.0.0.1:6379> SMEMBERS yourset
1) "your"
127.0.0.1:6379> SMOVE myset yourset hello	# 将指定的值从一个集合移动到另一个集合
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "hello2"
2) "hello1"
127.0.0.1:6379> SMEMBERS yourset
1) "hello"
2) "your"
```

### SDIFF、SINTER、SUNION

```shell
127.0.0.1:6379> SADD key1 a
(integer) 1
127.0.0.1:6379> SADD key1 b
(integer) 1
127.0.0.1:6379> SADD key1 c
(integer) 1
127.0.0.1:6379> SADD key2 c
(integer) 1
127.0.0.1:6379> SADD key2 d
(integer) 1
127.0.0.1:6379> SADD key2 e
(integer) 1
127.0.0.1:6379> SDIFF key1 key2		# 差集
1) "a"
2) "b"
127.0.0.1:6379> SINTER key1 key2	# 交集（共同好友的实现）
1) "c"
127.0.0.1:6379> SUNION key1 key2	# 并集
1) "d"
2) "b"
3) "c"
4) "a"
5) "e"
```

> 微博：
>
> 将A用户关注的所有人放在集合`set1`中，将他的粉丝放在`set2`中，就可以实现：**共同关注**、**共同爱好**、**好友推荐**

## Hash（哈希）

> - **相当于Java里面的 Map**
> - 本质上和 String 类型没有太大区别，还是简单的 key - value

### HSET、HGET、HMSET、HMGET、HGETALL、HDEL、HLEN、HEXISTS

```shell
127.0.0.1:6379> HSET myhash name neo				# set 一个具体各 key - value
(integer) 1
127.0.0.1:6379> HGET myhash name					# 获取一个key - value
"neo"
127.0.0.1:6379> HMSET myhash high 170CM weight 60KG	# set 多个 key - value
OK
127.0.0.1:6379> HMGET myhash name high				# 获取多个key - value
1) "neo"
2) "170CM"
127.0.0.1:6379> HGETALL myhash						# 获取全部的key - value
1) "name"
2) "neo"
3) "high"
4) "170CM"
5) "weight"
6) "60KG"
127.0.0.1:6379> HDEL myhash weight					# 删除指定hash中的key，value同时也被删除
(integer) 1
127.0.0.1:6379> HGETALL myhash
1) "name"
2) "neo"
3) "high"
4) "170CM"
127.0.0.1:6379> HLEN myhash							# 获取hash表的key的数量
(integer) 2
127.0.0.1:6379> HEXISTS myhash name					# 判断key是否存在
(integer) 1
127.0.0.1:6379> HEXISTS myhash weight
(integer) 0

```

### HKEYS 、HVALS、HINCRBY、HSETNX

```shell
127.0.0.1:6379> HKEYS myhash				# 获取所有的key
1) "name"
2) "high"
127.0.0.1:6379> HVALS myhash				# 获取所有的value
1) "neo"
2) "170CM"
127.0.0.1:6379> HSET myhash age 25	
(integer) 1
127.0.0.1:6379> HINCRBY myhash age 1		# 指定增量
(integer) 26
127.0.0.1:6379> HINCRBY myhash age -1
(integer) 25
127.0.0.1:6379> HSETNX myhash like java		# 如果不存在可以设置
(integer) 1
127.0.0.1:6379> HSETNX myhash like php		# 如果存在则不可以设置
(integer) 0
```

> **HSETNX 可用于分布式锁**

## ZSET （有序集合）

### ZADD

```shell
127.0.0.1:6379> ZADD myset 1 one			# 插入一个值
(integer) 1
127.0.0.1:6379> ZADD myset 2 two 3 three	# 插入多个值
(integer) 2
127.0.0.1:6379> ZRANGE myset 0 -1
1) "one"
2) "two"
3) "three"
```

### ZRANGEBYSCORE

```shell
127.0.0.1:6379> ZADD income 2500 emily						# 添加三个用户和其收入
(integer) 1
127.0.0.1:6379> ZADD income 5000 jack
(integer) 1
127.0.0.1:6379> ZADD income 500 neo
(integer) 1
127.0.0.1:6379> ZRANGEBYSCORE income -inf +inf				# 【升序】显示全部的用户
1) "neo"
2) "emily"
3) "jack"
127.0.0.1:6379> ZREVRANGE income 0 -1						# 【降序】显示全部的用户
1) "jack"
2) "emily"
3) "neo"
127.0.0.1:6379> ZRANGEBYSCORE income -inf +inf withscores	# 显示全部的用户并附带score
1) "neo"
2) "500"
3) "emily"
4) "2500"
5) "jack"
6) "5000"
127.0.0.1:6379> ZRANGEBYSCORE income -inf 2500 withscores	# 显示工资小于2500的人员
1) "neo"
2) "500"
3) "emily"
4) "2500"
127.0.0.1:6379> ZREVRANGE income 0 -1 withscores			
1) "jack"
2) "5000"
3) "emily"
4) "2500"
5) "neo"
6) "500"
```

### ZREM、ZCARD

```shell
127.0.0.1:6379> ZRANGE income 0 -1
1) "neo"
2) "emily"
3) "jack"
127.0.0.1:6379> ZREM income neo			# 移除元素
(integer) 1
127.0.0.1:6379> ZRANGE income 0 -1
1) "emily"
2) "jack"
127.0.0.1:6379> ZCARD income			# 获取集合中的元素个数
(integer) 2
```

### ZCOUNT

```shell
127.0.0.1:6379> ZADD myset 1 a
(integer) 1
127.0.0.1:6379> ZADD myset 2 b
(integer) 1
127.0.0.1:6379> ZADD myset 3 c
(integer) 1
127.0.0.1:6379> ZCOUNT myset 1 3		# 获取指定区间的集合数量
(integer) 3
127.0.0.1:6379> ZCOUNT myset 1 2
(integer) 2
```

### 总结

- 成绩、工资排序
- 权重消息：1 - 普通消息，2 - 重要消息
- 热搜、排行榜实现