---
layout: post
title: redis源码分析
date: 2016-09-01
categories: soft
tags: redis ds
---





  
  
  
  
  
  



    



### redis各文件作用 

redis源码总共6w多行。

| 文件| 作用|
|-|-|
| `adlist.c` 、 `adlist.h`| 双端链表数据结构的实现。                                        
| `ae.c` 、 `ae.h` 、 `ae_epoll.c` 、 `ae_evport.c` 、`ae_kqueue.c`、`ae_select.c`| 事件处理器，以及各个具体实现。          
| `anet.c` 、 `anet.h`| Redis 的异步网络框架，内容主要为对 socket 库的包装。            
| `aof.c`| AOF 功能的实现。                                              
| `asciilogo.h`| 保存了 Redis 的 ASCII LOGO 。                                 
| `bio.c`、 `bio.h`| Redis 的后台 I/O 程序，用于将 I/O 操作放到子线程里面执行， 减少 I/O 操作对主线程的阻塞。
| `bitops.c`| 二进制位操作命令的实现文件。                                      
| `blocked.c`| 用于实现 BLPOP 命令和 WAIT 命令的阻塞效果。                   
| `cluster.c`、 `cluster.h`| Redis 的集群实现。                                               
| `config.c` 、 `config.h`| Redis 的配置管理实现，负责读取并分析配置文件， 然后根据这些配置修改 Redis 服务器的各个选项。|
| `crc16.c` 、 `crc64.c` 、 `crc64.h`| 计算 CRC 校验和。                                                 
| `db.c`| 数据库实现。                                                
| `debug.c`| 调试实现。                                                  
| `dict.c` 、 `dict.h`| 字典数据结构的实现。                                        
| `endianconv.c` 、 `endianconv.h`| 二进制的大端、小端转换函数。                                
| `fmacros.h`| 一些移植性方面的宏。                                        
| `geo.h`、`geo.c`|位置相关（经纬度）
| `latency.h`、`latency.c`|延迟类
| `quicklist.h`、`quicklist.c`|list对象的底层实现，双端ziplist的链表|
| `help.h`、`utils/generate-command-help.rb` 程序自动生成的命令帮助信息。   
| `hyperloglog.c`| HyperLogLog 数据结构的实现。                                
| `intset.c` 、 `intset.h`| 整数集合数据结构的实现，用于优化 SET 类型。                 
| `lzf_c.c` 、 `lzf_d.c` 、 `lzf.h` 、 `lzfP.h`| Redis 对字符串和 RDB 文件进行压缩时使用的 LZF 压缩算法的实现
| `Makefile` 、 `Makefile.dep`| 构建文件。                                                  
| `memtest.c`| 内存测试。                                                  
| `mkreleasehdr.sh`| 用于生成释出信息的脚本。                                    
| `multi.c`| Redis 的事务实现。                                          
| `networking.c`| Redis 的客户端网络操作库，用于实现命令请求接收、发送命令回复等工作，文件中的函数大多为 write 、 read 、 close 等函数的包装，以及各种协议的分析和构建函数.
| `notify.c`| Redis 的数据库通知实现。
| `object.c`| Redis 的对象系统实现.
| `pqsort.c` 、 `pqsort.h`| 快速排序（QuickSort）算法的实现。
| `pubsub.c`| 发布与订阅功能的实现。                                
| `rand.c` 、 `rand.h`| 伪随机数生成器。                                       
| `rdb.c` 、 `rdb.h`| RDB 持久化功能的实现。                                 
| `redisassert.h`| Redis 自建的断言系统。
| `redis-benchmark.c`| Redis 的性能测试程序。                                    
| `server.c`| 负责服务器的启动、维护和关闭等事项。                      
| `redis-check-aof.c` 、 `redis-check-dump.c`| RDB 文件和 AOF 文件的合法性检查程序。                 
| `redis-cli.c`| Redis 客户端的实现。                                  
| `server.h`| Redis 的主要头文件，记录了 Redis 中的大部分数据结构，包括服务器状态和客户端状态。
| `redis-trib.rb`| Redis 集群的管理程序
| `release.c` 、 `release.h`| 记录和生成 Redis 的释出版本信息。                           
| `replication.c`| 复制功能的实现。                                            
| `rio.c` 、 `rio.h`| Redis 对文件 I/O 函数的包装, 在普通 I/O 函数的基础上增加了显式缓存、以及计算校验和等功能。|
| `scripting.c`| 脚本功能的实现。                          
| `sds.c` 、 `sds.h`| SDS 数据结构的实现，SDS 为 Redis 的默认字符串表示。
| `sentinel.c`| Redis Sentinel 的实现。
| `setproctitle.c`| 进程环境设置函数。                                    
| `sha1.c` 、 `sha1.h`| SHA1 校验和计算函数。                                 
| `slowlog.c` 、 `slowlog.h`| 慢查询功能的实现。                                    
| `solarisfixes.h`| 针对 Solaris 系统的补丁。                             
| `sort.c`| SORT 命令的实现。                                     
| `syncio.c`| 同步 I/O 操作。                                       
| `testhelp.h`| 测试辅助宏。                                          
| `t_hash.c` 、 `t_list.c` 、 `t_set.c` 、 `t_string.c` 、`t_zset.c`| 定义了 Redis 的各种数据类型，以及这些数据类型的命令。
| `util.c` 、 `util.h`| 各种辅助函数。                                           
| `valgrind.sup`| valgrind 的suppression文件。                                
| `version.h`| 记录了 Redis 的版本号。                                     
| `ziplist.c` 、 `ziplist.h`| ZIPLIST 数据结构的实现，用于优化 LIST 类型。                
| `zipmap.c` 、 `zipmap.h`| ZIPMAP 数据结构的实现，在 Redis 2.6 以前用与优化 HASH 类型, Redis 2.6 开始已经废弃。
| `zmalloc.c` 、 `zmalloc.h`| 内存管理程序


