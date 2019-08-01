## SQL执行顺序

#### 一、手写

> **select distinct**
>
> ​	<select_list>
>
> **from**
>
> ​	<left_table> 
>
> ​	<join_type>   **join**  <right_table> **on** <join_condition>
>
> **where**
>
> ​	<where_condition>
>
> **group by**
>
> ​	<group_by_list>
>
> **having**
>
> ​	<having_condition>
>
> **order by**
>
> ​	<order_by_condition>
>
> **limit**   
>
> ​	<limit_number>

#### 二、机读

> **from**	<left_table> 
>
> **on** 		<join_condition>
>
> <join_type>   **join**  <right_table>
>
> **where**  	<where_condition>
>
> **group by**  <group_by_list>
>
> **having**    <having_condition>
>
> **select**
>
> **distinct**	<select_list>
>
> **order** **by**	<order_by_list>
>
> **limit**	<limit_number>

![1550494318883](C:\Users\csw\AppData\Roaming\Typora\typora-user-images\1550494318883.png)

## Join图 

![âjoin å¾âçå¾çæç´¢ç"æ](https://raw.githubusercontent.com/mzlogin/mzlogin.github.io/master/images/posts/database/Visual_SQL_JOINS_orig.jpg)

#### inner join

> select * from employee a inner join department b on a.deptId = b.id;

#### left join(右表不满足条件的用null补全)

> select * from employee a left join department b on a.deptId = b.id;
>
> select * from employee a left join department b on a.deptId = b.id where b.id is null;

#### right join(左表不满足条件的用null补全)

> select * from employee a right join department b on a.deptId = b.id;
>
> select * from employee a right join department b on a.deptId = b.id where a.id is null;

#### full outer join(MySQL不支持该语法)

> select * from employee a left join department b on a.deptId = b.id
>
> union
>
> select * from employee a right join department b on a.deptId = b.id;

> select * from employee a left join department b on a.deptId = b.id where b.id is null
>
> union
>
> select * from employee a right join department b on a.deptId = b.id where a.id is null;