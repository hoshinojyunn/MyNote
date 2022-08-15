# Redis

## 一、启动

### 1.   正常安装启动

​		安装完成后，redis脚本命令放在/usr/local/bin下，

* 启动服务端：

```shell
redis-server [配置文件路径]
```

* 启动客户端：

```shell
redis-cli -p [要连接服务端的端口号]
```

* 测试：

```shell
cli：ping
server：pong
# 测试成功
```

### 2.  docker启动

```shell
# 启动docker 容器
docker run --name my-redis -p 6379:6379 -d redis
# redis-cli连接
redis-cli -p 6379
```

注意：==docker启动的redis容器默认没有配置文件，如果需要配置文件，则自己将配置文件拷贝进容器中，再进入容器中使用：redis-server  [配置文件路径]，按配置文件启动redis==

​		docker启动的redis的数据在容器的/data目录下，最好进行卷挂载保存数据。

## 二、redis-benchmark

### 1.   概述

​		redis-benchmark是redis自带的一个压测工具脚本，也在/usr/local/bin下。

### 2.  基本参数与使用

常用参数：

| 选项 | 描述                                | 默认值    |
| ---- | ----------------------------------- | --------- |
| -h   | 指定服务器主机名                    | 127.0.0.1 |
| -p   | 指定服务器端口                      | 6379      |
| -s   | 指定服务器socket                    |           |
| -c   | 指定并发连接数                      | 50        |
| -n   | 指定请求数                          | 10000     |
| -d   | 以字节的形式指定SET/GET值的数据大小 | 2         |
| -k   | 1=keep alive，0=reconnect           | 1         |

==不同版本默认值不一定相同，根据具体情况来看==

```shell
# 连接本机的6379端口的redis 指定并发连接数为100（100个客户端） 每个客户端携带100000个请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

## 三、基本操作

redis默认有16个数据库（在redis.conf下的databases记录）。

```shell
# redis默认使用第0号数据库 选取3号数据库
select 3
# 设置键值 键：name 值hoshino
set name hoshino
# 通过键获取值
get name
# 查看当前数据库所有键
keys
# 清除当前数据库内容
flushdb
# 清除全部数据库的内容
flushall

```

## 四、基本数据类型

### 1.  Redis-Key

​		这不是一个数据类型，是redis的对所有数据类型通用的键，上面已经介绍过了，这里介绍一些其他知识：

```shell
# 1. 判断一个键在当前数据库中是否存在 存在返回1 不存在返回0
EXISTS [key_name]
# 2. 将一个键值对从当前数据库移动到另一个数据库
move [key_name] [database_id]
# 3. 删除一个键值对
del [key_name1] [key_name2 ...]
# 4. 设置键值对保留时间（单位s） 查看键值对的剩余时间使用“ttl [key_name]”查看 ttl（过期队列）
EXPIRE [key_name] [time]
# 5. 查看当前key的类型
type [key_name]
```

### 2.  String

#### （1）前置知识（append、strlen、incr、decr、incrby、decrby）

```shell
# 前面的set创建的就是string类型
# 给string类型追加字符串，append的对象如果不存在，则创建一个
append [key_name] [str]
# 查看字符串长度
strlen [key_name]
#################################
# 创建浏览量
set views 0
# 增加浏览量
incr views #此时为1
# 减少浏览量
decr views #此时为0
# 增加指定值
incrby views 10 #此时为10
# 减少指定值
decrby views 5 #此时为5
```

#### （2）**截取替换字符串**（getrange、setrange）

```shell
# 设置字符串
set my_key abcdef
# 截取字符串 substring 截取0到3位（包括3）
getrange my_key 0 3		#abcd
# 替换字符串 从第一位开始替换
setrange my_key 1 xx	#axxdef
```

#### （3）**设置过期时间、判断设置**（setex、setnx）

```shell
# 设置过期时间(单位：s)
setex [key_name] [time] [str]
# 判断不存在键时再设置
setnx [key_name] [str]
#####################################
# 例：（ex:expire(过期)，nx:not exist(不存在)）
setex my_key1 30 "hello"  #该key30s后过期
setnx my_key2 "hello"	  #当库中没有这个key时才设置
```

#### （4）**同时设置与获取多个值**（mset、mget）

```shell
# 同时设置多个值
mset [k1 v1] [k2 v2] [k3 v3] ...
# 同时获取多个值
mget [k1] [k2] [k3] ...
# 判断设置多个值
msetnx [k1 v1] [k2 v2] ...
#####################################
# 例：
mset k1 v1 k2 v2 k3 v3		#设置三个键值对
mget k1 k2 k3				#返回v1 v2 v3
msetnx k1 v1 k4 v4			#设置失败 msetnx是原子性的 k1已经存在 即使k4不存在也无法设置
```

#### （5）**操作技巧**：

```shell
# 有时候我们需要保存一个对象的属性 可以用string类型设为json格式：
setnx user {name:hoshino,age:20}
# 或者
msetnx user:1:name hoshino user:1:age 20
#########################################################
# getset 先获取再存储
getset [key_name] [value]
#例：
getset name zhangsan 	#返回为null 此时name被设为zhangsan
getset name hoshino		#返回zhangsan 此时name被设为hoshino
```

### 3.  List

​	**list的底层是一个双向链表。在两边插入改动值效率最高，在中间操作效率较低。可以用list做队列、栈等**

#### （1）列表类型的插入与弹出（push、pop）

```shell
# 列表可以左插 可以右插 也可以左弹右弹
# 左插与左弹
Lpush [list_name] [item]
Lpop [list_name]
# 右插
Rpush [list_name] [item]
Rpop [list_name]
##########################################
# 例：
# 左插三个元素
Lpush list one
Lpush list two
Lpush list three
# 此时list里元素顺序从0开始：three two one
#右插一个元素
Rpush list right
# 此时list里元素顺序从0开始：three two one right
# 左弹 右弹
Lpop list
Rpop list
# 此时list里元素顺序从0开始：two one
```

#### （2）获取列表值（lindex、lrange）

```shell
# 根据下标获取列表值
lindex [list_name] [index]
# 根据范围获取多个值
lrange [list_name] [start_index] [end_index]
# 上述list中，元素有two one，则获取0号元素：
lindex list 0
# 获取0到1号元素（包括1）
lrange list 0 1
# 获取列表所有值
lrange list 0 -1
```

#### （3）移除列表指定值（lrem）

```shell
# 根据指定值移出列表，其中num表示移出几个指定值，从0开始找
lrem [list_name] [num] [item]
# 如果一个列表名为list从0开始有值：one two two three four three two three
# 移除两个three
lrem list 2 three
# 此时列表从0开始的值：one two two four three two three
```

#### （4）裁剪列表（ltrim）

```shell
# 如果一个列表名为list从0开始有值：one two two three four
# 裁剪语法：
ltrim [list_name] [start_index] [end_index]
# 裁剪list的1到3号元素（包括3）
ltrim list 1 3
#此时list从0开始有值：two two three
```

#### （5）rpoplpush

```shell
# rpoplpush命令是移除一个列表的元素 并将它加入到另一个列表中
# 如果一个列表名为list从0开始有值：one two three，而列表名为otherlist的列表什么都没有
rpoplpush list otherlist
# 此时list有：one two；otherlist有：three
```

#### （6）替换列表值（lset）

```shell
# lset
lset [list_name] [index] [item]
# 如果一个列表名为list从0开始有值：one two three
# 把0号位置的one替换为other
lset list 0 other
# 如果一个位置不存在则会报错，如替换4号位，或者替换一个不存在的列表的位置
```

#### （7）根据固定值插入（Linsert）

```shell
# linsert，表示要在list的“base_item”的前方或后方插入“insert_item”
linsert [list_name] [before|after] [base_item] [insert_item]