### 内存结构 

首先需要明确的是redis是一个key-value存储。

1.  key就是字符串，而value的可以是:string、list、map、set、sort set、(bitmaps、hyperloglogs、geospatial indexes)。
2.  各种value对应的是字符串对象、list object(列表对象)、hash object(哈希对象)、set object(集合对象)、sorted set object(有序集合对象)。
3.  每一种对象又是由这些数据结构组成的:sds(c string的变种)、list、dict、skipList、intSet、 zipList。
4.  一种对象可以底层对应多种数据结构实现，eg列表对象底层可能是list或者ziplist，可以进行转换。

其结构图如下：

![redis结构](/images/soft/redis.jpg)

### 对象 

    typedef struct redisObject {

        // 类型          
        unsigned type:4;

        // 编码
        //
        unsigned encoding:4;

        // 对象最后一次被访问的时间
        unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

        // 引用计数
        int refcount;

        // 指向实际值的指针
        void *ptr;

    } robj;

type

见server.h Object types

|对象|对象type属性的值|type命令的输出|
|-|-|
|字符串对象|OBJ_STRING|"string"|
|列表对象|OBJ_LIST|"list"|
|哈希对象|OBJ_HASH|"hash"|
|集合对象|OBJ_SET|"set"|
|有序集合对象|OBJ_ZSET|"zset"|

encoding

见server.h Objects encoding

|编码常量|编码所对应的底层数据结构|
|-|-|
|OBJ_ENCODING_INT|long类型的整数|
|OBJ_ENCODING_EMBSTR|embstr编码：的简单动态字符串|
|OBJ_ENCODING_RAW|简单动态字符串|
|OBJ_ENCODING_HT|字典|
|OBJ_ENCODING_LINKEDLIST|双端链表|
|OBJ_ENCODING_ZIPLIST|压缩列表|
|OBJ_ENCODING_INTSET|整数集合|
|OBJ_ENCODING_SKIPLIST|跳跃表和字典|
|OBJ_ENCODIGN_QUICKLIST|linked的ziplist|

