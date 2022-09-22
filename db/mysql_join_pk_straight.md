```sql
select pp.id, pp.name, b.name as brand_name
from product pp USE INDEX (`PRIMARY`)
join brand b USE INDEX (`PRIMARY`) on b.id = pp.brand_id
# JOIN (select product_id as product_id1 from product_status where type_id = 1 and code = 20) approvalStatus on pp.id = approvalStatus.product_id1
# JOIN (select product_id as product_id2 from product_status where type_id = 2 and code = 20) saleStatus on pp.id = saleStatus.product_id2
# JOIN (select product_id as product_id3 from product_status where type_id = 3 and code = 20) displayType on pp.id = displayType.product_id3
where 1=1
# and (1=2
#     or (1=1 AND pp.name like '%black%' AND pp.name like '%women%' )
#     or (1=1 AND pp.brand_id in ( select id from brand where name like '%maison%' ) AND pp.brand_id in ( select id from brand where name like '%kitsune' ) )
# )
# and pp.id in ( 1,2,3,4,5,6,7 )
order by pp.id desc  # <--- !!!!!!!!
limit 30 offset 0
;

# [2022-09-21 22:43:22] 30 rows retrieved starting from 1 in 7 s 618 ms (execution: 7 s 564 ms, fetching: 54 ms)
select 1
, b.id as b__id, b.name as b__name
, p.*
from product p
join brand b on p.brand_id = b.id
order by p.id desc
limit 30
;

# [2022-09-21 22:43:43] 30 rows retrieved starting from 1 in 123 ms (execution: 28 ms, fetching: 95 ms)
# [2022-09-21 22:46:45] 30 rows retrieved starting from 1 in 132 ms (execution: 36 ms, fetching: 96 ms)
select 1
, ( select id    from brand b where b.id = p.brand_id ) as b__id
, ( select name  from brand b where b.id = p.brand_id ) as b__name
, p.*
from product p
order by p.id desc
limit 30
;

# [2022-09-21 22:45:19] 30 rows retrieved starting from 1 in 120 ms (execution: 35 ms, fetching: 85 ms)
select 1
, p.*
from product p
order by p.id desc
limit 30
;
```