```

### 4.  Set

#### （1）存取值（sadd、smembers）、判断是否存在（sismember）

```shell
# 向set中存值
sadd [set_name] [item]
# 查看set中的元素
smembers [set_name]

# 判断set中是否存在指定元素
sismember [set_name] [item]
```

#### （2）scard、srem

```shell
# 查看set内元素的个数
scard [set_name]
# 移除set内的指定元素
srem [set_name] [item_name]

```

#### （3）srandmember、spop

```shell
# 随机选取set内的元素，其中num是随机取几个
srandmember [set_name] [num]
# 随机删除一个元素
spop [set_name]
```

#### （4）smove

```shell
# 将一个set的元素移动到另一个set中
smove [src_set] [dsc_set] [item]
```

#### （5）sdiff、sinter、sunion

```shell
# 查看两个set的差集
sdiff [set1] [set2]
# 查看两个set的交集
sinter [set1] [set2]
# 查看两个set的并集
sunion [set1] [set2]
```

==应用场景：用户的共同好友、共同关注==

### 5.  Hash

#### （1）hset、hget、hgetall、hlen

```shell
# 给hash表设k-v键值对
hset [hash_name] [field] [value]
# 获取hash表指定字段的值
hget [hash_name] [field]
# 获取哈希表中的所有k-v
hgetall [hash_name]
# 获取hash的字段数量
hlen [hash_name]
```

#### （2）hkeys、hvals

```shell
# 获取hash表的所有的键
hkeys [hash_name]
# 获取hash表的所有的值
hvals [hash_name]
```

#### （3）hexists、hsetnx、hmget、hmset、hincrby

注意：==hmset在redis6.0后已经过时，因为hset也能设置多个k-v了==

```shell
# 判断hash中指定字段是否存在
hexists [hash_name] [field]
# hash中不存在指定字段再设置，否则不设置
hsetnx [hash_name] [field] [value]
# 获取hash中的多个值
hmget [hash_name] [field1] [field2] ...
# 设置hash中的多个k-v
hmset [hash_name] [field1 value1] [field2 value2] ...
# 给hash的指定字段的值加上增量，其中num为增量值
hincrby [hash_name] [field] [num]
```

### 6.  Zset

有序集合

#### （1）zadd、ZRangeByScore、ZRevRangeByScore、ZRange、ZRevRange

```shell
# 向zset中加入值 在要加入的值前加一个score（排序的依据）
zadd [zset_name] [score1] [item1] [score2] [item2]  ...
# 如，添加成员的工资水平
zadd salary 500 hoshino
zadd salary 2500 hoshino1
zadd salary 1500 hoshino2
#######################################################
# 选取zset中的所有元素
zrange [zset_name] 0 -1 
# 逆向选取zset中所有元素
zrevrange [zset_name] 0 -1
#######################################################
# zrangebyscore 升序排序，将上述集合按score升序排序
zrangebyscore salary -inf +inf
# 带上score
zrangebyscore salary -inf +inf withscores
# 只选取工资2000以下的
zrangebyscore salary -inf 2000 withscores
# 降序排序
zrevrangebyscore salary +inf -inf
```

#### （2）ZRem

```shell
# 移除zset中的元素
zrem [zset_name] [item]
# 如一个zset中有：hoshino、hoshino1两个元素，要移除hoshino，则：
zrem z_set hoshino
```

#### （3）ZCard、ZCount

```shell
# 获取zset中的成员数量
zcard [zset_name]
# 获取指定score区间的成员数量
zcount [zset_name] [start_score] [end_score]
#################################
# 例，我想统计工资在500到2000范围的人数
zcount salary 500 2000
```

## 五、特殊数据类型

### 1.   geospatial（地理位置）

#### （1）geoadd

```shell
# 向geo中添加元素（指定经纬度）
geoadd [geo_name] [经度] [纬度] [item_name]
# 
```

#### （2）geopos、geohash

```shell
# 返回一个位置的经纬度
geopos [geo_name] [item_name]
# geo里的元素是二维结构的（经度与纬度），如果我们想要获得他的一维结构，可以获取它的hash值
geohash [geo_name] [item_name]
```

#### （3）geodist、georadius、georadiusbymember

```shell
# 计算两个位置之间的距离 units为单位 可以有m、km、ft(英尺)、mi(英里)
geodist [geo_name] [item1] [item2] [units]

