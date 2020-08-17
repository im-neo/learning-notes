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

### 

