# Redis 基本操作

## 五大数据类型

###  基本操作

```shell
# 查看所有的Key
127.0.0.1:6379> keys * 
# set key
127.0.0.1:6379> set name neo
# 判断当前Key是否存在
127.0.0.1:6379> exist name
# 移除当前Key
127.0.0.1:6379> move name 1
# 设置Key的过期时间，单位：s
127.0.0.1:6379> expire name 10
# 查看当前Key的剩余时间
127.0.0.1:6379> ttl name
# 查看当前Key的类型
127.0.0.1:6379> type name
```

> Redis 命令速查：http://www.redis.cn/commands.html



### String(字符串)

#### 基本操作

```shell
127.0.0.1:6379> FLUSHALL
OK
127.0.0.1:6379> set key1 v1			# 设置值
OK
127.0.0.1:6379> get key1			# 获取值
"v1"
127.0.0.1:6379> APPEND key1 "hello"	# 追加，如果当前 key 不存在就相当于 set key
(integer) 7
127.0.0.1:6379> get key1
"v1hello"
127.0.0.1:6379> STRLEN key1			# 获取Key的长度
(integer) 7
127.0.0.1:6379> APPEND key1 ",neo"
(integer) 11
```

#### 增减操作

```shell
127.0.0.1:6379> set views 0			# 初始浏览量为 0
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> INCR views 			# 自增 1
(integer) 1
127.0.0.1:6379> INCR views
(integer) 2
127.0.0.1:6379> DECR views			# 自减1
(integer) 1
127.0.0.1:6379> DECR views
(integer) 0
127.0.0.1:6379> INCRBY views 10 	# 自增设置的步长
(integer) 10
127.0.0.1:6379> DECRBY views 5		# 自减设置的步长
(integer) 5
```

> - 计数器
> - 统计多单位的数量
> - 粉丝数
> - 阅读量

#### 范围 - substring

```shell
127.0.0.1:6379> set key1 "hello,neo"		# 设置 key1 的值
OK
127.0.0.1:6379> get key1
"hello,neo"
127.0.0.1:6379> GETRANGE key1 0 5			# 截取字符串[0,3]
"hello,"
127.0.0.1:6379> GETRANGE key1 0 -1			# 获取全部的字符串 和 get key 是一样的
"hello,neo"
```

#### 替换 - replace

```shell
127.0.0.1:6379> set key2 abcdefg
OK
127.0.0.1:6379> SETRANGE key2 1 xx			# 替换指定位置的字符串
(integer) 7
127.0.0.1:6379> get key2
"axxdefg"
```

#### setex / setnx

> setex - set with expire			# 设置过期时间
>
> setnx - set if not exist			 # 不存在再设置，在分**布式锁**中会被使用

```shell
127.0.0.1:6379> SETEX  key3 30 "hello"		# 设置 key3 的值为 hello 且 30s 后过期
OK
127.0.0.1:6379> ttl key3
(integer) 25
127.0.0.1:6379> get key3
"hello"
127.0.0.1:6379> SETNX mykey "redis"			# 如果 mykey 不存在则创建 mykey
(integer) 1
127.0.0.1:6379> ttl key3
(integer) -2
127.0.0.1:6379> keys *
1) "views"
2) "key2"
3) "mykey"
4) "key1"
127.0.0.1:6379> SETNX mykey "java"			# 如果 mykey 存在则创建失败
(integer) 0
127.0.0.1:6379> get mykey
"redis"
```

#### 批量操作

```shell
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 		# 同时设置多个值
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379> mget k1 k2 k3				# 同时获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> MSETNX k1 v1 k4 v4			# MSETNX 是原子性操作，要么全部成功，要么全部失败
(integer) 0
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
```

#### 对象操作

```shell
127.0.0.1:6379> set user:1 {name:neo,age:25}			# 推荐
OK
127.0.0.1:6379> mset user:1:name neo user:1:age 25		# 不推荐
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "neo"
2) "25"
```

#### GETSET 组合命令

```shell
127.0.0.1:6379> GETSET db redis				# 如果不存在值，则返回 nil
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> GETSET db mysql				# 如果存在值，则获取原来的值，并设置新值
"redis"
127.0.0.1:6379> get db
"mysql"
```



### List(列表)