# georadius以给定的经纬度为中心，指定一个半径，返回在这个圆内的geo里的地理位置
# 基本使用 withdist使返回结果带上离中心的距离 withcoord使返回结果带上自身经纬度 count [num]，返回num个结果
georadius [geo_name] [中心经度] [中心纬度] [radius] [units] [withdist] [withcoord] [count [num]]
# 以上的georadius是指定经纬度来获得结果，而georadiusbymember则是使用一个geo里的成员作为中心
georadiusbymember [geo_name] [geo_member] [radius] [units] [withdist] [withcoord] [count [num]]
######################################################
# 如果有一个geo名为china:city，有成员116.23 31.25 beijing；118.31 26.30 shanghai；114.52 21.32 guangzhou，例：
# 查看beijing与shanghai的直线距离
geodist china:city beijing shanghai km
# 查看china:city里以112.31 24.35为中心的，半径500km的圆内的2个城市，带上经纬度
georadius china:city 112.31 24.35 500 km withcoord count 2
# 查看以china:city里beijing为中心的，半径500km的圆内的2个城市，带上直线距离
georadiusbymember china:city beijing 500 km withdist count 2
```

#### （4）底层实现

geospatial是基于zSet（有序集合）实现的，所以可以用zset的命令操作geo类型。

```shell
# 获取geo里的所有元素
zrange [geo_name] 0 -1
# 移除geo里的指定元素
zrem [geo_name] [item_name]
```

### 2.   hyperloglog

#### （1）基本介绍

​		基数：一个集合中的元素种类的个数，如一个集合有{a,b,c,d}则基数为4，{a,b,c,d,a}基数也为4（a是重复的）

​		hyperloglog是一个基数统计算法，空间复杂度低，误差小，在大数据背景下非常有用。

​		问题提出：传统的页面访问统计，最容易想到的解决方案是用set集合收集数据做归一化统计用户基数，然而这种方法在大数据背景下效率低，空间复杂度高，而hyperloglog只需要12kb（固定）的占用就能统计出大概，官方给出的误差是0.81%，还能接受。

​		提出者的名字缩写为PF。

#### （2）pfadd、pfcount

```shell
# 向一个基数集中添加元素
pfadd [pf_name] [item1] [item2] ...
# 统计一个基数集的元素数量（相同的不会统计）
pfcount [pf_name]
################################
# 例，一个页面访问者有a、b、c、d、e、f、g，其中a访问两次：
pfadd my_set a b c d e f g
pfadd my_set a
# 统计my_set中的基数
pfcount my_set
```

#### （3）pfmerge

```shell
# 将两个基数集合为另一个基数集，如果有一个基数集为set1{a,b,c}，第二个基数集为set2{c,d,f}，将两个集合合为一个set3
pfmerge set3 set1 set2
# 统计set3基数
pfcount set3  	#此时基数为5
```

注意：==如果统计不允许容错，则不要使用hyperloglog==。

### 3.  bitmaps

​		实际生活中，我们会有许多场景只有0与1两个状态。比如，一个人今天上班有没有打卡，一个账号今天活不活跃......

bitmaps就是一个0与1的数据结构。

#### （1）setbit、getbit、bitcount

```shell
# 给一个bitmaps设置一个值
setbit [bit_name] [index] [0|1]
# 获取bitmaps对应index的状态
getbit [bit_name] [index]
# 统计一个bitmaps一定范围内1的个数
bitcount [bit_name] [start_index] [end_index]
#########################################
# 例，一个人名为tom，他周一、周三、周六打卡了，其余没打卡，可以：
# 打卡
setbit tom 0 1
setbit tom 2 1
setbit tom 5 1
# 没打卡（可以不用写，默认所有位都是0）
setbit tom 1 0
setbit tom 3 0
setbit tom 4 0
setbit tom 6 0
# 统计他的打卡天数（默认不写start_index与end_index是统计全部）
bitcount tom
```

## 六、Redis事务

### 1.   概述

事务ACID原则：

1. 原子性
2. 一致性：事务前后数据的完整性必须保持一致
3. 隔离性：多个用户并发访问数据库时，多个并发事务之间要相互隔离
4. 持久性

Redis事务的本质：==一组命令的集合==，用一个队列存放起来，依次执行：

​		`队列 set set get 执行 `

​		==而Redis的单条命令是保证原子性的，但是事务不保证原子性！==

​		==Redis事务没有隔离级别的概念==

Redis事务执行流程：

		1. 开启事务
		1. 命令入队
		1. 执行事务

### 2.  基本使用

#### （1）开启、执行事务

```shell
# 开启事务
multi
# 设置命令入队
set k1 v1
set k2 v2
get k2
set k3 v3
# 执行事务
exec
```

#### （2）取消事务

```shell
# 开启事务
multi
# 设置命令入队
set k1 v1
get k1
# 此时，我不想要这个事务了，取消事务：
discard
```

### 3.   异常

#### （1）编译型异常

```shell
# 在事务的编译期就错了（语法错误）
# 如:
# 开启事务
multi
# 设置命令入队
set k1 v1
set k2 v2
getset k1	#语法错误 没有设值 打完后会报error，但它也会入队
# 执行事务
exec		#直接报错（编译错误），所有命令都无法执行
```

#### （2）运行时异常

```shell
# 语法没有错误，但是逻辑有错误
# 如：
# 先设置一个k1 值为一个字符串
set k1 "v1"
# 开启事务
multi
# 设置命令入队
set k2 v2
incr k1		#逻辑有错误，值为字符型，无法自增，但语法没有错误
get k1
# 执行事务
exec
#此时：
1) OK
2) error
3) "v1"
# 逻辑错误的语句无法执行，但其他命令可以执行（也恰恰说明了redis事务不保证原子性）
```

### 4.   watch、unwatch（乐观锁）

​	悲观锁每次执行操作都需要进行加锁解锁，效率低下。

​	乐观锁会在要修改对象时检查这个对象有没有被修改过，它允许其他线程对对象进行修改，效率高，经常使用。

```shell
# 在redis中，可以用watch实现乐观锁
####################################
# 基本设置与加锁操作
# 设置钱与支出
set money 100
set out 0
# 对money进行监视（加乐观锁）
watch money
##########################
# 开启事务
multi
# 对金额与支出进行修改
decrby money 20
incrby out 20
# 此时，如果另一个线程对money或out进行了修改，比如有线程2：set money 1000
# 那么接下来如果执行事务会返回nil（执行不成功）
exec	#执行失败
#########################################
# 此时，上面的乐观锁已经失效了，我们可以重新解锁再加锁
unwatch money
watch money
# 总结：如果事务执行失败，就先解锁再加锁
```

## 七、SpringBoot操作Redis

### 1.   Jedis

​		在springboot1.x，底层默认对redis的操作使用jedis实现，jedis在高并发环境下会产生线程不安全问题（机制类似BIO），想要解决这个问题可以用JedisPool（Jedis的线程池），目前，Jedis已经不推荐使用。

### 2.   Netty

​		在springboot2.x，底层对默认对redis的操作改为netty（机制类似NIO），多个进程间可以共享一个实例，线程安全。

​		对redis的操作都封装在RedisTemplete中（模板方法模式）。用户可以很方便地通过java代码操作redis。

## 八、Redis配置

### 1.   基本配置

#### （1）基本

```shell
# redis后台运行
daemonize no		#默认为no
# 包含，一个配置文件可以包含多个其他的配置文件，相当于import
include /xxx/xxx.conf
include /xxx/xxx/xxx.conf
# loglevel，redis总共支持四个日志级别：debug、verbose、notice、warning
loglevel notice		#默认为notice
# logfile，日志文件名称
logfile ""
# databases，数据库数量
databases 16
```

#### （2）网络相关

```shell
# 保护模式，非本地连接不能访问（在远程连接时要设为no）
protected-mode yes 		#默认为yes，在没有bind ip且没有密码的情况下，redis只接受本机响应
# bind，绑定主机	
bind 127.0.0.1 -::1		#默认只接受本机的访问请求，不写就无限制接受任何ip请求，服务器需要远程访问，需要将其注释掉
# port，端口号，默认6379
port 6379
# timeout，一个空闲的客户端维持多少秒会关闭，0表示关闭该功能（即永不关闭）
timeout 0
# tcp-keepalive，对客户端的心跳检测，每n秒检测一次，如果为0，则不会进行keepalive检测（建议设60）
tcp-keepalive 300
```

#### （3）安全

```shell
# 设置密码（也可以使用命令设置，但命令中设置的密码是临时的，重启redis后就没了）
requirepass xxx
命令：config set requirepass "xxx"
# 查看密码
config get requirepass
# 设置密码后，所有的命令均无法使用了，需要进行验证
auth "xxx"		#这样，就能使用命令了
```

#### （4）限制

```shell
# 设置redis同时可以与多少个客户端进行连接
maxclients 10000	#默认为10000，如果达到限制，redis会拒绝新的连接请求
# 设置redis可使用的内存空间（建议必须设置，否则内存占满将导致服务器宕机）
maxmemory <bytes>
# redis使用的内存空间达到限制时，会使用算法移除内存中的数据
maxmemory-policy <algorithm>
algorithm指定移除算法：
	- volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）
	- allkeys-lru：在所有集合key中，使用LRU算法移除key
	- volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
	- allkeys-random：在所有集合key中，移除随机的key
	- volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
	- noeviction：不进行移除。针对写操作，只是返回错误信息
