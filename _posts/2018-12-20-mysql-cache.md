## MYSQL缓存 ##
##### sql语句执行过程包括查询缓存，解析SQL，优化SQL，执行。MYSQL配置中有一个缓存开关，开关默认是关闭状态，也即禁止使用query_cache。 #####
##### 要查询状态，可以用下面的查询语句。 #####

  {% highlight html%}
    MYSQL> SHOW VARIABLES LIKE 'have_query_cache';
  {% endhighlight %}
