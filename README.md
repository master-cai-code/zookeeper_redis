# zookeeper_redis
#### 1.关于集群和分布式的理解：
  
      集  群：一群物理机器组成一个整体对外提供服务
      分布式：一个大系统拆分成多个子系统，子系统分落到不同机器上可以独立运行

## 2.Redis的介绍：
    
      1.非关系型数据库NoSQL
      2.数据都是在内存中----------------速度快
      3.key-value结构，支持的value类型有String，Hash，List，Set，Sorted Set----------数据类型丰富
      4.操作是原子性的-------------支持事务
      5.可将内存中的数据持久化到磁盘中（也就是文件中）
      
 ### 3.Redis的特性：
 
     1.Redis是单线程的：因为多线程会有上下文的切换，带来额外的开销，这个开销远大于比读数据的时间
     2.不过持久化的时候会fork出一个子进程
     
### 4. 各种键值的操作命令
   #### 4.1 String :    set  key  value                存map    计数功能
             set myName "cai"            get myName 
             
             
   #### 4.2 Hash :      hset HashName  key  value      存对象    存用户信息等
             hset stu name "cai"         hget  stu name 
             hmset  ...(一次性多个属性赋值)
             
             
   #### 4.3 List ：     lpush key value                双向链表   消息队列，分页功能
             lrange key start end   获取从下标start到end的值
             
             
   #### 4.4 Set ：      sadd key value                 集合    （集合差集，并集，交集）    全局去重 
             smembers key   获取集合中所有的成员
  
  
   #### 4.5 Sorted Set ：   zadd key score value       有序集合    排行榜，top N
  
  
### 5.Redis的应用

   #### 5.1 通过Redis实现分布式锁：   （一步设置key和value还有过期时间（预防死锁））
            setnx key value   +   expire key times  =  set key value ex time nx 
            
            
   #### 5.2 作为消息队列         （把 List 当消息管道）         
  
### 6.Redis的持久化  （从内存写入磁盘，再从磁盘写入内存）

   #### 6.1 RDB（Redis DataBase） ：把内存的全部数据定时dump到磁盘上 （全量，间隔一段时间）
      缺点：1. 不能实时持久化（会丢失最后一次快照后的所有修改）
           2. bgsave会fork出一个子进程，频繁执行，成本过高
           3. 全量同步，数据量大会由于IO影响性能
    
  #### 6.2 AOF （Append Only File）：记录除了查询以外，所有变更数据库的指令，以追加的方式保存到AOF文件中
        AOF文件通常比RDB文件更大
        
  #### 6.3 混合使用 ：先用bgsave以RDB形式将内存中的全部数据写入磁盘，之后再有新数据，再使用AOF追加到文件
  
### 7. Redis 的过期策略和淘汰机制 （当Redis内存存满了，咋办）

 #### 7.1 定期删除和惰性删除
      定期删除：每个100ms，随机抽查key是否过期，过期则删除。（由于是随机的抽查，会导致有的key到期了，但没删除）
      惰性删除：获取这个key的时候再检查是否过期
      定期+惰性仍然有问题： 定期删除没抽中，也没及时获取这个key ，惰性删除也失效，也会导致内存越来越大，此时需要内存淘汰机制
      内存淘汰：移除最近最少使用的key

 ### 8.缓存穿透和缓存击穿
     8.1 缓存穿透： 故意大量（高并发）请求  缓存中某一个不存在的数据，导致请求都到数据库上   （一个不存在的key） ....
     
     8.2 缓存雪崩： 大量的key同一时间集体失效，导致请求到数据库上                          （很多key失效） 过期时间加上一个随机值
     
     8.3 缓存击穿： 在某一个key正好刚过期的时候，来了大量高并发的请求                       （一个key失效）   .....
    
### 9. Redis高可用：主从复制、哨兵机制

   #### 9.1 主从复制：   主写从读、从先全量再增量同步
       
       一个master主数据库，多个slave从数据库.主数据库负责写操作，并把改变的数据同步到从数据库；从数据库主要负责读操作
       slave从数据库先全量同步再增量同步
  
  #### 9.2 哨兵机制：  监控Master和Slave数据库的状态（是否正常运行，Master挂掉后，选举新的Master）   sentinel
    和zookeeper类似
    
    
    
    
    
    
## zookeeper的介绍：