# 移除样本数量，LRU与TTL算法都并非是精确的算法，而是估算值，可以设置样本的数量，从其中选择这么多数量的key去移除其中最LRU的那个：
maxmemory-samples 5
```

#### （5）持久化

```shell
# redis持久化有两种方式 AOF与RDB默认是RDB模式：
appendonly no		#开启AOF模式可以设为yes
# 持久化生成的文件
appendfilename "appendonly.aof"	#rdb模式生成 .rdb文件

#持久化文件同步策略
# appendfsync always		#每次修改都会同步。消耗性能
appendfsync everysec	#每秒执行一次同步，可能会丢失这1s的数据
# appendfsync no			#不执行同步，这个时候操作系统自己同步数据，速度最快
########################################
# 保存持久化文件的策略
save 900 1			#表示如果900秒内至少1个key发生变化，则重写rdb文件
save 300 10			#表示如果每300秒内至少10个key发生变化，则重写rdb文件
save 60 10000		#表示如果每60秒内至少10000个key发生变化，则重写rdb文件
# 保存持久化文件出错时，是否停止继续写入，比如保存rdb文件时，磁盘满了，redis保存出错，这时如果该项为yes就会拒绝客户端继续set值
stop-writes-on-bgsave-error yes		#默认yes，停止写入
# 对持久化文件是否进行压缩
rdbcompression yes 			#默认yes，采用LZF算法进行压缩
# 保存持久化文件的文件名
dbfilename dump.rdb			#默认为dump.rdb
# 每次保存持久化文件是否进行校验
rdbchecksum yes				#默认为yes
```

​		说明：==我们也可以在redis-cli使用save命令执行一个同步保存操作，将当前redis实例的所有数据快照以rdb文件的形式保存到硬盘，但这样会阻塞所有客户端（生产环境不用），生产环境下可以使用bgsave异步执行文件保存操作==。

​		==bgsave就是主进程fork出一个专门用来保存持久化文件的子进程，然后由这个子进程专门负责将数据保存到磁盘。==

#### （6）主从复制

```shell
# 指定自身的主机，默认所有redis都是主机，所以默认不配置，要当从机需要指定
replicaof [master_ip] [master_port]
#例，指定6379端口的redis当自己的主机
replicaof 127.0.0.1 6379
```



### 2.   说明

​		针对docker构建的redis镜像，如果希望使用自己的配置文件，需要对配置文件进行修改：

```shell
# 1. 首先，需要将bind注释掉：
#bind 127.0.0.0 -::1

