## MYSQL缓存 

### 缓存配置 

##### sql语句执行过程包括查询缓存，解析SQL，优化SQL，执行。MYSQL配置中有一个缓存开关，开关默认是关闭状态，也即禁止使用query_cache。 
##### 要查询状态，可以用下面的查询语句。 

  {% raw %}
    MYSQL> SHOW VARIABLES LIKE 'have_query_cache';
  {% endraw %}

##### mysql查询缓存会跟踪查询相关的每个表，如果表发生变化(写、更新、修改表结构等)，则该表相关的所有缓存失效。 

##### 查询mysql中缓存的设置情况，使用下面的命令： 

{% raw %}
mysql> SHOW STATUS LIKE 'Qcache%';
{% endraw %}

##### 如果查询结果中显示 query_cache_size =0，则表示没有设置，应该到my.ini中设置。 
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

##### 下面是不缓存的相关情况

| 函数等 | 说明 |
|:--------|:-------:|
| BENCHMARK()   | 返回服务器执行表达式的时间，而不会涉及分析和优化的开销,用于测试特定操作的执行速度   |
| CONNECTION_ID()   | 返回已经建立连接的连接ID   |
| CURRENT_DATE(),CURDATE()   | 当前日期，例如2018-12-20   |
| CURRENT_TIME(),CURTIME()   | 当前时间，例如14:23:23   |
| CURRENT_TIMESTAMP() | 当前日期和时间，例如2018-12-20 15:41:27 |
| DATABASE() | 当前使用数据库名称 |
| ENCRYPT(param1) |带一个参数的ENCRYPT()|
| FOUND_ROWS() | select语句中增加SQL_CALC_FOUND_ROW选项，然后执行select FOUND_ROWS(),可以返回匹配全量的行数,适用于sql中有limit分页的清况 |
| GET_LOCK(),RELEASE_LOCK() | |
| LAST_INSERT_ID() | |
| LOAD_FILE() | |
| MASTER_POS_WAIT() | |
| NOW() | |
| RAND() | |
| SYSDATE() | |
| UNIX_TIMESTAMP() | 不带参数的UNIX_TIMESTAMP() |
| USER() | |
| 引用自定义变量或者mysql系统数据库中的表|
| SELECT * FROM ...WHERE autoincrement_col IS NULL | |
|=====

### 打开QCache带来的额外的消耗

* 查询操作之前会先检查是否命中缓存
* 查询完毕之后，如果没有命中缓存，则需要写入缓存
* 当写入数据的时候，如果缓存开启，则需要将该表的所有缓存置为无效。如果缓存数据较大，消耗较大，因为该操作靠全局锁来保证完整性。

### 查看缓存使用状态

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














