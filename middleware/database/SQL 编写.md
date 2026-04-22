# SQL编写

## 经典场景

1、最近的一条数据

最近：order by time desc limit 1

主要是多个人的时候，比如：查询最近 10 个人的最近一条登录记录

```sql
select * from login_log l inner join (
	select user_id, max(login_time) nearest_login_time from login_log l1 group by user_id
) as temp on l.user_id=temp.user_id and l.login_time=temp.nearest_login_time
order by l.login_time desc
limit 10;
```

核心是：先找到每个用户最近的时间，再通过自关联，来获取所有的数据，然后再 order by 之后 limit

增强：最近的几条数据

1.1 部门前三高的薪水的人

```sql
# 1. 找出每个部门下前三高的薪水
#   1.1 前三高，转化为：比这个薪水高的薪水 distinct 之后不能超过2条
# 2. 找出这个部门下，薪水是这些薪水的人
# 3. join获得部门名称

select d.name as 'Department', e1.name as 'Employee', e1.salary
from employee e1 join department d on e1.departmentId = d.id
# 3> 表示：e1.salary在前三里
where 3 > (
    # 取出比e1.salary大的薪水数值有几个
    select count(distinct e2.salary)
    from employee e2 
    where e2.salary > e1.salary
    # 要将部门也关联上，否则可能会查出不同部门间复核条件的数据
    and e1.departmentId = e2.departmentId
);
```



2、连续出现，前后有关联

例：连续出现三次的数字有哪些

```sql
select * 
from 
	log l1,
	log l2,
	log l3
where l1.id=l2.id-1
	and l2.id=l3.id-1
	and l1.num = l2.num
	and l2.num = l3.num;
```

核心：通过同时开启多个表视图，然后表之间进行比较来实现想要的效果