# 2. 在服务器上，把protected-mode改为no
protected-mode no

# 3. Dockerfile基于redis构建自己的redis
FROM redis:latest

# 4. Dockerfile中，将配置文件考入容器中（也可以卷挂载）
COPY 主机相对于Dockerfile的相对路径 容器绝对路径

# 5. 执行命令
CMD [ "redis-server", "容器内的配置文件路径" ]

# 6. 构建镜像
docker build -f Dockerfile -t myredis:latest

# 7. 运行容器
docker run -d --name myredis -p 6379:6379 myredis:latest

# 8. 此时，查看运行中的容器docker ps，你会发现没有容器在运行，回想docker容器的运行机制：docker的执行进程发现容器前台没有进程运行时，会将容器停止。再想想配置文件中的daemonize是不是yes，是yes则redis为后台运行，前台自然没有进程运行，所以docker会将容器停止，将daemonize改为no即可
```

## 九、Redis持久化

### 1.  RDB持久化

#### （1） 基本操作

​		上面已经已经介绍过redis.conf里的redis持久化配置。

​		那么，什么时候会触发持久化保存文件的机制呢？

##### ~1. 持久化触发机制

> 1. save的规则满足的情况下
> 2. 执行flushall命令
> 3. 退出redis，也会自动触发机制，产生rdb文件
>
> 而在docker环境下：重启容器，也会触发机制，产生rdb文件

##### ~2. 恢复rdb文件的数据

​		恢复数据十分简单，只需要将rdb文件放在redis的启动目录下，redis启动时会自动检查dump.rdb文件，恢复其中的数据。

​		可以通过如下命令查看redis的启动目录：

```shell
# 正常主机下
config get dir
1)"dir"
2)"/usr/local/bin"   # rdb该存放在这
# docker下
config get dir
1)"dir"
2)"/data"			 # rdb该存放在这
```

#### （2）RDB持久化的优缺点

* 优点：
  * 适合大规模的数据恢复，因为会fork一个子进程去做持久化操作，效率很高。
  * 对数据的完整性要求不高。
* 缺点：
  * 需要一定的时间间隔进行操作，可能就是因为这一小段时间间隔造成最后一次修改的数据丢失。
  * fork进程时，会占用一定的内存空间。

### 2.  AOF持久化

#### （1）基本操作

​		aof与rdb相似，也是使用一个子进程去做持久化操作，不同于rdb的是：aof保存的不是数据，而是存数据的操作，比如每次写数据的操作（如set、hset等）都会存入aof文件中。redis启动时也会自动检测aof文件进行数据设置（类似mysql的备份）。

​		redis默认使用rdb持久化，如果需要aof，上面配置文件也介绍过了（加深印象，再介绍一遍吧），需要开启：

```shell
applendonly yes
# aof文件名，默认为appendonly.aof
appendfilename "appendonly.aof"
# aof文件同步策略
appendfsync everysec # 默认每秒同步一次
```

​		提醒：==aof只会记入写数据的操作，毕竟读数据不涉及数据的恢复（废话）==

#### （2）AOF持久化的优缺点

* 优点：
  * 每一次修改都同步，文件的完整性会更好
* 缺点：
  * 相对于数据文件来说，aof文件远大于rdb文件，面对大数据情况下，因为aof存的是对数据的操作，它的恢复数据的操作是比rdb慢得多的。
  * aof运行效率也比rdb慢。

### 3.  建议

​		持久化应该如何配？根据实际情况看：

1. 如果你只用redis做缓存，不需要任何持久化操作。
2. 在redis集群环境下，因为rdb文件只用作后备用途，建议只在slave（从机）上持久化rdb文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则，提升性能。
3. 如果enable aof，好处是在最恶劣的情况下也只会丢失不超过2秒的数据，启动脚本较简单，只load自己的aof文件就可以了，代价是带来了持续的IO操作，并且AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘允许，应该尽量减少aof rewrite的频率，aof重写的默认大小64m太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
4. 如果不enable aof，仅靠Master-Slave Repllcation实现高可用性也可以，能省掉一大笔IO，也减少了rewriter时带来的系统波动。代价是如果Master/Slave（主/从机）同时宕机，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的rdb文件，载入较新的那个。

## 十、Redis发布订阅

### 1.   发布订阅模式

​		当许多客户端订阅了一个频道，只需要将消息推送到这个频道，频道会自动遍历所有订阅人，将消息推送给他们：

![pubsub2](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/pubsub2.png)

### 2.  基本命令

#### （1）SUBSCRIBE、PUBLISH、UNSUBSCRIBE

```shell
# 订阅频道，频道不存在会自动注册
subscribe [channel_name1] [channel_name2] ...
# 发布消息
publish [channel_name] [message]
# 退订指定频道
unsubscribe [channel_name]
################################
# 消息格式：
# 频道publish一条消息后，订阅方收到：
1) "message"		#收到一条消息
2) "channel_name"	#从哪个频道收到消息
3) "message"		#收到什么消息
```

#### （2）PSUBSCRIBE、PUNSUBSCRIBE

```shell
# 第一个P代表Pattern，即给定一个pattern，订阅（退订）符合这个pattern的频道
psubscribe [pattern1] [pattern2] ...
punsubscribe [pattern1] [pattern2] ...
```

## 十一、Redis主从复制

### 1.  基本介绍

​		主从复制就是一个主机配一个或多个从机，主机将数据复制到从机上，==主机主要复制数据写入，从机主要复制数据读取，这样就做到了读写分离。==

​		主从复制在大数据环境下性能优越，一般来说，单机redis在内存容量达到20G时就会达到性能瓶颈了，所以我们需要搭建redis集群。其次，在生产环境下，如果主机宕机，从机可以顶替主机的位置继续进行工作。

​		主从复制优点十分多，一眼能想到的：负载均衡、读写分离、高可用性。

​		==在默认情况下，所以的redis都是主机==

![在这里插入图片描述](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/2021041820541725.png)

### 2.  基本搭建

#### （1）查看当前redis的主从复制信息

```shell
info replication
# 单机状态下输出如下信息：
127.0.0.1:6379> info replication
# Replication
role:master					#当前redis的职责：主机
connected_slaves:0			#连接的从机数量：0
master_failover_state:no-failover	
master_replid:1e1dc5e2678487bf148a186e378c86e347bfa1e8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

