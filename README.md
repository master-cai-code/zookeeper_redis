# zookeeper_redis          
     2020.3.18    17:20
#### 1.关于集群和分布式的理解：
  
      集  群：一群物理机器组成一个整体对外提供服务
      分布式：一个大系统拆分成多个子系统，子系统分落到不同机器上可以独立运行
      
      
      
      

## 2.Redis的介绍：
    
      1.非关系型数据库NoSQL
      2.数据都是在内存中----------------速度快
      3.key-value结构，支持的value类型有String，Hash，List，Set，Sorted Set----------数据类型丰富
      4.操作是原子性的-------------支持事务
      5.可将内存中的数据持久化到磁盘中（也就是文件中）
      
      6.单线程+IO多路复用（select+poll+epoll）待补充...   IO复用是同步IO，没有新开线程去执行
             select：一个个轮询，无并发能力还会阻塞卡死    遍历
             poll：链表实现轮询，无最大连接数的限制        遍历
   
             epoll：事件驱动，socket好了主动“举手”告知     回调
      
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
    和zookeeper类似  监控和选举..
    
    
    
    
    
    
    
    
    
    
    
    
## zookeeper的介绍：解决分布式应用当中的数据管理问题 == 文件系统 + 通知机制
### 1. 文件系统-------树形结构
   #### 1.1 四种类型的 znode 
            持久节点  ；持久序列编号  ；临时节点  ；临时序列编号
 
 ### 2.通知机制 ：客户端注册监听它关心的节点，当节点发生变化时，zookeeper会通知客户端 （也称 发布与订阅）
 
### zookeeper应用
 ### 3. 配置管理：把一个配置写入znode节点， 相关的应用程序进行监听，一旦节点发生变化，zookeeper会通知应用程序，应用程序获取新的配置信息
 ### 4. 命名服务：客户端能根据服务名字获取资源或服务的地址------------服务的注册与发现
 ### 5. 分布式锁：
          5.1 独占：创建临时节点，谁创建成功谁抢到锁，释放连接后，临时节点删除
          5.2 时序：创建临时有序节点，每个节点一个编号，按从小到大的顺序执行
          
  ### 6.zookeeper选举机制 ： 一个leader 多个follower 不够还有observer-----------半数以上，按照zxid再myid的大小，大的获胜
      leader负责写操作，数据同步到follower，投票的发起和决议
      follower 负责读，写操作转发给leader ，有投票资格
      observer 负责读，写操作转发给leader，但无资格投票
      集群刚启动和leader挂掉后
 ### 7.zookeeper保证的是CP：   C：Consistency    A：Availability   P：Partition tolerance
      C: 数据一致性：多台机器数据始终一样
      A: 可用性：读写请求在一定时间内得到响应
      P: 分区容忍性： 分布式要求节点间是互通的，由于网络故障，形成了独立的分区，彼此不能互通交换数据.当一个数据只存在于一个节点上时，
      不连通的分区将得不到该数据（这是不能容忍的），所以要把数据分布到不同节点保存，复制的节点越多，容忍性越高.但会带来一致性和可用性的问题....
     
 ###  8. 观察者模式：一个对象（目标对象）的改变会引起其他对象（观察者）的改变    
     1.目标类里用一个ArrayList存储观察者对象
     2.目标类还有增加、删除、通知观察者的方法等
     3.通知观察者就是遍历ArrayList集合，取出里面的观察对象，调用观察者的方法（让观察者发生改变）
