---
title: LeetCode-MySQL-Easy
date: 2022/10/19 21:57:36
updated: 2023/3/23 23:01:58
categories:
- Database
tags:
- SQL
---

# 175

[combine-two-tables](https://leetcode.cn/problems/combine-two-tables/)

两个表关联查询

```sql
select firstName,lastName,city,state 
from person as a left join (
    select * from address
) as b
on a.personId = b.personId
```

# 181

[employees-earning-more-than-their-managers](https://leetcode.cn/problems/employees-earning-more-than-their-managers/)

简单关联查询

```sql
select a.name as Employee from employee as a 
left join employee as b
on a.managerid = b.id
where a.salary > b.salary
```

# 182

[duplicate-emails](https://leetcode.cn/problems/duplicate-emails/)

查找同一列中重复元素，使用having对分组结果筛选

- 我的解法，未通过，无值返回null

```sql
select Email from (
    select Email,count(Email) as a from Person order by a limit 1
) as b
```

- 无值返回空

```sql
select Email from person group by Email having count(Email) > 1
```

# 183

[customers-who-never-order](https://leetcode.cn/problems/customers-who-never-order/)

简单关联查询

```sql
select a.name as Customers from Customers as a
where a.id not in (
    select CustomerId from orders
)
```

# 196

[delete-duplicate-emails](https://leetcode.cn/problems/delete-duplicate-emails/)

我的解法，不符合题目要求中删除最小的id

```sql
delete from person where id in 
(
    select id from
    (
        select id,count(id) as a 
            from person 
            group by email having a > 1
    ) as b
)
```

笛卡尔积，先筛选email相同，在删除id大的数据

```sql
delete p1 from person as p1,person as p2
where p1.id > p2.id and p1.email = p2.email
```

# 197

[rising-temperature](https://leetcode.cn/problems/rising-temperature/)

用一列不同行计算，使用`datediff()`计算两个日期之间的天数

```sql
select a.id 
    from weather as a,weather as b
    # dateDiff(a,b) 返回a,b两个日期之间的天数差
where dateDiff(a.recordDate,b.recordDate) = 1
and 
b.Temperature < a.Temperature
```

# 511

[game-play-analysis-i](https://leetcode.cn/problems/game-play-analysis-i/)

找到日期最小的

```sql
select player_id,min(event_date) as first_login
    from activity
group by player_id
```

# 584

[find-customer-referee](https://leetcode.cn/problems/find-customer-referee/)

查询结果包含null

```sql
select name from customer where referee_id != 2 or referee_id is null
```

使用`ifnull()`

```sql
select name from customer 
# ifnull(a,b) 如果a是null返回b，如果a不是null，返回a
where ifnull(referee_id,0) != 2
```

# 586

[customer-placing-the-largest-number-of-orders](https://leetcode.cn/problems/customer-placing-the-largest-number-of-orders/)

```sql
select customer_number from (
    select customer_number,count(customer_number) as a
    from orders
    group by customer_number
    order by a desc
) as a limit 1
```

# 595

[big-countries](https://leetcode.cn/problems/big-countries/)

```sql
select name,population,area from world 
where area > 3000000 or population >= 25000000
```

# 596

[classes-more-than-5-students](https://leetcode.cn/problems/classes-more-than-5-students/)

```sql
select class from (
    select class,count(class) as a from Courses
    group by class having a >= 5
) as b
```

# 607

[sales-person](https://leetcode.cn/problems/sales-person/)

```sql
select c.name as name from SalesPerson as c 
    where c.sales_id not in (
        select b.sales_id from orders as b where com_id in (
            select a.com_id as com_id from company as a
                where a.name = 'RED'
        )
)
```

# 620

[not-boring-movies](https://leetcode.cn/problems/not-boring-movies/)

```sql
select * from cinema 
where description != 'boring' and id % 2 = 1 
order by rating desc
```

# 627

[swap-salary](https://leetcode.cn/problems/swap-salary/)

case用法： case a when cond1 then exp1 else cond2 then exp2 else exp3

当a满足条件cond1时， 返回exp1 当a满足条件cond2时， 返回exp2 否则 返回exp3

```sql
update salary set sex = (
    case sex when 'm' then 'f' else 'm' end
)
```

# 1050

[actors-and-directors-who-cooperated-at-least-three-times](https://leetcode.cn/problems/actors-and-directors-who-cooperated-at-least-three-times/)

我的解法

```sql
select actor_id,director_id from (
    select actor_id,director_id,count(actor_id) as a,count(director_id) as b
    from ActorDirector
	group by actor_id,director_id having a >= 3 and b >= 3

) as c
```

```sql
select actor_id,director_id
from ActorDirector
group by actor_id, director_id
having count(*)>=3
```