#### （2）启动多个redis

> 1. 先将redis配置文件复制多份，比如redis6379.conf、redis6380.conf、redis6381.conf   . . .
> 2. 对各个配置文件的配置进行一定修改：
>    * 端口
>    * pid文件名
>    * log文件名
>    * dump.rdb名
> 3. redis-server redis6379.conf、redis-server redis6380.conf  . . .
>
> 一般说来，搭建redis集群最好写个shell脚本，不然太麻烦了。



#### （3）配置主从关系

​		因为默认情况下，redis都是主机，所以配置主从关系只需要在想要指定的从机处，用从机去指定一个它的主机就好了。

```shell
# 例如，本机上有三个redis，6379、6380、6381
# 我希望用6379当主机，6380与6381当从机：
# 6380指定6379为自己的主机
redis-cli_of_6380 > slaveof 127.0.0.1 6379
# 6381指定6379为自己的主机
redis-cli_of_6381 > slaveof 127.0.0.1 6379
```

注意：==使用命令配置主从关系是暂时的，永久配置需要在配置文件中指定==，redis配置部分已经介绍过。

### 3.  测试

​		配置好主从关系后，现在进行读写测试：

```shell
# 在主机设置值，主机可写可读
6379 >  set k1 v1
6379 >  get k1	#v1
# 从机取值成功
6380 >  get k1	#v1
6381 >  get k1  #v1
# 从机设值
6380 >  set k2 v2	#error，从机只能读，无法写
# 说明主机会自动将数据复制到从机
```