type和encoding可以相互结合.具体结合方式可参照开头的[大图](#memory)。

eg:

1.  redisObject的type属性为REDIS_STRING、encoding为REDIS_ENCODING_INT表明使用整数值实现字符串对象

2.  redisObject的type属性为REDIS_HASH、encoding为REDIS_ENCODING_ZIPLIST表明使用压缩列表实现的哈希对象;encoding为REDIS_ENCODING_HT表明使用双向链表实现的哈希。

可以使用type和object encoding来查看redis对象的类型type和底层的数据结构类型encoding

    127.0.0.1:6379> set msg "xx"
    OK
    127.0.0.1:6379> type msg
    string
    127.0.0.1:6379> object encoding msg
    "embstr"

### 数据结构 

redis底层的数据结构有这么几种:sds、list、dict、skipList、intSet、 zipList。

1.  对象的底层实现，比如列表对象listObject其底层可能就是使用的list数据结构。

2.  服务器的一些其他功能的底层实现也是这些数据结构，比如redisServer保持多个客户端的状态信息，监视器等功能底层实现也是list；redis本身就是个kv存储系统，redis数据库的底层就是使用了dict，然后dict对象底层也是用的dict，有点蒙了，可以对照上图。

#### sds 

sds: simple dynamic string

    struct sdshdr {

        // 记录 buf 数组中已使用字节的数量
        // 等于 SDS 所保存字符串的长度
        int len;

        // 记录 buf 数组中未使用字节的数量
        int free;

        // 字节数组，用于保存字符串
        char buf[];

    };


#### list 

    typedef struct listNode {

        // 前置节点
        struct listNode *prev;

        // 后置节点
        struct listNode *next;

        // 节点的值
        void *value;

    } listNode;


    typedef struct list {

        // 表头节点
        listNode *head;

        // 表尾节点
        listNode *tail;

        // 链表所包含的节点数量
        unsigned long len;

        // 节点值复制函数
        void *(*dup)(void *ptr);

        // 节点值释放函数
        void (*free)(void *ptr);

        // 节点值对比函数
        int (*match)(void *ptr, void *key);

    } list;

#### dict(hash) 

    typedef struct dict {

        // 类型特定函数
        dictType *type;

        // 私有数据
        void *privdata;

        // 哈希表
        dictht ht[2];

        // rehash 索引
        // 当 rehash 不在进行时，值为 -1
        int rehashidx; /* rehashing not in progress if rehashidx == -1 */

    } dict;

    typedef struct dictType {

        // 计算哈希值的函数
        unsigned int (*hashFunction)(const void *key);

        // 复制键的函数
        void *(*keyDup)(void *privdata, const void *key);

        // 复制值的函数
        void *(*valDup)(void *privdata, const void *obj);

        // 对比键的函数
        int (*keyCompare)(void *privdata, const void *key1, const void *key2);

        // 销毁键的函数
        void (*keyDestructor)(void *privdata, void *key);

        // 销毁值的函数
        void (*valDestructor)(void *privdata, void *obj);

    } dictType;

    typedef struct dictht {

        // 哈希表数组
        dictEntry **table;

        // 哈希表大小
        unsigned long size;

        // 哈希表大小掩码，用于计算索引值
        // 总是等于 size - 1
        unsigned long sizemask;

        // 该哈希表已有节点的数量
        unsigned long used;

    } dictht;

    typedef struct dictEntry {

        // 键
        void *key;

        // 值
        union {
            void *val;
            uint64_t u64;
            int64_t s64;
        } v;

        // 指向下个哈希表节点，形成链表
        struct dictEntry *next;

    } dictEntry;

扩容与收缩、链地址法。

渐进式rehash

#### skipList 

    typedef struct zskiplist {

        // 表头节点和表尾节点
        struct zskiplistNode *header, *tail;

        // 表中节点的数量
        unsigned long length;

        // 表中层数最大的节点的层数
        int level;

    } zskiplist;


    typedef struct zskiplistNode {

        // 后退指针
        struct zskiplistNode *backward;

        // 分值
        double score;

        // 成员对象
        robj *obj;

        // 层
        struct zskiplistLevel {

            // 前进指针
            struct zskiplistNode *forward;

            // 跨度
            unsigned int span;

        } level[];

    } zskiplistNode;

跳跃表的介绍。

#### intSet  

    typedef struct intset {

        // 编码方式
        uint32_t encoding;

        // 集合包含的元素数量
        uint32_t length;

        // 保存元素的数组
        int8_t contents[];

    } intset;

类型转换

#### zipList 

    typedef struct zlentry {

        // prevrawlen ：前置节点的长度
        // prevrawlensize ：编码 prevrawlen 所需的字节大小
        unsigned int prevrawlensize, prevrawlen;

        // len ：当前节点值的长度
        // lensize ：编码 len 所需的字节大小
        unsigned int lensize, len;

        // 当前节点 header 的大小
        // 等于 prevrawlensize + lensize
        unsigned int headersize;

        // 当前节点值所使用的编码类型
        unsigned char encoding;

        // 指向当前节点的指针
        unsigned char *p;

    } zlentry;

连锁更新

#### 内存结构-多态 

![底层数据结构的变换](/images/soft/redis_object_transform.jpg)


### 调试redis 

使用vi，ctags，cscope，gdb进行调试 参见[我是这样看源码的](/2016/02/28/view_source)

因为我们要调试，所以还是[使用源码进行安装](/2015/12/28/redis_basic#hello)

1. 启动服务端redis-server
2. 启动客户端redis-cli
3. gdb attach到服务进程上`gdb -p pid`，这种方式可以调试redis接收请求的过程，但是必须redis服务器启动起来。所以如果观察redis的启动流程正确的姿势应该是gdb可执行文件
4. 打断点，对不同数据结构进行断点调试
ps: mac上的gdb还需设置下: http://jingyan.baidu.com/article/925f8cb8fa362ec0dde0561a.html

执行过程如下：

    cd /Users/lichuangjian/soft/redis-3.0-annotated
    make install
    gdb ./src/redis-server
    b 3952
    r

![gdb_redis](/images/soft/gdb_redis.png)

### redis启动流程 


![redis启动流程](/images/soft/redis_start.jpg)

redis.c中有两个全局变量
`struct RedisServer * server`、 `struct RedisCommand *commandTable`

1. 初始化redisServer的配置initServerConfig()  

2. 如果指定了配置文件，加载配置文件，修改默认配置loadServerConfig()

3. 打开监听端口listenToPort()

4. 打开unix本地端口anetUnixServer()

5. 创建数据库并进行初始化（默认是16个）

6. 创建pubSub相关数据结构

7. 初始化server的其他一些配置

8. 为 serverCron() 创建时间事件aeCreateTimeEvent()

9. 为 TCP 连接关联连接应答（accept）处理器aeCreateFileEvent()

10. 为本地套接字关联应答处理器aeCreateFileEvent()

11. 如果AOF功能打开，打开或创建aof文件

12. 如果服务器以cluster方式运行，初始化cluster, clusterInit()

13. 初始化复制功能有关的脚本缓存replicationScriptCacheInit()

14. 初始化脚本系统scriptingInit()

15. 初始化慢查询功能slowlogInit()

16. 初始化BIO系统bioInit()

17. 从aof或者rdb文件中载入数据

18. 启动集群？

19. 运行事件处理器。

Q&A ?

1. 如何将其设置为守护进程的。

### redis接受请求 

![redis请求处理流程](/images/soft/redis_cmd_process.jpg)

`p (struct redisServer) server`

`p (struct redisClient) c`

redis事件接口

    // 文件事件处理器，eg: readQueryFromClient、acceptTcpHandler、
    // acceptUnixHandler、sendReplyToClient、acceptHandler ..
    typedef void aeFileProc(struct aeEventLoop *eventLoop, int fd, void *clientData, int mask);
    // 时间事件处理器，eg
    typedef int aeTimeProc(struct aeEventLoop *eventLoop, long long id, void *clientData);
    // 事件终结处理器，eg
    typedef void aeEventFinalizerProc(struct aeEventLoop *eventLoop, void *clientData);
    // 事件预处理器，eg
    typedef void aeBeforeSleepProc(struct aeEventLoop *eventLoop);

redis命令实现接口

    // eg：setCommand，getCommand，zrangeCommand..
    typedef void redisCommandProc(redisClient *c);

redis的事件包含有时间事件和文件事件。

以最简单的get xx为例

    127.0.0.1:6379> get msg
    "heheda"
    (3405.11s)

acceptTcpHandler

1. anetTcpAccept -> anetGenericAccept : 创建客户端连接
2. acceptCommonHandler -> createClient :
   1. 分配redisClient内存空间，
   2. 设置io非阻塞，
   3. 禁用 Nagle 算法，
   4. 设置tcpKeepAlive时间，
   5. 添加读事件并绑定readQueryFromClient事件处理器,
   6. 绑定client到db0,
   7. 初始化事务和发布订阅相关数据结构
   8. ...

readQueryFromClient

1. 从缓冲区读取客户端命令内容
2. processInputBuffer或者processMultibulkBuffer
   1. 解析命令sdssplitargs,并为解析后的每一个参数创建为一个stringObject（解析如下例子所示）
   2. 执行命令processCommand
      1. lookupCommand,根据命令名查找对应的redisCommand
      2. addReply

    *  sds *arr = sdssplitargs("timeout 10086\r\nport 123321\r\n");
    *  会得出
    *  arr[0] = "timeout"
    *  arr[1] = "10086"
    *  arr[2] = "port"
    *  arr[3] = "123321

#### reactor

todo

redis的文件事件处理采用的是reactor模式，通过select/poll/epoll/kqueue这些I/O多路复用函数库，解决了一个线程处理多个连接的问题。
mac上其实现为kqueue，linux为epool，sun为select，可参见config.h和ae.c中的声明。

   ae.c

    #else
        #ifdef HAVE_EPOLL
        #include "ae_epoll.c"
        #else
            #ifdef HAVE_KQUEUE
            #include "ae_kqueue.c"
            #else
            #include "ae_select.c"
            #endif
        #endif
    #endif


    config.h

    /* Test for polling API */
    #ifdef __linux__
    #define HAVE_EPOLL 1
    #endif

    #if (defined(__APPLE__) && defined(MAC_OS_X_VERSION_10_6))|| defined(__FreeBSD__)|| defined(__OpenBSD__)|| defined (__NetBSD__)
    #define HAVE_KQUEUE 1
    #endif

    #ifdef __sun
    #include <sys/feature_tests.h>
    #ifdef _DTRACE_VERSION
    #define HAVE_EVPORT 1
    #endif
    #endif

### redis多机  

### redis使用场景 

### 参考 

[redis设计与实现]<http://redisbook.com/>

[mit公开课算法导论:跳跃表]<http://open.163.com/movie/2010/12/7/S/M6UTT5U0I_M6V2TTJ7S.html>
