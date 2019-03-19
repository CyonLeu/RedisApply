#Redis缓存应用基础

##1、Redis简介
  Redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多：String(字符串)、List(链表)、Set(集合)、ZSet(有序集合)和Hash（哈希类型）。与Memcached一样，为了保证效率，数据都是缓存在内存中。较大区别是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。
  
 
   
   Redis 是一个高性能的key-value数据库。目前相对较优秀的NoSQL数据库。单线程支持10w+QPS。
   
   
##2、Redis安装 （Mac OSX）
   
   1) 官网 <https://redis.io> 下载最新版本，目前是Redis 5.0.3；
   
   2）本地文件解压到目录：/usr/local/redis-5.0.3
   
   3）编译源程序，进入redis目录
   
   `make `
   
   4）启动redis服务，进入Redis/bin目录

  ` ./redis-server ../redis.conf `
  
  ```
➜  bin ./redis-server ../redis.conf
30578:C 18 Mar 2019 17:43:18.825 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
30578:C 18 Mar 2019 17:43:18.825 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=30578, just started
30578:C 18 Mar 2019 17:43:18.825 # Configuration loaded
30578:M 18 Mar 2019 17:43:18.826 * Increased maximum number of open files to 10032 (it was originally set to 256).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 30578
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

30578:M 18 Mar 2019 17:43:18.827 # Server initialized
30578:M 18 Mar 2019 17:43:18.828 * DB loaded from disk: 0.000 seconds
30578:M 18 Mar 2019 17:43:18.828 * Ready to accept connections
  
  ```
   
   5）通过redis-cli连接使用
   
   ` ./redis-cli`
   
   ```
-rwxrwxrwx   1 root     staff   247K  2 13 16:48 redis-cli
-rwxrwxrwx   1 root     staff   1.3M  2 13 16:48 redis-server
➜  bin ./redis-cli
127.0.0.1:6379> keys *
   
   ```
   
   
   6）Redis的配置文件
  	 
  	 ```
  	 daemonize：如需要在后台运行，把该项的值改为yes
  	 pdifile：把pid文件放在/var/run/redis.pid，可以配置到其他地址
  	 bind：指定redis只接收来自该IP的请求，如果不设置，那么将处理所有请求，在生产环节中最好设置该项
  	 port：监听端口，默认为6379
  	 timeout：设置客户端连接时的超时时间，单位为秒
  	 loglevel：等级分为4级，debug，revbose，notice和warning。生产环境下一般开启notice
  	 logfile：配置log文件地址，默认使用标准输出，即打印在命令行终端的端口上
  	 database：设置数据库的个数，默认使用的数据库是0
  	 save：设置redis进行数据库镜像的频率
  	 rdbcompression：在进行镜像备份时，是否进行压缩
  	 dbfilename：镜像备份文件的文件名
  	 dir：数据库镜像备份的文件放置的路径
  	 slaveof：设置该数据库为其他数据库的从数据库
  	 masterauth：当主数据库连接需要密码验证时，在这里设定
  	 requirepass：设置客户端连接后进行任何其他指定前需要使用的密码
  	 maxclients：限制同时连接的客户端数量
  	 maxmemory：设置redis能够使用的最大内存
  	 appendonly：开启appendonly模式后，redis会把每一次所接收到的写操作都追加到appendonly.aof文件中，当redis重新启动时，会从该文件恢复出之前的状态
  	 appendfsync：设置appendonly.aof文件进行同步的频率
  	 vm_enabled：是否开启虚拟内存支持
  	 vm_swap_file：设置虚拟内存的交换文件的路径
  	 vm_max_momery：设置开启虚拟内存后，redis将使用的最大物理内存的大小，默认为0
  	 vm_page_size：设置虚拟内存页的大小
  	 vm_pages：设置交换文件的总的page数量
  	 vm_max_thrrads：设置vm IO同时使用的线程数量
  	 
  	 ```
　　
#3、基本命令应用
1）String 字符串

```
127.0.0.1:6379> SET name Brace
OK
127.0.0.1:6379> get name
"Brace"
127.0.0.1:6379> setnx name Good
(integer) 0
127.0.0.1:6379> 

```
重点说明 SETNX

只在键 key 不存在的情况下， 将键 key 的值设置为 value 。
若键 key 已经存在， 则 SETNX 命令不做任何动作。
SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。
返回值
命令在设置成功时返回 1 ， 设置失败时返回 0 。

