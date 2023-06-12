





version >= 21.7.3.14



```sql
explain
select
	database,
	table,
	count(1) cnt
from
	system.parts
where
	database in ('datasets', 'system')
group by
	database,
	table
order by
	database,
	cnt desc
limit 2 by database;
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202305232230498.png)













![](https://raw.githubusercontent.com/imattdu/img/main/img/202305232233366.png)







```sql
EXPLAIN AST SELECT number from system.numbers limit 10;
```









a join b 

b会加载内存









左右表均是分布式表









OPTIMIZE TABLE test_a FINAL;

根据orderBy字段去重
