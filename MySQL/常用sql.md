查询当月天数

```sql
select date 
FROM(
SELECT DATE_FORMAT(DATE_SUB(last_day('2020-04-01'), INTERVAL xc-1 day), '%Y-%m-%d') as date
FROM ( 
			SELECT @xi:=@xi+1 as xc from 
			(SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6) xc1, 
			(SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6) xc2,  
			(SELECT @xi:=0) xc0 
) xcxc) x0 where x0.date >= (select date_add('2020-04-01',interval-day('2020-04-01')+1 day))
```

查询上周同期数据

```sql
 and DATEDIFF( date_format(now() , '%Y-%m-%d' ) , date_format( order_time, '%Y-%m-%d' ) ) >= 7
                and DATEDIFF( date_format(now() , '%Y-%m-%d' ) , date_format( order_time, '%Y-%m-%d' ) ) &lt;= WEEKDAY(now()) + 7
```



查询上月同期数据

```sql
  and DATE_FORMAT(order_time,'%Y-%m-%d') &lt;= IFNULL(DATE_FORMAT(concat(extract(year_month from date_add(now(),interval -1 MONTH)),SUBSTRING(CURDATE(),9)),'%Y-%m-%d'),DATE_FORMAT(concat(extract(year_month from date_add(now(),interval -1 MONTH)),SUBSTRING(CURDATE(),9) - 1),'%Y-%m-%d'))
                and DATE_FORMAT(order_time,'%Y-%m-%d') >= DATE_FORMAT(concat(extract(year_month from date_add(now(),interval -1 MONTH)),'01'),'%Y-%m-%d');
```