​		宕机测试：

```shell
# 将主机关闭
6379 >  shutdown
# 在没有设置哨兵的情况下，主机宕机后，从机也无法顶替主机的位置，只能一直当从机，但是取值还是可用的
6380 >  get k1	#v1
6380 >  info replication	# slave
#########################
# 主机恢复后，主从可以继续同步数据
# 如果是使用命令行设置的主从关系，从机重启后，会变为主机，只要重新设置主从关系，数据可以自动同步
6380 restart
6380 >  get k1	#null
6380 >  slaveof 127.0.0.1 6379
6380 >  get k1	#v1
```

### 4.  复制原理

主从复制大概原理如下：

1. slave启动成功后连接到master后会发送一个sync同步命令。

2. master接到命令，启动后台存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕后，master将传送整个数据文件到slave，完成一次完全同步。

这里涉及两个官方给出的概念：

* 全量复制：master把整个数据文件给slave，slave接收到数据文件后将其存盘并加载到内存
* 增量复制：master把后续设值，修改数据的命令依次传给slave，slave完成数据同步。

所以，slave只要是重新连接master，一次全量复制就会被自动执行。因此，即使从机断连，再次连接主机也能够同步数据。

### 5.  链路模型

​		前面的一主多从结构从某种角度看也是略微不健康的：

1. 主机宕机后，从机没法获取新数据。
2. 要想从从机中选取一个当主机，需要改变所有从机的从属关系。

​		在此提出新模型：链路模型。

![image-20220810191502496](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220810191502496.png)

​		像上图，层层从属，也是能通过主机同步所有从机的数据的。这样当主机宕机，只需要将主机之后的从机身份改为主机即可，无需改动其他从机的配置。

​		注意：==即使按照模型来看，第二个从机的主机是第一个从机，但第一个从机的身份仍是slave，不代表第一个从机有master的身份，所以其无法写入数据，在master宕机后，仍然需要手动将第一个从机的身份改为master==

### 6.  哨兵模式

#### （1）基本介绍

​		以上的模型，在主机无法工作后，都需要手动修改模型，费时费力，在面对大型集群时显得力不从心。

在redis 2.6.x之后，提供了哨兵（sentinel）模式。

​		哨兵模式：将一个子进程独立出来作为哨兵，监控各个redis服务器，当master宕机后，从slave中自动选出一个做master顶替。

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/11320039-57a77ca2757d0924.png)

​		在这里，哨兵做的事：

* 通过定时发送命令，探测redis服务器让服务器返回响应，监控其运行状态，包括主从。
* 当哨兵检测到master宕机，会自动将slave切换成master，通过**发布订阅模式**，通知在自己检测范围内的其他的从服务器，修改配置文件，让其他从机切换自身的主机。

#### （2）多哨兵模式

​		哨兵也有可能会出问题，为此，我们可以让多个哨兵进行监控，各个哨兵之间也互相监控，这样就形成了多哨兵模式。

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/11320039-3f40b17c0412116c.png)

#### （3）故障切换过程（failover）

​		假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，就称为**客观下线**，主机客观下线后，哨兵之间就会进行一次投票，投票的结果由一个哨兵取得，进行failover操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机。这样对于客户端而言，一切都是透明的。

​		// TODO: 关于投票算法，这里不做介绍，也许以后会补上

#### （4）如何选取新主机？

哨兵选定新主库用四个字概括：**“筛选+打分”**。简单来说，就是根据筛选条件选出候选从库，然后通过打分，选出最高分的作为新主库。下面说一下筛选条件和打分规则。

**筛选条件**

首先从库必须在线运行。

其次从库网络状态良好。从库和主库断连超出一定的阈值就把这个从库筛掉。这个阈值就是`down-after-milliseconds`，表示主从库断连的最大连接超时时间。例如发生超过10次，就认定从库的网络状况不好。

关于`down-after-milliseconds`，值越小，哨兵就越敏感。当网络拥塞但主库正常，可能会发生不必要的切换。而当主库真的故障了，就切换及时，对业务影响最小。

因此`down-after-milliseconds`要设置合适的值，既减少不必要的切换，也保证能够及时切换，降低对业务的影响。

**打分规则**

第一轮，优先级最高的从库得分高。可通过`slave-priority`配置项，设置从库优先级。

