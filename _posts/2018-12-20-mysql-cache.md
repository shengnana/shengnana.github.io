## MYSQL缓存 ##

### 缓存配置 ###

##### sql语句执行过程包括查询缓存，解析SQL，优化SQL，执行。MYSQL配置中有一个缓存开关，开关默认是关闭状态，也即禁止使用query_cache。 
##### 要查询状态，可以用下面的查询语句。 #####

  {% raw %}
    MYSQL> SHOW VARIABLES LIKE 'have_query_cache';
  {% endraw %}

##### mysql查询缓存会跟踪查询相关的每个表，如果表发生变化(写、更新、修改表结构等)，则该表相关的所有缓存失效。 

##### 查询mysql中缓存的设置情况，使用下面的命令： #####

  {% raw %}
    MYSQL> SHOW STATUS LIKE 'Qcache%';
  {% endraw %}

##### 如果查询结果中显示 query_cache_size =0，则表示没有设置，应该到my.ini中设置。 #####

  {% raw %}
    query_cache_size=128M 
    query_cache_type=1 
  {% endraw %}

##### query_cache_size表示mysql用于缓存的内存块大小。mysql会将这个内存块设置为一个个变长的数据块。mysql启动是会申请一个连续的内存块。之后需要缓存的时候，在查询开始时会根据参数申请一个大于参数query_cache_min_res_unit的块来存储数据，因为返回结果前无法获知结果多大，所以即使数据很小，也会分配同样大小的内存块。 

##### 申请内存块时需要锁住内存块，所以这个操作很慢，mysql会尽量避免这个操作，选择尽量小的内存块，如果不够则继续申请。如果存储完毕有多余则会释放。 

### 缓存命中 

##### 缓存存放在一个引用表中，通过hash值引用。不会被命中的常见的两种情况：
* hash值是根据sql查询，数据库，客户端协议版本等进行hash，任何改动都会导致hash变化而不会命中缓存，例如去掉空格
* sql语句中任何不定值的参数都会导致不写入缓存，所以也就不会命中，例如now()，current_date(),自定义函数，自定义变量等。虽然这种语句不会缓存，但是在查询开始之前仍然会先查询缓存，只有解析SQL之后才会知道语句中是否含有不定值。

##### 下面是不缓存的相关情况 #####

| 执行表达式 | 说明 |
|:---------|:---------|
| BENCHMARK()   | 返回服务器执行表达式的时间，而不会涉及分析和优化的开销,用于测试特定操作的执行速度   |
| CONNECTION_ID()   | 返回已经建立连接的连接ID   |
|----
| CURRENT_DATE(),CURDATE()   | 当前日期，例如2018-12-20   |
| CURRENT_TIME(),CURTIME()   | 当前时间，例如14:23:23   |
| CURRENT_TIMESTAMP() | 当前日期和时间，例如2018-12-20 15:41:27 |
|----
| DATABASE() | 当前使用数据库名称 |
| ENCRYPT(param1) |带一个参数的ENCRYPT()|
| FOUND_ROWS() | select语句中增加SQL_CALC_FOUND_ROW选项，然后执行select FOUND_ROWS(),可以返回匹配全量的行数,适用于sql中有limit分页的清况 |
|----
| GET_LOCK(),RELEASE_LOCK() | |
| LAST_INSERT_ID() |  |
| LOAD_FILE() |  |
| MASTER_POS_WAIT() |  |
| NOW() |  |
| RAND() |  |
| SYSDATE() |  |
| UNIX_TIMESTAMP() | 不带参数的UNIX_TIMESTAMP() |
| USER() |  |
|----
| 引用自定义变量或者mysql系统数据库中的表|  |
| SELECT * FROM ...WHERE autoincrement_col IS NULL | 与NULL做比对 
{: rules="groups"}


### 打开QCache带来的额外的消耗 ###

* 查询操作之前会先检查是否命中缓存
* 查询完毕之后，如果没有命中缓存，则需要写入缓存
* 当写入数据的时候，如果缓存开启，则需要将该表的所有缓存置为无效。如果缓存数据较大，消耗较大，因为该操作靠全局锁来保证完整性。

### 查看缓存使用状态 ###

  {% raw %}
    mysql> show status like ‘%Qcache%’; 

    Variable_name           | Value     | 
   
      +————————-----------------+———–+ 
 
    | Qcache_free_blocks      | 1         | 
    | Qcache_free_memory      | 134208800 | 
    | Qcache_hits             | 0         | 
    | Qcache_inserts          | 0         | 
    | Qcache_lowmem_prunes    | 0         | 
    | Qcache_not_cached       | 2         | 
    | Qcache_queries_in_cache | 0         | 
    | Qcache_total_blocks     | 1         | 

  {% endraw %}
  
  #### 解析 #####
    {% raw %}
      * Qcache_free_blocks:表示查询缓存中目前还有多少剩余的blocks，如果该值显示较大，则说明查询缓存中的内存碎片过多了，可能在一定的时间进行整理。  
    减少碎片： 
      合适的query_cache_min_res_unit可以减少碎片，这个参数最合适的大小和应用程序查询结果的平均大小直接相关，可以通过内存实际消耗（query_cache_size - Qcache_free_memory）除以Qcache_queries_in_cache计算平均缓存大小。 
      可以通过Qcache_free_blocks来观察碎片，这个值反应了剩余的空闲块，如果这个值很多，但是 
      * Qcache_lowmem_prunes却不断增加，则说明碎片太多了。可以使用flush query cache整理碎片，重新排序，但不会清空，清空命令是reset query cache。整理碎片期间，查询缓存无法被访问，可能导致服务器僵死一段时间，所以查询缓存不宜太大。 
      * Qcache_free_memory:查询缓存的内存大小，通过这个参数可以很清晰的知道当前系统的查询内存是否够用，是多了，还是不够用，DBA可以根据实际情况做出调整。 
      * Qcache_hits:表示有多少次命中缓存。我们主要可以通过该值来验证我们的查询缓存的效果。数字越大，缓存效果越理想。 
      * Qcache_inserts: 表示多少次未命中然后插入，意思是新来的SQL请求在缓存中未找到，不得不执行查询处理，执行查询处理后把结果insert到查询缓存中。这样的情况的次 数，次数越多，表示查询缓存应用到的比较少，效果也就不理想。当然系统刚启动后，查询缓存是空的， 这很正常。 
      * Qcache_lowmem_prunes:该参数记录有多少条查询因为内存不足而被移除出查询缓存。通过这个值，用户可以适当的调整缓存大小。 
      * Qcache_not_cached: 表示因为query_cache_type的设置而没有被缓存的查询数量。 
      * Qcache_queries_in_cache:当前缓存中缓存的查询数量。 
      * Qcache_total_blocks:当前缓存的block数量。   
   {% endraw %}