SETEX：常用的SETEX ，当我们设置key的值时，同时会设置其过期时间。
将键 key 的值设置为 value ， 并将键 key 的生存时间设置为 seconds 秒钟。
如果键 key 已经存在， 那么 SETEX 命令将覆盖已有的值。
SETEX 命令的效果和以下两个命令的效果类似：

SET key value
EXPIRE key seconds  # 设置生存时间
SETEX 和这两个命令的不同之处在于 SETEX 是一个原子（atomic）操作， 它可以在同一时间内完成设置值和设置过期时间这两个操作， 因此 SETEX 命令在储存缓存的时候非常实用。
返回值
命令在设置成功时返回 OK 。 当 seconds 参数不合法时， 命令将返回一个错误

2）HASH 哈希表

	```
	127.0.0.1:6379> hset hName nickname "zhangsan"
	(integer) 1
	127.0.0.1:6379> hget hName nickname
	"zhangsan"
	127.0.0.1:6379> hexists hName nickname
	(integer) 1
	127.0.0.1:6379> hexists hName uid
	(integer) 0
	127.0.0.1:6379> hlen hName
	(integer) 1
	127.0.0.1:6379> hincrby hName uid 3
	(integer) 3
	127.0.0.1:6379> hget hName uid
	"3"
	127.0.0.1:6379> hset hName age 20
	(integer) 1
	127.0.0.1:6379> hmset hName nickname "lisi" age 12
	OK
	127.0.0.1:6379> hmget hName nickname uid age
	1) "lisi"
	2) "3"
	3) "12"
	127.0.0.1:6379> hkeys hName
	1) "nickname"
	2) "uid"
	3) "age"
	127.0.0.1:6379> hvals hName
	1) "lisi"
	2) "3"
	3) "12"
	127.0.0.1:6379> hgetall hName
	1) "nickname"
	2) "lisi"
	3) "uid"
	4) "3"
	5) "age"
	6) "12"
	127.0.0.1:6379> 
	
	```
	
	
3) LIST 列表

BRPOP 是列表的阻塞式(blocking)弹出原语。
它是 RPOP key 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 BRPOP 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。
关于阻塞操作的更多信息，请查看 BLPOP key [key …] timeout 命令， BRPOP 除了弹出元素的位置和 BLPOP key [key …] timeout 不同之外，其他表现一致。

返回值
假如在指定时间内没有任何元素被弹出，则返回一个 nil 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 key ，第二个元素是被弹出元素的值。

```

127.0.0.1:6379> LPUSH list 1 2 3 4 5
(integer) 5
127.0.0.1:6379> LPUSH list 6
(integer) 6
127.0.0.1:6379> lrange list 0 6
1) "6"
2) "5"
3) "4"
4) "3"
5) "2"
6) "1"
127.0.0.1:6379> lpop list
"6"
127.0.0.1:6379> rpop list
"1"
127.0.0.1:6379> rpoplpush list list2
"2"
127.0.0.1:6379> lrange list2  0 5
1) "2"
127.0.0.1:6379> lrange list 0 5
1) "5"
2) "4"
3) "3"
127.0.0.1:6379> lrem list 2 4
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "5"
2) "3"
127.0.0.1:6379> lpush list 5
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "5"
2) "5"
3) "3"
127.0.0.1:6379> lrem list 2 5
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "3"
127.0.0.1:6379> brpop list 30
1) "list"
2) "3"
127.0.0.1:6379> brpop list 30 list2 20
1) "list2"
2) "2"
127.0.0.1:6379> 


```

4) SET 集合 (ZSET)

```
127.0.0.1:6379> sadd set "goods"
(integer) 1
127.0.0.1:6379> sadd set "this" "is"
(integer) 2
127.0.0.1:6379> smembers set
1) "is"
2) "this"
3) "goods"
127.0.0.1:6379> spop set is
(error) ERR value is not an integer or out of range
127.0.0.1:6379> spop set 1
1) "goods"
127.0.0.1:6379> smembers set
1) "is"
2) "this"
127.0.0.1:6379> SRANDMEMBER set 2
1) "is"
2) "this"
127.0.0.1:6379> SRANDMEMBER set 1
1) "is"
127.0.0.1:6379> SRANDMEMBER set 1
1) "is"
127.0.0.1:6379> SRANDMEMBER set 1
1) "this"
127.0.0.1:6379> srem set this
(integer) 1
127.0.0.1:6379> smembers set
1) "is"

```