第二轮，和旧主库同步程度最接近的从库得分高。很容易理解，越接近主库，说明数据丢失越少。前面介绍主从复制时，已经知道主从库是通过`repl_backlog_buffer`保持同步的，所以`slave_repl_offset`最接近`master_repl_offset`，得分高。

第三轮，ID号小的从库得分高。每个实例都会有一个ID。这是最后一轮，必须决出胜负，所以就选择最小ID的作为新主库。

#### （5）哨兵模式基本配置

编写sentinel.conf文件：

```shell
#当前Sentinel服务运行的端口，默认26379，有哨兵集群再另配
port 26379  
# 哨兵监听的主服务器 后面的2是表示有2个哨兵主观认为master宕机就是客观宕机
sentinel monitor mymaster 127.0.0.1 6379 2
# 3s内mymaster无响应，则主观认为mymaster宕机了
sentinel down-after-milliseconds mymaster 3000
#如果10秒后,mysater仍没启动过来，则启动failover  
sentinel failover-timeout mymaster 10000  
# 执行故障转移时， 最多有1个从服务器同时对新的主服务器进行同步
sentinel parallel-syncs mymaster 1
# 如果主机有密码，则需要配置
sentinel auth-pass mymaster 123456
```

这里是一个哨兵，如有需求，可以编写多个哨兵的配置sentinel1.conf、sentinel2.conf . . .

**启动哨兵：**

```shell
# 通过redis-sentinel启动，在/usr/local/bin下
redis-sentinel sentinel.conf
```

**启动完毕后，会将其他元数据信息追加写入到配置文件中：**

```shell
sentinel known-replica mymaster 192.168.0.60 6380 #代表redis主节点的从节点信息
sentinel known-replica mymaster 192.168.0.60 6381 #代表redis主节点的从节点信息
##########################
#如果配置了哨兵集群，会发现其他哨兵
#代表感知到的其它哨兵节点
sentinel known-sentinel mymaster 192.168.0.60 26380 52d0a5d70c1f90475b4fc03b6ce7c3c56935760f 
 #代表感知到的其它哨兵节点
sentinel known-sentinel mymaster 192.168.0.60 26381 e9f530d3882f8043f76ebb8e1686438ba8bd5ca6 
```

## 十二、Redis缓存穿透、击穿与雪崩

### 1.  缓存穿透

​		当查询redis中没有的数据时，该查询会下沉到数据库层，同时数据库层也没有该数据，当这种情况大量出现或被恶意攻击时，接口的访问全部透过Redis访问数据库，而数据库中也没有这些数据，我们称这种现象为"缓存穿透"。缓存穿透会穿透Redis的保护，提升底层数据库的负载压力，同时这类穿透查询没有数据返回也造成了网络和计算资源的浪费。

​		解决方案：

* 在接口访问层对用户做校验，如接口传参、登陆状态、n秒内访问接口的次数；
* 利用布隆过滤器，将数据库层有的数据key存储在位数组中，以判断访问的key在底层数据库中是否存在；如果redis内不存在该数据，则通过布隆过滤器判断数据是否在数据库内，如果不在，直接返回null即可，避免了查询底层数据库的动作。
* 如果底层数据库查询不到，就往redis里面设一个空值，这样再次请求这个数据时，就会从redis得到一个空值，避免查询底层数据库。

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/n41yh8r1hu.png)

### 2.  缓存击穿

​		缓存击穿和缓存穿透从名词上可能很难区分开来，它们的区别是：穿透表示底层数据库没有数据且缓存内也没有数据，击穿表示底层数据库有数据而缓存内没有数据。

​		比如，一个成千上万访问量的热点数据，可能其设置了60s的过期时间，在60.1秒的时候大量并发访问了这个热点，而其在redis内已经过期了，查询操作就会下沉到底层数据库，数据库负载压力骤增，这种现象称为”缓存击穿“。

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/97521eq6uv.png)

解决方案：

* 延长key过期时间或设置永不过期。
* 利用互斥锁（分布式锁）保证同一时刻只有一个线程（客户端）可以查询底层数据库，一旦查到数据就缓存到redis内，避免其他大量请求同时穿过redis访问底层数据库。

### 3.  缓存雪崩

​		缓存雪崩是缓存击穿的"大面积"版，缓存击穿是数据库缓存到Redis内的热点数据失效导致大量并发查询穿过redis直接击打到底层数据库，而缓存雪崩是指Redis中大量的key几乎同时过期，然后大量并发查询穿过redis击打到底层数据库上，此时数据库层的负载压力会骤增，我们称这种现象为"缓存雪崩"。事实上缓存雪崩相比于缓存击穿更容易发生，对于大多数公司来讲，同时超大并发量访问同一个过时key的场景的确太少见了，而大量key同时过期，大量用户访问这些key的几率相比缓存击穿来说明显更大。

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/gcy1ji8j6p.png)

解决方案：

* 在可接受的时间范围内随机设置key的过期时间，分散key的过期时间，以防止大量的key在同一时刻过期；
* 对于一定要在固定时间让key失效的场景(例如每日12点准时更新所有最新排名)，可以在固定的失效时间到来时在接口服务端设置随机延时，将请求的时间打散，让一部分查询先将数据缓存起来；
* 延长热点key的过期时间或者设置永不过期，这一点和缓存击穿中的方案一样；
