### 数据库
- RDMS：MySQL
- NoSQL：MongoDB，Redis，HBase  
四种：KV存储，列族存储（HBase），文档存储（MongoDB），图式存储（Neo4j）
- NewSQL：FoundationDB
### 特性
- 单线程
- 丰富的数据结构
- 支持复制和集群
- 两种持久化方式
- 最多十六个数据库，0-15，select选择数据库，默认为0
### 数据类型
- string  
支持超时失效，支持不存在创建，支持存在才修改，整数支持自增自减  
- set  
SINTER交集，SISMEMBER是否在set中
- sorted set  
ZCARD返回集合元素个数，ZRANK表示按score从小到大0开始的索引值
- list
- hash  

手册
https://lvtao.net/content/book/redis.htm
### 认证
conf中requirepass，cli中auth
### 事务
先执行multi，然后可执行多个命令，仅放入队列，exec时同时执行
watch，乐观锁，期间如果数据被其他程序改变，事务不会执行  
### 发布订阅
支持模式订阅
### 持久化
不能取代备份
- RDB  
快照的方式，二进制格式，SAVE和BGSAVE可手动保存  
- AOF  
追加的方式，BGREWRITEAOF以命令的格式重写内存中数据到文件，写完后替换当前文件并追加其间的操作  
默认为RDB，启动时优先使用AOF，BGSAVE和BGREWRITEAOF不会同时运行
### 复制
支持链式复制，默认只读，非阻塞方式同步，主服务配置了requirepass，从需要masterauth密码认证  
`slaveof IP Port`
### 高可用（sentinel）
监控主Redis，可以监控多组Redis服务器，如果出故障了，选一个从的作为新的主Redis  
`redis-sentinel redis-sentinel.conf`
##### 配置
`sentinel monitor mastername ip port quorum`设置一组Redis及其主Redis地址及法定哨兵数量（用于判断下线，一般奇数）  
`sentinel down-after-milliseconds mastername milliseconds`多久断开算下线  
`sentinel parallels-syncs mastername numslaves`新主Redis同时连接的从服务数量  
`sentinel failover-timeout mastername milliseconds`多久没有恢复算超时  
##### 常用命令
`sentinel masters`获取所有主服务  
`sentinel slaves mastername`获取所有从服务  
`sentinel get-master-addr-by-name mastername`  
`sentinel reset`  
`sentinel failover mastername`手动触发主服务故障
### 集群
- Codis（豌豆荚）
- Redis Cluster（官方）
- Cerberus（芒果TV）