> 在 Redis 里面，可以把 List 当成：==堆==、==栈==、==阻塞队列==
>
> 所有的 List 命令都是以==L==开头

#### LPUSH、RPUSH - 插入

```shell
127.0.0.1:6379> LPUSH list one			# 将一个或多个值插入到列表的头部（左）
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1		# 获取List中所有的值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> LRANGE list 0 1			# 通过区间获取具体的值
1) "three"
2) "two"
127.0.0.1:6379> RPUSH list zero			# 将一个或多个值插入到列表的尾部（右）
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "zero"

```

#### LPOP、RPOP - 移除

```shell
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "zero"
127.0.0.1:6379> LPOP list				# 移除列表的第一个元素
"three"
127.0.0.1:6379> RPOP list				# 移除列表的最后一个元素
"zero"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
```

#### LINDEX - 通过下标获取列表中的一个值、LLEN - 获取列表的长度

```shell
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LINDEX list 1			# 通过下标获取列表中的一个值
"one"
127.0.0.1:6379> LINDEX list 0
"two"
127.0.0.1:6379> LLEN list				# 获取列表的长度
(integer) 2
```

#### LREM - 移除列表中指定个数的value，精确匹配

```shell
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LPUSH list three
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> LREM list 1 one			# 移除列表中指定个数的value，精确匹配
(integer) 1
127.0.0.1:6379> LREM list 2 three
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
```

#### LTRIM - 通过下标截取指定长度的列表

```shell
127.0.0.1:6379> RPUSH list "hello"
(integer) 1
127.0.0.1:6379> RPUSH list "hello1"
(integer) 2
127.0.0.1:6379> RPUSH list "hello2"
(integer) 3
127.0.0.1:6379> RPUSH list "hello3"
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
2) "hello1"
3) "hello2"
4) "hello3"
127.0.0.1:6379> LTRIM list 1 2			# 通过下标截取指定长度的列表
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "hello1"
2) "hello2"
```

#### RPOPLPUSH - 将列表中的最后一个元素移动到新的列表中

```shell
127.0.0.1:6379> clear
127.0.0.1:6379> RPUSH list "hello"
(integer) 1
127.0.0.1:6379> RPUSH list "hello1"
(integer) 2
127.0.0.1:6379> RPUSH list "hello2"
(integer) 3
127.0.0.1:6379> RPOPLPUSH list otherlist		# 将列表中的最后一个元素移动到新的列表中
"hello2"
127.0.0.1:6379> LRANGE list 0 -1
1) "hello"
2) "hello1"
127.0.0.1:6379> LRANGE otherlist 0 -1
1) "hello2"
```

#### LSET - 更新列表中指定下标的值

```shell
127.0.0.1:6379> EXISTS list				# 判断指定列表是否存在
(integer) 0
127.0.0.1:6379> LSET list 0 item		# 如果不存在则更新报错
(error) ERR no such key
127.0.0.1:6379> LPUSH list value
(integer) 1
127.0.0.1:6379> LRANGE list 0 0
1) "value"
127.0.0.1:6379> LSET list 0 item		# 更新列表中指定下标的值
OK
127.0.0.1:6379> LRANGE list 0 0
1) "item"
127.0.0.1:6379> LSET list 1 other
(error) ERR index out of range
```

#### LINSERT - 在列表中指定的元素前后插入值

```shell
127.0.0.1:6379> LPUSH list hello
(integer) 1
127.0.0.1:6379> LPUSH list world
(integer) 2
127.0.0.1:6379> LINSERT list before world other		# 在列表中指定的元素前插入值
(integer) 3
127.0.0.1:6379> LRANGE list  0 -1
1) "other"
2) "world"
3) "hello"
127.0.0.1:6379> LINSERT list after hello neo		# 在列表中指定的元素后插入值
(integer) 4
127.0.0.1:6379> LRANGE list  0 -1
1) "other"
2) "world"
3) "hello"
4) "neo"
```

#### 小结

- list 实际上是一个链表， before 、after，left 、right 都可以插入值
- 如果 key 不存在，则创建新的链表
- 如果 key 存在，新增内容
- 如果移除了所有值，就相当于一个空链表，也代表不存在
- 在两边插入或者改动值，效率最高，在中间插入元素，效率会比较低
- 消息队列（`Lpush`/ `Rpop`）,栈（`Lpush `/ `Lpop`）

